# Selenium Dockerized :whale:

This repository creates Docker images that are perfect to run end-to-end tests.

## What Are the Goals of This Project?

Setting up a selenium environment is complicated and error prone.

This repository provides an docker image that is ready to go, no configuration required.

The following components are included:

1. Node 10
2. Java Runtime Environment 8
3. Google Chrome
4. Seleniums Driver for Google Chrome

## Relevant URLs (*)

Get images from the [Docker Hub](https://hub.docker.com/r/5minds/selenium_dockerized/)

## How Do I Set This Project Up?

### Prerequesites

1. Docker

### Setup/Installation

Simply build the docker image:

```bash

docker build . --tag 5minds/selenium_dockerized

```

## How Do I Use This Project?

### Usage

The intended way to run this image is to mount all files required your tests
into the container. Then you supply your script via the `COMMAND` argument.

```bash

$MY_PROJECT_FOLDER = (...)

docker run \
  --env HOME=$MY_PROJECT_FOLDER \
  --workdir $MY_PROJECT_FOLDER \
  --volume=$MY_PROJECT_FOLDER:$MY_PROJECT_FOLDER:Z \
  --rm \
  --name my-e2e-container \
  5minds/selenium_dockerized:test \
  test/run_e2e_tests.sh

```

## What Else Is There to Know?

This project is mainly used by BPMN-Studio.

### Authors/Contact Information

- Paul Heidenreich <paul.heidenreich@5minds.de>

### Related Projects

- [BPMN-Studio](https://github.com/process-engine/bpmn-studio)
