# build lighthouse-0.0.762 on Arm64 

This document describes how to build lighthouse-0.0.762 on Arm64. The major reference is the build instruction in https://github.com/jenkins-x/lighthouse#building, which is written for and verified on x86-64.

# Environment
The build is run natively on aarch64 machines. The server used is :

- Memory : 32 G
- CPU : 32 cores

# Prerequisites
* install go version>=1.13.8

  For arm64, follow this link to download Go 1.13.8 

  https://golang.google.cn/dl/go1.13.8.linux-arm64.tar.gz

  Download the archive and extract it.

* install Pre-commit

  I used pip. Other installation methods exist as well.

  `$ pip install pre-commit`

* install Dep

  follow this link to install:

  https://github.com/golang/dep

* update go proxy and environment

  Optionally, set up GOPROXY suitable to your development environment.

  ```
  $ go env -w GO111MODULE=on
  $ go env -w GOPROXY=https://goproxy.io,direct
  ```

* set GOPATH

  `$ export GOPATH=your file`

# Build lighthouse

```shell
$ git clone -b v0.0.0762 https://github.com/jenkins-x/lighthouse.git
$ cd lighthouse
$ make build 
GO111MODULE=on go build -i -ldflags "-X github.com/jenkins-x/lighthouse/pkg/version.Version='0.0.769-dev+a7d56689'" -o bin/webhooks cmd/webhooks/main.go
go: downloading github.com/sirupsen/logrus v1.6.0
go: downloading github.com/jenkins-x/go-scm v1.5.157
go: downloading github.com/prometheus/procfs v0.0.11
go: downloading golang.org/x/sys v0.0.0-20200327173247-9dae0f8f5775
go: downloading github.com/tektoncd/pipeline v0.14.2
go: downloading github.com/mattn/go-zglob v0.0.1
go: downloading github.com/Azure/go-autorest v14.2.0+incompatible
go: extracting github.com/mattn/go-zglob v0.0.1
go: extracting github.com/sirupsen/logrus v1.6.0
go: downloading github.com/gorilla/sessions v1.2.0
......
go: extracting github.com/go-logr/logr v0.1.0
go: extracting gopkg.in/fsnotify.v1 v1.4.7
go: finding sigs.k8s.io/controller-runtime v0.5.0
go: finding github.com/go-logr/logr v0.1.0
go: finding gopkg.in/fsnotify.v1 v1.4.7
GO111MODULE=on go build -i -ldflags "-X github.com/jenkins-x/lighthouse/pkg/version.Version='0.0.769-dev+a7d56689'" -o bin/lighthouse-tekton-controller cmd/tektoncontroller/main.go
GO111MODULE=on go build -i -ldflags "-X github.com/jenkins-x/lighthouse/pkg/version.Version='0.0.769-dev+a7d56689'" -o bin/gc-jobs cmd/gc/main.go
```

Now, all binary file has been built in folder `bin`, you can see them.

```
$ ls -l bin
total 225308
drwxr-xr-x  2 root root     4096 Sep  7 11:13 ./
drwxr-xr-x 12 kong kong     4096 Sep  7 11:12 ../
-rwxr-xr-x  1 root root 50185946 Sep  7 11:12 foghorn*
-rwxr-xr-x  1 root root 38862882 Sep  7 11:13 gc-jobs*
-rwxr-xr-x  1 root root 47533778 Sep  7 11:12 keeper*
-rwxr-xr-x  1 root root 46825787 Sep  7 11:13 lighthouse-tekton-controller*
-rwxr-xr-x  1 root root 47966110 Sep  7 11:12 webhooks*
```

Next, packing these binary files into docker container images.
# Build container images 
The foghorn Dockerfile using ubuntu18.04 as base image

```
FROM ubuntu:18.04
RUN  apt-get update &&  apt install openssh-client  ca-certificates git -y
COPY foghorn /foghorn
RUN mkdir /jxhome
ENV JX_HOME /jxhome
ENTRYPOINT ["/foghorn"]
```

```
$ docker build -f Dockerfile_foghorn -t lighthouse-foghorn:0.0.738 .
```



Other dockerfile is similar.

Here is an example of what I used,You can refer to the following link: https://github.com/yyunk/jenkins-x-arm-support/tree/master/doc/lighthouse       

I built these container images:

```
lighthouse-lighthouse-tekton-controller:0.0.738
lighthouse-webhooks:0.0.738
lighthouse-keeper:0.0.738
lighthouse-foghorn:0.0.738
lighthouse-gc-jobs:0.0.738
```

# Deploy on cluster

```
$ git clone https://github.com/jenkins-x/lighthouse/tree/master/charts/lighthouse
$ cd lighthouse/charts
```

Change the `values.yaml`, modify that yaml file to use the container image you build.

Here is an example of what I used in my arm64 server. You can refer to the following link

https://github.com/yyunk/jenkins-x-arm-support/blob/master/doc/lighthouse/myvalues.yaml

Then run:

`$ helm upgrade lighthouse lighthouse --install --values myvalues.yaml`

# Success criteria
When helm charts can be installed, the pods' status are Running and don't fail / restart for 5 minutes. That means that the pods startup and don't fail basically.
You can use command to see it 

`$ kubectl get pod -w`

```
lighthouse-foghorn-575576ff5d-6j9tz                          1/1     Running            1         1h
lighthouse-keeper-57f7997d86-9bmfm                           1/1     Running            1         1h
lighthouse-tekton-controller-6cb84565f7-xxr7f                1/1     Running            0         1h
lighthouse-webhooks-65c857664b-ttfc9                         1/1     Running            1          1h
```

