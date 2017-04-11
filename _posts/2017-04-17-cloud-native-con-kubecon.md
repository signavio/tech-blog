We recently attended Cloud Native Con Europe | KubeCon because we are interested in and using to some degree cloud native technology and services.
Specifically, we wanted to learn more about Kubernetes and containerisation.


What is the [Cloud Native Computing Foundation](https://www.cncf.io/)?
Part of the Linux Foundation, their mission is to:
>create and drive the adoption of a new computing paradigm that is optimized for modern distributed systems environments capable of scaling to tens of thousands of self healing multi-tenant nodes.

Current projects include:

* [Kubernetes](http://kubernetes.io/)
* [Prometheus](https://prometheus.io/) - query language, time series database
* [OpenTracing](http://opentracing.io/)
* [Fluentd](http://www.fluentd.org/)
* [Linkerd](https://linkerd.io/)
* [gRPC](http://grpc.io/)
* [CoreDNS](http://coredns.io/)
* [containerd](http://containerd.io/)
* [rkt](https://github.com/rkt/rkt)

It is actively growing: containerd and rkt were just announced during a conference keynote!

You probably noticed or heard about a recent [AWS outage](https://aws.amazon.com/message/41926/) that seemed to take down a lot of the internet.
Maybe you didn't hear about how some things [were not affected at all because of Kubernetes](https://thenewstack.io/kubernetes-credited-saving-spire-service-s3-outage/).
We think that is pretty cool!

>With modern web services, users expect applications to be available 24/7, and developers expect to deploy new versions of those applications several times a day.
Containerization helps package software to serve these goals, enabling applications to be released and updated in an easy and fast way without downtime.
Kubernetes helps you make sure those containerized applications run where and when you want, and helps them find the resources and tools they need to work.

Sentence about what is containerization?

From an application developers point of view containerisation is extremely useful.
For example I can quickly create, destroy and recreate local instances of e.g. Kafka, PostgreSQL, Zookeeper, Elastic Search, Flink, etc as well as multiple instances of the same technology that do not interfere with each other with too much overhead.
Changing the version of a dependency is easy.
I also like the fact that installation of such dependencies is now effectively managed in code (yaml files) and versioned in git and overall OWNED more by the development team.
Containers are much more lightweight than a full blown operating system - and so a lot faster.
I often have quite a lot of containers running at once on my laptop.
This allows for a much clearer understanding of application dependencies as well as rapid development and prototyping.

Stefan - dev ops / qa side

replication controllers, deamon set, stateful set, non stateful set

what is containerisation?
pid1

WHat did we learn / hear about?
Kubernetes 1.6 (alpha -> beta -> stable)

problems
app developer difficult to use still, steep learning curve

Tooling / supporting eco system
minikube
helm

Where next - for kubernetes, for us
cluster federation

for us
use containerisation more

what is our ground layer?
RancherOS
CoreOS
Ubuntu?
https://cloudplatform.googleblog.com/2017/04/Container-Optimized-OS-from-Google-is-generally-available.html
