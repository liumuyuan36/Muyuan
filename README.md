# Muyuan
Hello world
i need  to prove that I have the ability to finish all the task
# Climate archive app

[![Acceptance tests](https://github.com/sebsteinig/climate-archive/actions/workflows/cypress.yml/badge.svg)](https://github.com/sebsteinig/climate-archive/actions/workflows/cypress.yml) [![Build and test](https://github.com/sebsteinig/climate-archive/actions/workflows/docker.yml/badge.svg)](https://github.com/sebsteinig/climate-archive/actions/workflows/docker.yml) [![Production acceptance tests](https://github.com/sebsteinig/climate-archive/actions/workflows/production.yml/badge.svg)](https://github.com/sebsteinig/climate-archive/actions/workflows/production.yml)

## Deploying to production

A remote Jenkins CI job controls deployment of the static site, [https://climatearchive.org/](https://climatearchive.org/). This periodically polls this Github hosted repo for new git tags.

Deployment is triggered by tagging the repo, to ensure deployments are from a fixed point on the main branch.

At present the automated deployment step has been disabled to support code reviews by Research IT prior to deployment. Once reviewed the the CI/CD pipeline is temporarily enabled to deploy the latest `v` prefixed tag of code to the production website. The Climate Archive npm Docker image is used to generate static files which are then deployed to https://climatearchive.org/ hosted on the Research IT's static hosting service, at the University of Bristol.

Example code to tag a new deployment.

```bash
git checkout main
git pull
git tag -a v0.0.7 -m "Deploy version 0.0.7 to production"
git push origin v0.0.7
```

## Development - using local node installation

### Installation

Prerequisistes:

* [Node.js](https://nodejs.org/en/download/).

Run the following commands:

``` bash
# Install dependencies (only the first time)
npm install

# Run the local server at localhost:8080
npm run dev

# Build for production in the dist/ directory
npm run build
```

### Running tests

On the command line:
``` bash
./node_modules/.bin/cypress run
```

With integrated user interface:
``` bash
./node_modules/.bin/cypress open
```

## Development - using Docker

Use the `Dockerfile` within repository to build a controlled node environment. This Docker image will then used to create and run a custom *node* Docker container, that installs development node packages and also runs the development server. Your local code will be mounted into the `/var/src/` directory of the running Docker container. This will allow you to see your local changes in your local browser.

### Installation

Build the docker image and call it `npm`
``` bash
docker build -t npm .
```

Run docker container with the newly created image `npm`, called `climate-map`. 
``` bash
docker run -v "`pwd`":/var/src/ -p 8080:8080 -d --name climate-map npm
```
    
### View in the browser

Open http://localhost:8080

Changes to your local code within the code repo, will be picked up automaically within the running Docker container.

### Build distribution

Generate static files

```bash
docker build -t npm .
docker run \
    -v "`pwd`":/var/src/ \
    --entrypoint="" \
    --rm npm \
    npm run build
```

### Install a new package:

e.g. a dev package
``` bash
docker exec climate-map npm install cypress --save-dev
```

e.g. a prod package
``` bash
docker exec climate-map npm install gsap --save-prod
```

### Update a pacakge
``` bash
docker exec climate-map npm update cypress

or

docker exec climate-map npm install cypress@latest --save-dev
```

### Uninstall a package

```
docker exec climate-map npm uninstall cypress-visual-regression --save-dev
docker exec climate-map npm uninstall cypress --save-dev
```

### Audit npm packages

docker exec climate-map npm audit

### View running docker containers
``` bash
docker ps --all
```

### View container logs
``` bash
docker logs climate-map
```

### Stop & remove the developemnt docker container
``` bash
docker stop climate-map && docker rm climate-map
```

### Running acceptance tests
Run Cypress tests against a temporary development enviroment using docker-compose

```bash
sh run-dev-tests.sh
```

NB: You can view the dev instance being tested at http://localhost:8081

## Updating tests

Go to the `cypress/integration/` to find the test scripts. 

If the layout has changes and regression screenshot tests will start to fail, remove existing screenshots and regenerate base images. See https://github.com/mjhea0/cypress-visual-regression for more details. New screen shots will be created with the updated layout. Ensure you compare results to ensure all differences are anticpated. 

Example command
```bash
docker compose -f docker-compose.yml \
    -f docker-compose.override.yml \
    -f docker-compose-regenerate-base-images.yml \
    up --build --exit-code-from cypress
```
