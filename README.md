# Workshop Guidebook: Automated GraphQL Application Security Testing

This workshop is designed to help you get started with GraphQL application security testing in GitHub Actions. Participants get hands-on experience with:

* GitHub Actions workflows
* Dependabot software composition analysis (SCA)
* CodeQL static application security testing (SAST) scanning
* StackHawk dynamic application security test (DAST) scanning

You can find the slide deck for this workshop [here](https://docs.google.com/presentation/d/1OqDYWux-dAwmzfDx4DbfnnnGfIBJiwYIG88zTD5b8YM/edit?usp=sharing).

If you are attending the December 2021 [GraphQL Galaxy](https://graphqlgalaxy.com/workshops-3h) live workshop, [join us on Discord](https://discord.gg/ekrCgSTR93). Find us in **#dec6-security-testing-stackhawk** under the **🚀GQL WORKSHOPS** category in the GitNation Tech Communities Discord.

---

## Prerequisites

To get the most out of this workshop, make sure you have the following prerequisites.

* A web browser
* GitHub - [Sign up](https://github.com/signup) if you don't have an account

## Step 1: Continuous Integration in GitHub Actions

First, clone our test GraphQL application. We will build and test it in a GitHub Actions workflow automatically on every commit. Later, we will add various other automated security tests for it.

Fork the `vuln-graphql-api` app:

<https://github.com/kaakaww/vuln-graphql-api>

Go to the **Code** section of your newly forked repository in GitHub. Create a new file using the **Add file --> Create new file** button. Name the file `.github/workflows/build-and-test.yml`, and add the following contents:

```yaml
# .github/workflows/build-and-test.yml
name: Build and Test
on:
  push:
jobs:
  hawkscan:
    name: Build and Test
    runs-on: ubuntu-20.04
    steps:
      - name: Clone repo
        uses: actions/checkout@v2
      - name: Build the app
        run: docker-compose build
```

Commit the change.

Go to the **Actions** section of your repository, and you should see the new workflow running.

## Step 2: Dependency Scanning with Dependabot

Next, add Software Composition Analysis (SCA) to your GitHub repository to scan the GraphQL test application for known dependency vulnerabilities.

Go to the **Settings** section of your repo, and find the **Security & analysis** section in the left pane. Enable the **Dependency graph**, **Dependabot alerts**, and **Dependabot security updates** features in this section. Dependabot is now configured.

Go to the **Security** section of your GitHub repo, and click into the **Dependabot alerts** on the left pane. Examine some of the dependency alerts, and see if you can resolve them.

## Step 3: Static Code Analysis with CodeQL

Go to the **Security** section of your repo. Click on **Set up code scanning**. Find the **CodeQL Analysis** code scanning tool, and click **Set up this workflow**.

Examine the GitHub Actions workflow, `.github/workflows/codeql-analysis.yml`, and commit it to the repo.

Now go to the **Actions** section of your repo, and watch your new CodeQL workflow run.

When CodeQL has finished, examine the results in the **Security** section under **Code scanning alerts** in the left pane.

## Step 4: Dynamic App Scanning with StackHawk 🦅

[Sign up](https://app.stackhawk.com) for a StackHawk Developer account. Follow the Get Started flow to:

* Create your StackHawk API key
* Create your first "application" in the StackHawk platform
* Create a starter configuration file for HawkScan to scan your application.

### Create and Save the API Key

The first step in the getting started flow is to create your API key.

Stash your StackHawk API key in GitHub Secrets. In your repo, navigate to the **Settings** section, and find **Secrets** in the left pane.

Add a secret named `HAWK_API_KEY`, and add your StackHawk API key as the value.

### Commit the `stackhawk.yml` Configuration File

The next step is to create your first application in the StackHawk platform. When you create a new app, StackHawk helps you generate a `stackhawk.yml` configuration file that is tuned to the kind of application you want to scan.

In this step, take care to specify that your app is a GraphQL app, and it has an introspection endpoint at the default `/graphql` path.

Download the `stackhawk.yml` file that you generate in this step. Copy the contents into a new file at the base of your repo named `stackhawk.yml`. Commit the file.

Your configuration file should look similar to this, but with your own unique App ID.

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

### Add a StackHawk Scan to your Build and Test Workflow

Update your Build and Test workflow. Add a step to start the `vuln-graphql-api` service, and a step to run HawkScan using the StackHawk Action at the end:

```yaml
# .github/workflows/build-and-test.yml
name: Build and Test
on:
  push:
jobs:
  hawkscan:
    name: Build and Test
    runs-on: ubuntu-20.04
    steps:
      - name: Clone repo
        uses: actions/checkout@v2
      - name: Build the app
        run: docker-compose build
      - name: Run the app
        run: docker-compose up --detach
      - name: Scan the app
        uses: stackhawk/hawkscan-action@v1.3.2
        with:
          apiKey: ${{ secrets.HAWK_API_KEY }}
```

Commit this change.

### Check your Scan Results

Go to the **Actions** section of your repo, and watch your updated Build and Test workflow run. Examine the **Run HawkScan** job step logs.

[Check your scan results](https://app.stackhawk.com/scans) on the StackHawk platform.

## WORKSHOP COMPLETE

You just automated SCA, SAST, and DAST scanning of a GraphQL application!

Read more about [GitHub Actions](https://docs.github.com/en/actions), [CodeQL](https://codeql.github.com/docs/), and [Dependabot](https://docs.github.com/en/code-security/supply-chain-security/keeping-your-dependencies-updated-automatically/configuration-options-for-dependency-updates). And check out the [GitHub Actions Marketplace](https://github.com/marketplace?type=actions), where you can find other Actions to build out your pipeline.

Go deeper with HawkScan to tune it for *your* application.

* [GraphQL Configuration](https://docs.stackhawk.com/hawkscan/configuration/graphql-configuration.html) - Details on how to tune your GraphQL scan.
* [Authenticated Scanning](https://docs.stackhawk.com/hawkscan/authenticated-scanning.html) - Guides for authenticating HawkScan to your application for deeper scans.
* [Continuous Integration](https://docs.stackhawk.com/continuous-integration/), where you can see our guides for integrating HawkScan with the most popular CI/CD systems.
* [StackHawk Blog](https://www.stackhawk.com/blog), with technical tips, tricks, and walkthroughs to help you secure and test your applications.
