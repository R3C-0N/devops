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

**docker build and push**
```yaml
# define job to build and publish docker image
build-and-push-docker-image:
    # run only when code is compiling and tests are passing
    needs: test-backend
    environment: devops-tp02
    runs-on: ubuntu-22.04

    # steps to perform in job
    steps:
        - name: Checkout code
          uses: actions/checkout@v2.5.0

        # Login with docker login action
        - name: Docker Login
          uses: docker/login-action@v2.1.0
          with:
              username: ${{secrets.DOCKER_USER}}
              password: ${{secrets.DOCKER_PWD}}

       # Then we will build every images with the same method/action
        - name: Build image and push backend
          uses: docker/build-push-action@v3
          with:
          # relative path to the place where source code with Dockerfile is located
            context: ./backend-api
            # Note: tags where the image is pushed
            tags:  ${{secrets.DOCKER_USER}}/tp01-backend
            # Push if github.ref (the branch updated) is equal to 'refs/heads/TP02'
            push: ${{ github.ref == 'refs/heads/TP02' }}

        - name: Build image and push database
          uses: docker/build-push-action@v3
          with:
            context: ./database
            tags:  ${{secrets.DOCKER_USER}}/tp01-database
            push: ${{ github.ref == 'refs/heads/TP02' }}

        - name: Build image and push httpd
          uses: docker/build-push-action@v3
          with:
              context: ./http-server
              tags:  ${{secrets.DOCKER_USER}}/tp01-httpd
              push: ${{ github.ref == 'refs/heads/TP02' }}
```

