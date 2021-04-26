# Couchbase GitPod Example

This repository is an example of how you can use Couchbase inside [Gitpod](https://gitpod.io/). 

## How to run it

Simply access the following URL https://gitpod.io/#https://github.com/couchbaselabs/CouchbaseGitPodExample.git


## How it Works

There are 2 important files on this project: *.gitpod.Dockerfile* and *.gitpod.yml*


### .gitpod.Dockerfile

This file is the base image used to run your workspace on GitPod. I have already generate an image called *deniswsrosa/couchbase6.6.2-gitpod* that you can extend and install all the extra packages you need. For instance:

```
FROM deniswsrosa/couchbase6.6.2-gitpod

#Simple example on how to extend the image to install Java and maven
RUN apt-get -qq update && \
     apt-get install -yq maven default-jdk
```

In the file Couchbase 6.6.2 has already been previously installed, and as we need maven and JDK to run a simple java app, I'm extending the image and installing these two packages.


### .gitpod.yml

This file allows you to configure your workspace on GitPod [read mode about it here](https://www.gitpod.io/docs/config-gitpod-file/).
In our case, it looks like the following:

```
image:
  file: .gitpod.Dockerfile

tasks:
- name: Start Couchbase
  before:  cd /opt/couchbase/ && ./start-cb.sh && sleep 20
- name: Create Bucket
  init: while [[ "$(curl -s -o /dev/null -w ''%{http_code}'' -u Administrator:password localhost:8091/pools/default)" != "200" ]]; do sleep 5; done && couchbase-cli bucket-create -c localhost:8091 --username Administrator --password password --bucket default --bucket-type couchbase --bucket-ramsize 256
# exposed ports
ports:
- port: 8091
  onOpen: notify
- port: 8092-10000
  onOpen: ignore
- port: 4369
  onOpen: ignore

vscode:
  extensions:
    - redhat.java
    - vscjava.vscode-java-debug
    - vscjava.vscode-java-test
    - pivotal.vscode-spring-boot
```

First, we need to specify the docker image that we want to use (.gitpod.Dockerfile in this case).
Then, we specify 2 tasks called "Start Couchbase" and "Create Bucket":

#### Task: Start Couchbase

In the base image we added a script that starts couchbase manually and init the cluster with the user "Administrator" and password "password".
You can't use the documented ways of starting Couchbase, as gitpod overwrites some files while applying its layer on the top of the image that we specified.

#### Task: Create Bucket
On this task, we wait until the cluster is initialized and then create a bucket called "default", you can leverage this task to also load data into Couchbase.


