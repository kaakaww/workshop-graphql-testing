# Workshop Guidebook: Automated GraphQL Security Testing

This workshop shows you how to scan a GraphQL application with StackHawk, and automate that scan in GitHub Actions.

You can find the slide deck for this workshop [here](https://docs.google.com/presentation/d/1OqDYWux-dAwmzfDx4DbfnnnGfIBJiwYIG88zTD5b8YM/edit?usp=sharing).

Not attending our workshop right now? [Watch it](https://www.youtube.com/watch?v=7SiYpZYDlEg) on your own schedule.

---

## Prerequisites

To get the most out of this workshop, make sure you have the following prerequisites.

* Discord - Find us in **#oct11-gql-security-testing** under the 🧩 RADV FREE WORKSHOPS category
* Docker -- [Get](https://docs.docker.com/get-docker) the latest version
* HawkScan -- ```docker pull stackhawk/hawkscan```

## Step 1: Fork the Test Application

Fork the `vuln-graphql-api` app:

<https://github.com/kaakaww/vuln-graphql-api>

Then clone your fork to your workstation:

```shell
git clone git@github.com:<YOUR-GITHUB-ORG>/vuln-graphql-api
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

Follow the guidance to save your StackHawk API key to `~/.hawk/hawk.rc`

Copy the intial HawkScan configuration file, `stackhawk.yml`, to the base of your project directory:

```yaml
# ./stackhawk.yml
app:
  applicationId: <YOUR-APP-ID>
  env: Development
  host: http://localhost:3000
```

> ☝️ Replace `<YOUR-APP-ID>` with the App ID you created in the StackHawk platform.

Scan `vuln-graphql-api`:

```shell
source ~/.hawk/hawk.rc
docker run -e API_KEY=${HAWK_API_KEY} --rm -v $(pwd):/hawk:rw -it --network host stackhawk/hawkscan:latest
```

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
  autoPolicy: true
  autoInputVectors: true
hawk:
  spider:
    base: false
```

Scan again:

```shell
docker run -t -e API_KEY=${HAWK_API_KEY} -v $(pwd):/hawk --network host stackhawk/hawkscan:latest
```

## Step 5: Automate in GitHub Actions

Add your StackHawk API key as a GitHub Secret. Go to your repository in GitHub, and under the **Settings** section, find **Secrets** in the left-hand pane.

Enter your StackHawk API key as a secret named `HAWK_API_KEY`.

Create the workflow, `.github/workflows/build-and-scan.yml`:

```yaml
# .github/workflows/build-and-scan.yml
name: Build and Scan
on:
  push:
jobs:
  hawkscan:
    name: Build and Scan
    runs-on: ubuntu-20.04
    steps:
      - name: Clone repo
        uses: actions/checkout@v2
      - name: Build and run vuln-graphql-api
        run: docker-compose up --build --detach
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

Check your workflow in GitHub Actions, and your scan results on [StackHawk](https://app.stackhawk.com).

## Workshop Complete

You just automated DAST GraphQL scanning in a build pipeline!

Here are some additional resources for further tuning StackHawk for *your* applications.

* [HawkDocs](https://docs.stackhawk.com) - StackHawk Documentation.
* [GraphQL Configuration](https://docs.stackhawk.com/hawkscan/configuration/graphql-configuration.html) - Details on how to tune your GraphQL scan.
* [Authenticated Scanning](https://docs.stackhawk.com/hawkscan/authenticated-scanning.html) - Guides for authenticating HawkScan to your application for deeper scans.
* [Continuous Integration](https://docs.stackhawk.com/continuous-integration/) - Guides for integrating HawkScan with the most popular CI/CD systems.
* [StackHawk Blog](https://www.stackhawk.com/blog) - Tips, tricks, and strategies to help you continuously test and secure your applications.
