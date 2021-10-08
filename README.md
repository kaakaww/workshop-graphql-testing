# Workshop Cheat Sheet: Automated GraphQL Security Testing

This workshop shows you how to get started scanning a GraphQL application with StackHawk. You will also learn to automate scanning on every code push to GitHub using GitHub Actions.

Not attending our workshop right now? You can watch the [June '21 GraphQL workshop](https://www.youtube.com/watch?v=7SiYpZYDlEg) on your own schedule.

---

## Prerequisites

To get the most out of this workshop, make sure you have the following prerequisites before getting started.

* Discord 
  * [Install](https://discord.com/) Discord
  * [Join](https://discord.gg/2XsFrkMVdc) the GitNation Server
  * Find us in **#oct11-gql-security-testing** – it's under the 🧩RADV FREE WORKSHOPS category
* Docker -- [install](https://docs.docker.com/get-docker) the latest version
* HawkScan -- ```docker pull stackhawk/hawkscan```
* The GitHub CLI (`gh`)
  * [Install](https://github.com/cli/cli#installation) the CLI
  * Login -- ```gh auth login```

## Step 1: Fork the Test Application

Fork the vuln-graphql-api app with the GitHub CLI:

```shell
gh repo fork kaakaww/vuln-graphql-api
```

...or from the website:

```shell
open https://github.com/kaakaww/vuln-graphql-api
```

Clone it to your workstaion with the GitHub CLI:

```shell
gh repo clone vuln-graphql-api
```

...or with `git`:

```shell
git clone <YOUR-GITHUB-ORG>/vuln-graphql-api
```

Enter your cloned project directory:

```shell
cd vuln-graphql-api
```

Prepare the vuln-graphql-api project directory for the workshop:

```shell
./scripts/workshop-prep.sh
```

## Step 2: Run the Test App

Build and run the test app:

```shell
export SERVER_PORT=3000
docker compose up --build --detach
```

Browse to the test app

```shell
open http://localhost:3000
```

## Step 3: My First HawkScan

Sign up for a StackHawk Developer Account

```shell
open https://app.stackhawk.com
```

Create an App, Environment, and API Key in the Getting Started flow.

Scan vuln-graphql-api

```shell
export SERVER_PORT=3000
export API_KEY=hawk.XXXxXXXXXXxXXXxXXxxX.XXXxXXXxXXxxXXXXxXXX
docker run -t -e API_KEY -v $(pwd):/hawk --network host stackhawk/hawkscan
```

## Step 4: Tune for GraphQL

Update your stackhawk.yml configuration:

```yaml
# ./stackhawk.yml
app:
 applicationId: ${APP_ID}
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

Scan again

```shell
docker run -t -e API_KEY -v $(pwd):/hawk --network host stackhawk/hawkscan
```

## Step 5: Automate in GitHub Actions

Create `./.github/workflows/hawkscan.yml`

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

Commit. Push. Scan.

```shell
git add .
git commit -m "bring the noise"
git push
```

## All Done

Congratulations!

Now try it on *your* application! Here are some additional resources to help you on your way.

* [HawkDocs](https://docs.stackhawk.com), where you can read all the details on how to configure and run HawkScan in your environment.
* [GraphQL Configuration](https://docs.stackhawk.com/hawkscan/configuration/graphql-configuration.html), where you can find more details on tuning your GraphQL scan
* [Continuous Integration](https://docs.stackhawk.com/continuous-integration/), where you can see our guides for integrating HawkScan with the most popular CI/CD systems
