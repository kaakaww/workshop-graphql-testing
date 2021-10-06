# Workshop Guide: Automated GraphQL Security Testing

This repo contains course materials for a workshop that StackHawk launched in June of 2021. In it, we help walk you through scanning a simple GraphQL application in GitGub Actions (a CI/CD platform) with the HawkScan DAST scanner.

If you missed the workshop and can't wait for the next one, watch the June '21 GraphQL workshop in the StackHawk YouTube channel (like and subscribe), and follow along with this copy/paste guidebook.


## Prerequisites
Docker
https://docs.docker.com/get-docker

HawkScan
docker pull stackhawk/hawkscan

vuln-graphql-api (Fork This)
https://github.com/kaakaww/vuln-graphql-api

Discord
https://discord.com/

## Ask Questions!
Join us at https://discord.gg/9zT2Vy5m 
Then find us in the #graphql-security-testing channel
Fork the GraphQL Test App
Fork it
https://github.com/kaakaww/vuln-graphql-api 

Clone it
git clone .../vuln-graphql-api

Prepare it for the workshop
./scripts/workshop-prep.sh
Run the Test App
Build and run
export SERVER_PORT=3000
docker compose up --build --detach
 # or...
docker-compose up --build --detach

Browse to it
http://localhost:3000

My First HawkScan
Sign up
https://app.stackhawk.com 

Scan vuln-graphql-api
export SERVER_PORT=3000
export API_KEY=hawk.XXXXXXXxXXXXXXXxxxxXXXXXxxxx
docker run -t -e API_KEY -v $(pwd):/hawk --network host stackhawk/hawkscan

Tune for GraphQL
Update your stackhawk.yml configuration
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

Scan again
docker run -t -e API_KEY -v $(pwd):/hawk --network host stackhawk/hawkscan

Automate in GitHub Actions
Create ./.github/workflows/hawkscan.yml
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
        uses: stackhawk/hawkscan-action@v1.3.0
        with:
          apiKey: ${{ secrets.HAWK_API_KEY }}

Commit. Push. Scan.
git add .
git commit -m “bring the noise”
git push

Thanks!
