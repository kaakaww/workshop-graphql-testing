# Workshop Guidebook: Automated GraphQL Security Testing

This workshop shows you how to scan a GraphQL application with StackHawk, and automate that scan in GitHub Actions.

You can find the slide deck for this workshop [here](https://docs.google.com/presentation/d/1OqDYWux-dAwmzfDx4DbfnnnGfIBJiwYIG88zTD5b8YM/edit?usp=sharing).

Not attending our workshop right now? [Watch it](https://www.youtube.com/watch?v=7SiYpZYDlEg) on your own schedule.

---

## Prerequisites

To get the most out of this workshop, make sure you have the following prerequisites.

* Discord - Find us in **#oct11-gql-security-testing** under the 🧩RADV FREE WORKSHOPS category
* Docker -- [Get](https://docs.docker.com/get-docker) the latest version
* HawkScan -- ```docker pull stackhawk/hawkscan```
* The GitHub CLI (Optional)
  * [Install](https://github.com/cli/cli#installation) the CLI
  * Login -- ```gh auth login```

## Step 1: Fork the Test Application

Fork the `vuln-graphql-api` app with the GitHub CLI:

```shell
gh repo fork kaakaww/vuln-graphql-api
```

...**or**, fork it from the website:

<https://github.com/kaakaww/vuln-graphql-api>

Then clone your fork to your workstation with the GitHub CLI:

```shell
gh repo clone vuln-graphql-api
```

...**or** clone it with `git`:

```shell
git clone <YOUR-GITHUB-ORG>/vuln-graphql-api
```

Enter your cloned project directory:

```shell
cd vuln-graphql-api
```

Prepare the `vuln-graphql-api` project directory for the workshop:

```shell
./scripts/workshop-prep.sh
```

## Step 2: Run the Test App

Build and run the test app:

```shell
export SERVER_PORT=3000
docker compose up --build --detach
```

Browse to the test app:

```shell
open http://localhost:3000
```

## Step 3: Your First HawkScan

[Sign up](https://app.stackhawk.com) for a StackHawk Developer Account. Create an API Key, App ID, Environment, and HawkScan initial configuration file in the Getting Started flow.

Copy the intial HawkScan configuration file, `stackhawk.yml`, to the base of your project directory:

```yaml
# ./stackhawk.yml
app:
 applicationId: <YOUR-APP-ID>
 env: Development
 host: https://localhost:3000
```

> ☝️ Replace `<YOUR-APP-ID>` with the App ID you created in the StackHawk platform.

Scan `vuln-graphql-api`:

```shell
export API_KEY=<YOUR-API-KEY>
docker run -t -e API_KEY -v $(pwd):/hawk --network host stackhawk/hawkscan
```

> ☝️ Replace `<YOUR-API-KEY>` with the API key you created in the StackHawk platform.

## Step 4: Tune for GraphQL

Update your `stackhawk.yml` configuration file:

```yaml
# ./stackhawk.yml
app:
 applicationId: <YOUR-APP-ID>
  env: Development
  host: http://localhost:3000
  graphqlConf:
    enabled: true
    operation: QUERY
  autoPolicy: true
  autoInputVectors: true
hawk:
  spider:
    base: false
```

Scan again:

```shell
docker run -t -e API_KEY -v $(pwd):/hawk --network host stackhawk/hawkscan:latest
```

## Step 5: Automate in GitHub Actions

Add your StackHawk API key as a GitHub Secret:

```shell
gh secret set HAWK_API_KEY --repos="vuln-graphql-api"
```

Create the workflow, `.github/workflows/hawkscan.yml`:

```yaml
# .github/workflows/hawkscan.yml
name: HawkScan
on:
  push:
jobs:
  hawkscan:
    name: HawkScan
    runs-on: ubuntu-20.04
    steps:
      - name: Clone repo
        uses: actions/checkout@v2
      - name: Build and run vuln-graphql-api
        run: >
          SERVER_PORT=3000
          docker-compose up --build --detach
      - name: Run HawkScan
        uses: stackhawk/hawkscan-action@v1.3.1
        with:
          apiKey: ${{ secrets.HAWK_API_KEY }}
```

Push your changes to GitHub:

```shell
git add .
git commit -m "add HawkScan to the build workflow"
git push
```

## All Done

Congratulations! You just automated DAST scanning in a build pipeline.

Now try StackHawk on *your* application! Here are some additional resources to help you on your way.

* [HawkDocs](https://docs.stackhawk.com), where you can read all the details on how to configure and run HawkScan in your environment.
* [GraphQL Configuration](https://docs.stackhawk.com/hawkscan/configuration/graphql-configuration.html), where you can find more details on tuning your GraphQL scan
* [Continuous Integration](https://docs.stackhawk.com/continuous-integration/), where you can see our guides for integrating HawkScan with the most popular CI/CD systems
