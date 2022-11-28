# Workshop Guidebook: Automated GraphQL Application Security Testing

This workshop is designed to help you get started with automated GraphQL application security testing using GitHub Actions. Participants get hands-on experience with:

* GitHub Actions workflows
* Dependabot software composition analysis (SCA)
* CodeQL static application security testing (SAST) scanning
* StackHawk dynamic application security test (DAST) scanning

You can find the slide deck for this workshop [here](https://docs.google.com/presentation/d/13EageIyLyV2ad_g8y7t3gGNOJE3SlMrbRaQ1hh6V51w/edit?usp=sharing)

---

## Prerequisites

To get the most out of this workshop, make sure you have the following prerequisites.

* A web browser
* GitHub account - [Sign up](https://github.com/signup) if you don't have one

## Agenda

In this workshop we will work through the following tasks:

1. Fork a test GraphQL app, and set up a CI/CD workflow for it in GitHub Actions.
2. Enable Software Composition Analysis (SCA) scanning with Dependabot.
3. Enable Static Application Security Test (SAST) scanning with CodeQL.
4. Enable Dynamic Application Securituy Test (DAST) scanning with StackHawk.

## Step 1: Continuous Integration in GitHub Actions

First, fork the test GraphQL application repository, [`vuln-graphql-api`](https://github.com/kaakaww/vuln-graphql-api). We will build and test it in a GitHub Actions workflow on every commit.

Fork the `vuln-graphql-api` app from GitHub using the "Fork" button at the top right side of the page:

<https://github.com/kaakaww/vuln-graphql-api>

After forking, go to the **Code** section of your new forked repository in GitHub. Create a new file using the **Add file --> Create new file** button. Name the file `.github/workflows/build-and-test.yml`, and add the following contents:

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
        uses: actions/checkout@v3
      - name: Build the app
        run: docker-compose build
```

Commit the change.

Go to the **Actions --> All Workflows** section of your repository, and you should see the new workflow running.

## Step 2: Dependency Scanning with Dependabot

Next, add Software Composition Analysis (SCA) to your GitHub repository to scan the GraphQL test application for known dependency vulnerabilities.

Go to the **Settings --> Code security and analysis** section in your repo. Enable the **Dependency graph**, **Dependabot alerts**, and **Dependabot security updates** features in this section. Dependabot is now configured.

Go to the **Security --> Dependabot** section of your GitHub repo, and click into the **Dependabot alerts** on the left pane. Examine some dependency alerts, and see if you can resolve them.

## Step 3: Static Code Analysis with CodeQL

Go to the **Security --> Code Scanning** section of your repo. Click on **Configure scanning tool**. Find the **CodeQL Analysis** code scanning tool in the "Security" workflow list, and click **Configure**.

This will add a file to your repo.  Examine the GitHub Actions workflow, `.github/workflows/codeql-analysis.yml`, and commit it to the repo.

Now go to the **Actions --> CodeQL** section of your repo, and watch your new CodeQL workflow run.

When CodeQL has finished, examine the results in the **Security** section under **Code scanning alerts** in the left pane.

## Step 4: Dynamic App Scanning with StackHawk 🦅

[Sign up](https://app.stackhawk.com) for a StackHawk Developer account. Click on the "Confirm Your Email Address" link in the email invite and choose "Scan My Application."

Follow the Get Started flow:

* In the "Scanner Setup" step
  * Select "StackHawk Docker Image" as the Scanner Type
  * Copy the API Key from StackHawk and put it in GitHub Secrets
    * In GitHub, go to **Settings --> Secrets --> Actions**
    * Click on "New repository secret"
    * Make the name `HAWK_API_KEY` and paste the StackHawk API Key as the secret
* Next step, create your first Application in the StackHawk platform
  * The Application name can be whatever you want
  * Pick the Environment 
  * For the Host URL, put `http://localhost:3000`
* After that, set the App Type
  * "Application Type" to `API`
  * "API Type" to `GraphQL`
  * "GraphQL Introspection Point" to `/graphql`
* Last step shows you an example `stackhawk.yml` file
  * Copy and paste that into a new file at the root of your new repo called `stackhawk.yml`
   
At this point, you have a StackHawk account with an API Key and Application all set up to receive scans!

#### Optional: Create and Save an API Key

If you didn't get the StackHawk API Key during the Getting Started flow, you can do so now.

In StackHawk, click on your name/icon on the left menu, and click **Settings --> API Keys**.  Then click the "Create New Api Key", give it a name, and copy the shown key so it can be pasted into a GitHub Secret.

* Copy the API Key from StackHawk and put it in GitHub Secrets
    * In GitHub, go to **Settings --> Secrets --> Actions**
    * Click on "New repository secret"
    * Make the name `HAWK_API_KEY` and paste the StackHawk API Key as the secret

### Optional: Create an Application and Configuration

If you want, you can create a different application in the StackHawk platform. When you create a new app, StackHawk helps you generate a `stackhawk.yml` configuration file that is tuned to the kind of application you want to scan.

In this step, take care to specify that
  * The "Host" is `http://localhost:3000`
  * The "Application Type" is `API`
  * The "API Type" is `GraphQL`
  * The URL Path is `/graphql`

Download the `stackhawk.yml` file that you generate in this step. Copy the contents into a new file at the base of your repo named `stackhawk.yml`. Commit the file.

Your configuration file should look similar to this, but with your own unique App ID.

```yaml
# ./stackhawk.yml
# -- stackhawk configuration for vuln-graphql-api --
app:
  # -- An applicationId obtained from the StackHawk platform. --
  applicationId: <YOUR_APP_ID> # (required)
  env: Development # (required)
  host: http://localhost:3000 # (required)

  # -- Customized Configuration for GraphQL/SOAP/OpenAPI, add here --
  graphqlConf:
    enabled: true
    schemaPath: /graphql # OR...
    operation: ALL # Types: ALL, QUERY, MUTATION
    requestMethod: POST # Types: POST, GET
  autoPolicy: true
  autoInputVectors: true

# BONUS: Fail the scan if we find issues of HIGH criticality
hawk:
  failureThreshold: high
```

### Add a StackHawk Scan to your Build and Test Workflow

Update your Build and Test workflow by editing `blob/main/.github/workflows/build-and-test.yml`.  Go to this file in GitHub and click the "pencil" icon to edit directly.

Add two steps:
  * One to start the `vuln-graphql-api` service
  * Another to run HawkScan using the StackHawk Action at the end

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
        uses: actions/checkout@v3
      - name: Build the app
        run: docker-compose build
      - name: Run the app
        run: docker-compose up --detach
      - name: HawkScan
        uses: stackhawk/hawkscan-action@v2.0.0
        with:
          apiKey: ${{ secrets.HAWK_API_KEY }}
```

Commit this change.

### Check your Scan Results

Go to the **Actions** section of your repo, and watch your updated "Build and Test" workflow run. Examine the **HawkScan** job step logs.

[Check your scan results](https://app.stackhawk.com/scans) on the StackHawk platform.

## WORKSHOP COMPLETE

You just automated SCA, SAST, and DAST scanning of a GraphQL application!

Read more about [GitHub Actions](https://docs.github.com/en/actions), [CodeQL](https://codeql.github.com/docs/), and [Dependabot](https://docs.github.com/en/code-security/supply-chain-security/keeping-your-dependencies-updated-automatically/configuration-options-for-dependency-updates). And check out the [GitHub Actions Marketplace](https://github.com/marketplace?type=actions), where you can find other Actions to build out your pipeline.

Go deeper with HawkScan to tune it for *your* application.

* [GraphQL Configuration](https://docs.stackhawk.com/hawkscan/configuration/graphql-configuration.html) - Details on how to tune your GraphQL scan.
* [Authenticated Scanning](https://docs.stackhawk.com/hawkscan/authenticated-scanning.html) - Guides for authenticating HawkScan to your application for deeper scans.
* [Continuous Integration](https://docs.stackhawk.com/continuous-integration/) - Guides for integrating HawkScan with the most popular CI/CD systems.
* [StackHawk Blog](https://www.stackhawk.com/blog) - Technical tips, tricks, and walkthroughs to help you secure and test your applications.
