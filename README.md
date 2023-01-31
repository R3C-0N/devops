# TP 2

### 2-1 What are testcontainers?

There are containers used to launch every app tests

### 2-2 Document your Github Actions configurations.

**main.yml**
```yml
name: CI devops 2023
'on':
# For each push on branch TP02 launch github Action
    push:
        branches: TP02
        # Do nothing on pull request
    pull_request:

jobs:
    test-backend:
        runs-on: ubuntu-22.04
        steps:
        #checkout your github code using actions/checkout@v2.5.0
            - uses: actions/checkout@v2.5.0

        #do the same with actions/setup-java that enable to setup jdk 17
            - name: Set up JDK 17
              uses: actions/setup-java@v3.9.0
              with:
                distribution: corretto
                java-version: '17'

        #finally build your app with the latest command
            - name: Build and test with Maven
              run: mvn clean verify --file ./backend-api/app/pom.xml
```
**Execution**
![github action](./img/github-action.png)