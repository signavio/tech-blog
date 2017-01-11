---
title: How to orchestrate microservices with Apacha Kafka in a development environment?
description: Pros and cons to create a docker management tool
tags:
  - docker
  - docker-compose
  - node.js
---


We develop an application that consist of multiple micro services that communicate through Apache Kafka. This is awesome but is really annoying when developing. Our production system uses `kubernetes` and `docker`, so why can we not use the same setup when developing?
Well for one you would have to rebuild the image after changes and restart the cluster.
Gladly there is this nice server called `nginx` (pronounced: 'engine-x', for those like me that didn't know). This server, as a docker image allows us to link our locally running dev server for one service with the other services and Apache Kafka inside docker.

Problem solved!

But wait, there is more.

#### How do we tell `docker` which proxy to use? How does this `nginx` know which ports to forward to where, and which `hostname` to use?

I am so glad you asked.

We use `docker-compose` and the corresponding `docker-compose.yaml` files to tell `docker-compose` what to do.

```
version: 2
services:
  - service-one
and some more code in this block
```
