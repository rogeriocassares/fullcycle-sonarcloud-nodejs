# Step by Step

Create a npm project and install dependencies
```bash
npm init
npm install jest @types/jest sonar-scanner --only-dev   
```
Modify package.json script section to match the following:
```json
  ...
  "scripts": {
    "test": "jest --coverage"
  },
  ...
```
Create src directory to write the code with `sum.js` and `sum.test.js` files.

Code the files, run the js with 
```bash
node sum.js
```

Test js with 
```bash
npm run test
----

> js@1.0.0 test
> jest --coverage

 PASS  src/sum.test.js
  ✓ add 1 + 2 to be equal3 (2 ms)

---------|---------|----------|---------|---------|-------------------
File     | % Stmts | % Branch | % Funcs | % Lines | Uncovered Line #s 
---------|---------|----------|---------|---------|-------------------
...files |     100 |      100 |     100 |     100 |                   
 sum.js  |     100 |      100 |     100 |     100 |                   
---------|---------|----------|---------|---------|-------------------
Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        0.586 s
Ran all test suites.
```

This shall create a directory with the results named as `coverage`

Create a file named as `sonar-project.properties`:
```properties
sonar.projectKey=node-test
sonar.projectName=node-test

sonar.sourceEncoding=UTF-8
sonar.sources=src

sonar.exclusions=**/*.test.ts

sonar.tests=src
sonar.test.inclusions=**/*.test.js

sonar.javascript.covaragePlugin=lcov
sonar.javascript.lcov.reportPaths=coverage/lcov.info
```

Then, create a Github Workflow to CI pipeline at `.github/workflows/ci.yaml`

```yaml
name: ci-sonarcloud
on:
  pull_request:
    branches:
      - develop

jobs:
  run-ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Use Node.js 18.6
        uses: actions/setup-node@v3
        with:
          node-version: 18.6
      
      - name: Install dependencies
      - run: npm ci

      - name: Build app --if-present
      - run: npm run build --if-present

      - name: Run tests
      - run: npm run test
      
      - name: SonarCloud Scan
        uses: SonarSource/sonarcloud-github-action@master
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}  # Needed to get PR information, if any
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

Initialize the .gitignore file to not push to origin all the files
```gitignore
.scannerwork
coverage
node_modules
```

Initialize git in the local machine with a main and a develop branch and push all the files to the GitHub

```bash
git init
git add .
git commit -m 'ci: add nodejs project files to the remote repository'
git branch -M main
git remote add origin https://github.com/rogeriocassares/fullcycle-sonarcloud-nodejs.git
git push -u origin main

git checkout -b develop
git push origin develop
```

On SonarCloud account, analyze a new Project and configure the SonarCloud bot on Github to access it and use the "With Github Actions".

Then, get the SONAR_TOKEN and add it as a SECRET in the Github repository configuration.

Then, adjust the `sonar.productKey` and `sonar.organization` values in the `sonar-project.properties` file.


Now, on Github change the default branch to develop.

Create a new branch from develop to modify the files and submit a new Pull Request
```bash
git checkout -b feature/ci-sonarcloud
git add .
git commit -m 'ci: add ci-sonarcloud configuration'
git push origin feature/ci-sonarcloud
```

On Github, check the develop branch and press to `Compare & pull request`, then `Create pull request` and `Merge`.

Once the first job is successfully completed, then go to repository settings and create rules to the develop to `Require status checks to pass before merging` and `Require branches to be up to date before merging` choosing `run-ci` status check and include `Include administrators` Options, then save it!

We got it! 

Now, on every PR to the develop branch, the github action called run-ci will run, call the sonarcloud process and after an async response from sonarcloud QualityGate it shall aprove to merge this branch to develop! 

All done!

God bless you!