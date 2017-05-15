---
author: Christian Kniep
layout: post
title: "Container 'Build' Manifesto : Image Size & Layering"
date: 2017-05-07
tags: container manifesto, build
---

*This is a guest post written by [Christian Kniep](http://qnib.org/about/) who presented at the [Signavio Tech MeetUp on April 20th](https://www.meetup.com/de-DE/Signavio-Tech-Talks/events/237960063/) where Christian  introduced some guidelines for building and running containers.
The post appeared first on [qnib.org](http://qnib.org/2017/05/07/container-manifesto-1-build/)*

During the [April MeetUp](https://www.meetup.com/M-E-L-I-G-Berlin-Metrics-Events-Logs-Inventory-Glue/events/238948734/) I introduced some guidelines about how to build and run containers, the talk was recorded and is available on youtube:

<iframe width="560" height="315" src="https://www.youtube.com/embed/DPI1eAgts0w?ecver=1" frameborder="0" allowfullscreen></iframe>

From my experience it is way to easy to stick with the virtual machine mindset when first starting with containers. It won`t cause a problem when one gets his feet wet - until it does and begins to hunt you with subtle problem scenarios. Even more so, when one shifts from pure containers to orchestrated services using Docker SWARM or Kubernetes.

To share what I have learned in the four years of tinkering with Linux Containers (started with Docker 0.7 in 2013), this blog post will cover the first part of the Manifesto: [Image Size](#image-size) and [Layering](#layering).

Most of the points are targets with wiggle room like the size of a container image. 0 Byte would be the perfect size, but won`t result in something useful. 
I marked them **desirable** to indicate that they are a goal which one should thrive for, but you can start tinkering around ignoring it.
**BUT(!1!!)**, please keep in mind that you are in a danger zone leading to misery and pain in scenarios, unlike your laptop or big workstation. Scenarios in which you want to interact with other containers or services.
I am advising you to honour the guidelines as soon as you can or at least be aware of the consequences. :)

### [**Desirable**] Avoid Big Container Images
<a name="image-size"/></a>
**TL;DR** Start with the smallest image possible and only add what is really necessary to support your application at runtime.
 
One could create a container image to run `htop` like this:

```
FROM ubuntu:14.04

RUN apt-get update
RUN apt-get install -y htop
CMD ["htop"]
```

But I guess we agree that this is a bit insane, as you use a full -fledged operating system (`ubuntu:14.04` is almost 200MB big) to run only the process `htop`.

The big end goal is to run an empty user-land with a precompiled binary.

```
$ tar cv --files-from /dev/null | docker import - scratch
sha256:864c85770a1d9e585b93a84f24607a0b1d0e87dfa0b9e000aef5c20925557f73
$ FROM scratch

COPY go-test  /
ENTRYPOINT ["/go-test"]
```

To get this done in one go, use multi-staged builds (needs docker-engine >= 17.05).

```
$ git clone https://github.com/qnib/go-test.git ; cd go-test
$ cat Dockerfile
FROM qnib/uplain-golang

WORKDIR /usr/local/src/github.com/qnib/go-test/
COPY vendor/vendor.json ./vendor/vendor.json
COPY main.go ./main.go
RUN govendor fetch +m \
 && govendor build
 
FROM scratch

COPY --from=0 /usr/local/src/github.com/qnib/go-test/go-test  /
ENTRYPOINT ["/go-test"]
$ docker build -t qnib/go-test                                                                                                                           git:(master|)
> docker build 'qnib/go-test'
Sending build context to Docker daemon  1.985MB
Step 1/8 : FROM qnib/uplain-golang
 ---> 804bc466b7d8
Step 2/8 : WORKDIR /usr/local/src/github.com/qnib/go-test/
 ---> a103ad6f3237
Removing intermediate container dcdfc423beb8
Step 3/8 : COPY vendor/vendor.json ./vendor/vendor.json
 ---> 5a9d7fe1e7af
Removing intermediate container 1b61f2650da4
Step 4/8 : COPY main.go ./main.go
 ---> 803fb0c76b0e
Removing intermediate container b5834e34b7f5
Step 5/8 : RUN govendor fetch +m  && govendor build
 ---> Running in 60439f83d009
 ---> c4c82b13d1a4
Removing intermediate container 60439f83d009
Step 6/8 : FROM scratch
 --->
Step 7/8 : COPY --from=0 /usr/local/src/github.com/qnib/go-test/go-test /
 ---> 204705e5c0dd
Removing intermediate container 912feecb660a
Step 8/8 : ENTRYPOINT /go-test
 ---> Running in 780f0ac65d25
 ---> c1d640aa9f67
Removing intermediate container 780f0ac65d25
Successfully built c1d640aa9f67
Successfully tagged qnib/go-test:latest
$ docker run -t qnib/test
mySum(1,2) = 3
myPosSub(20,5) = 15
myMultiply(2,2) = 4
$ docker images |grep qnib/test
qnib/test     latest      3893b9a2513b        About a minute ago   1.56MB
```

This bugger is only 1.5MB in size, easily downloaded by any device, even on slow Internet connections.

**Take a breath** and make sure you inhale what just happened here. By using multi-stage build, your building container image can be as big as you like, the resulting artifact (the binary, JAR, ...) will just be copied over (`COPY --from=0`) in a subsequent step using a stripped down parent.

A good starting point, with a decently stripped down user-land, is Alpine Linux (`docker pull alpine`). 
The base image has only 5MB, even though it is a full OS with package manager (`apk --no-cache add vim`) and alike.
Careful though, it is not based on glibc, which might cause some trouble - depending on the app.

### [**Desirable**] Leverage Layering
<a name="layering"/></a>

**TL;DR** By using multiple layers (steps in the Dockerfile) when building an image, caching can be leveraged when building and downloading images.

A Dockerfile like this...

```
FROM alpine:3.5

RUN echo "123" > /etc/mya.txt \
 && echo "234" > /etc/myb.txt \
 && echo "345" > /etc/myc.txt \
 && echo "456" > /etc/myd.txt \
 && echo "567" > /etc/mye.txt \
 && rm -f /etc/myc.txt
```

...was considered cool at some point, because it only creates one layer, which won`t include `/etc/myc.txt`, as it is created but ultimately removed at the end. 
This might be hip again, when we are looking into squashing builds!

The downside is, that the files `my{a..e}` are not reusable in other images - only if the exact outcome is provided.

Let's imaging the following...

```
FROM alpine:3.5

RUN echo "123" > /etc/mya.txt
RUN echo "234" > /etc/myb.txt
RUN echo "345" > /etc/myc.txt
RUN echo "456" > /etc/myd.txt
RUN echo "567" > /etc/mye.txt
```

The following image has nothing to build, as it can just leverage the layers of the first image.

```
FROM alpine:3.5

RUN echo "567" > /etc/mye.txt
RUN echo "456" > /etc/myd.txt
RUN echo "345" > /etc/myc.txt
```

And this is also true, when pulling/pushing an image, as the layers are content addressed, the docker-engine will reuse the layers from the first image. In the example the layers are not big in size, but replace `echo 123` by a blob of 100MB.

In a more real life example, it might make a lot of sense to group installations logically, so that they are reusable.
My rule-of-thumb is to ask: are they independent or will a separat step increase the size of the resulting image?

```
FROM ubuntu:16.04

RUN apt-get update \
 && apt-get install -y openjre8 \
 && apt-dist clean
 
RUN apt-get update \
 && apt-get install -y nginx \
 && apt-dist clean
```

The Dockerfile might look suspicious, because it repeats `apt-get update` and `apt-dist clean` in both steps and didn't we all learn DRY (Don't Repeat Yourself)?

But this repetition only takes place at build-time, not to mention that the individual step will just install the content and (hopefully) removes the intermediate files used by the package manager; hence, the outcome should be the same - but in two different file-system layers.

When building multiple images like that, the download is sped up, because (hopefully) a lot of layers are shared.
This images:

```
FROM ubuntu:16.04

RUN apt-get update \
 && apt-get install -y nginx \
 && apt-dist clean
```

```
FROM ubuntu:16.04

RUN apt-get update \
 && apt-get install -y postgres \
 && apt-dist clean
 
RUN apt-get update \
 && apt-get install -y nginx \
 && apt-dist clean
```

... combined with the first image, result in three file-system layers (`nginx`, `postgres` and `openjre8`) and are going to be reused when downloading. If one would have use single-layer images each file-system layer is different and can not be reused.

Furthermore caching the steps benefits when reruning the build, while only changing the second step. The first one won`t be repeated. A general rule I follow is to position the steps according to their expected change rate (stable stuff on top) and the duration (build time, download time) of the step.

## Next Steps

Having looked at _Image size & layering_, the next steps are to look at build and runtime guidelines.
Here are my build guidelines.

* **Idea** Leverage Squashing (?)
* **Strict** Avoid Multi-Version Image
* **Strict** Do not use `docker commit`
* **Strict** Avoid Use of Tag `:latest`
* **Desirable** Sane Defaults and docker-compose file
* **Desirable** Design for Smallest-Scale and Distributed Deployment
* **Desirable** Use `pre-run` entrypoint
* **Desirable** Use `.dockerignore` and `--compress`

And the runtime parts:

* **Strict** Keep Containers Emphemeral
* **Strict** One Process per Container
* **Desirable** Fast Container Startup
* **Desirable** No Assumptions! (like IP addresses)
* **Strict** Configuration via Environment / Secrets
* **Strict** Unprivileged users
* **Idea** Use read-only container file-system