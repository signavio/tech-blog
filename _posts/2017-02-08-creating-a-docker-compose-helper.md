---
title: Orchestrating microservices with Apache Kafka in a development environment
description: Pros and cons of creating a Docker management tool
tags: Docker, docker-compose, Node.js
---


[Signavio Process Intelligence](https://www.signavio.com/products/process-intelligence/) is an application that consists of multiple microservices that communicate through [Apache Kafka](https://kafka.apache.org).
We build and deploy our services as [Docker](https://www.docker.com) images and use [Kubernetes](https://www.kubernetes.io/) to manage our production system.
For someone that has not yet worked on this kind of architecture, this is awesome.
You can update one service and don't need to think about how you can integrate the new version, as Kubernetes (mostly) takes care of it.

But how can we develop on such a service?

- Apache Kafka and a database are required for a service to work.
- Changes in Kafka topics affect other services in the application.
- To verify you need to test if your service can communicate with other services correctly.

## Docker Compose to the rescue

> [\[Docker\] Compose](https://docs.docker.com/compose/) is a tool for defining and running multi-container Docker applications.

With a simple `.yml` configuration file you can orchestrate multiple Docker containers.
This helps with running all necessary micro-services locally.
First step completed.
To connect a locally running service into the Docker cluster we use a simple `nginx` proxy.
The next question would be how to chose if a service or a proxy should be run.

The first approach was using different `docker-compose.yml` files.
We would have different files per services, one where the actual Docker image was used and another one, where a `nginx` proxy with the correct name and ports was defined.
([Normal](#docker-composeservice-1devyml) vs [proxy](#docker-composeservice-1proxyyml) configuration)

Problem solved!

This approach worked very well, considering that the command needed to start a cluster was very large.

```
docker-compose \
- f docker-compose.service-1.dev.yml \
- f docker-compose.service-2.proxy.yml \
- f docker-compose.service-3.dev.yml \
up --build
```

As more services are added to the system this command would grow and it would be very hard for a developer to keep track of what services needed to be started in which setting.

That’s when I had the idea of creating a small tool to remove a lot of complexity.

`cool-docker-tool -p service-2 up --build` would have the same effect as the chain of `docker-compose` commands.

The idea was that if nothing is given except for the `docker-compose` command, all services should be started from their Docker image.
For every service specified in the `-p` option the `proxy` configuration should be used.

## Using multiple Compose files

This worked really well for a few weeks.
Then we learned about `docker-compose.override.yml`.
In this file you can easily overwrite service configurations from the `docker-compose.yml`.

This way saves a lot of files in SCM as we only need the `docker-compose.yml` and the `docker-compose.override.yml`.

The `override` file contains all the proxy definitions, so a developer can just comment out services that should be run normally and has an very simple way of starting his work environment.
([override](#docker-composeoverrideyml) configuration)

## Conclusion

It was a lot of fun experimenting with a `docker-compose` wrapper but in the end the built-in way was a better approach to solve this problem.
The only issue we still have is the annoyance of commenting in the `override` file.
Although this is a very simple approach and changes are always visible, it is still annoying to use another file.
In my opinion we still haven't found the perfect solution for our problem.

## Configuration files

#### `docker-compose.service-1.dev.yml`

```YAML
version: '2'
services:
  service-1:
    image: <docker-repo-url>/service-1:latest
    ports:
      - "${PORT}:8080"
    environment:
      - <some configuration>
```

### `docker-compose.service-1.proxy.yml`

```YAML
version: '2'
services:
  service-1:
    build: nginx/
    environment:
      - DEV_HOST=${HOST_IP}
      - DEV_PORT=${PORT}
    ports:
      - "${PORT}:8080"
```

### `docker-compose.override.yml`

```YAML
version: '2'
services:
#  service-1:
#    image: nginx
#    volumes:
#      - ./nginx/default.conf.template:/etc/nginx/conf.d/default.conf.template
#    environment:
#     - DEV_HOST=${HOST_IP}
#     - DEV_PORT=${PORT}
#
  service-2:
    image: nginx
    volumes:
      - ./nginx/default.conf.template:/etc/nginx/conf.d/default.conf.template
    environment:
      - DEV_HOST=${HOST_IP}
      - DEV_PORT=${PORT}
```
