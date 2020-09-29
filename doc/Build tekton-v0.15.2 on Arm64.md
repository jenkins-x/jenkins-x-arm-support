# Build tekton-v0.15.2 on Arm64
This document describes how to build tekton-v0.15.2 on Arm64.

The major references of this document are :

* https://github.com/tektoncd/pipeline/blob/master/docs/install.md

* https://github.com/jenkins-x-charts/tekton/blob/master/tekton/README.md

# Environment
The build is run natively on aarch64 machines. The server used is :

- Memory : 32 G

- CPU : 32 cores

# Prerequisites
* install go version >= 1.13.8

  For arm64, follow this link to download Go 1.13.8 :

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

  ```shell
  $ go env -w GO111MODULE=on
  $ go env -w GOPROXY=https://goproxy.io,direct
  ```

* set GOPATH

  `$ export GOPATH=your file`
# Build pipeline
```shell
$ git clone -b v0.15.2 https://github.com/tektoncd/pipeline.git
$ cd pipeline
```
## Modification of Makefile

Golint is a linter for Go source code, golint prints out style mistakes. At the moment of this writing, Golint cannot be installed successfully. As it's not crucial to the core function of tekton, here I temporarily removed it from Makefile. For more detail about this issue, see  https://docs.google.com/document/d/1vwsii0WeFaGHE92uBTazyVHKT9kIb-pjkdVQ_0Niwcg/edit?usp=sharing

Because some binary can be built not only by main.go, I change the main.go in Makefile to *.go. So it can build binary by all source code include in folder.

Here is an example of what I changed in the Makefile. You can refer to the following link: https://github.com/yyunk/pipeline/commit/5838ed97c142f49e72f2b68cbecd4ff04acfc6fb

## Start the build

After changing the Makefile, you can enter the `cmd` folder, then you can see folders.

__controller, creds-init, entrypoint, git-init, imagedigestexporter, kubeconfigwriter, nop, pullrequest-init, webhook.__

Enter each folder, and enter command `$ make -f ../../Makefile all`, the binary can be built.

Or you can just use shell script 

```$ for i in `realpath cmd/*`; do cd $i; make -f ../../Makefile all; done```

The result binary files are in cmd/*/.bin/github.com/tektoncd/pipeline.

Next step, build gcs-fetcher

```shell
$ cd pipeline/vendor/github.com/GoogleCloudPlatform/cloud-builders/gcs-fetcher/cmd/gcs-fetcher
$ make -f pipeline/Makefile all
```

Now, all binary files are built.

There are 

* pipeline/cmd/controller/.bin/github.com/tektoncd/pipeline

* pipeline/cmd/creds-init/.bin/github.com/tektoncd/pipeline

* pipeline/cmd/entrypoint/.bin/github.com/tektoncd/pipeline

* pipeline/cmd/git-init/.bin/github.com/tektoncd/pipeline

* pipeline/cmd/kubeconfigwriter/.bin/github.com/tektoncd/pipeline

* pipeline/cmd/nop/.bin/github.com/tektoncd/pipeline

* pipeline/cmd/pullrequest-init/.bin/github.com/tektoncd/pipeline

* pipeline/cmd/webhook/.bin/github.com/tektoncd/pipeline

* pipeline/cmd/imagedigestexporter/.bin/github.com/tektoncd/pipeline

* pipeline/vendor/github.com/GoogleCloudPlatform/cloud-builders/gcs-fetcher/cmd/gcs-fetcher/.bin/github.com/tektoncd/pipeline

Next, building container images, then deploying tekton with helm.

# Build container images 
The controller Dockerfile
Using ubuntu18.04 as base image

```
FROM ubuntu:18.04
RUN  apt-get update &&  apt install openssh-client git -y
RUN mkdir /ko-app
COPY ./pipeline  /ko-app/controller
ENTRYPOINT ["/ko-app/controller"]
```
```docker build -f Dockerfile_controller -t controller:0.15.2-arm64 .```    

Other dockerfiles are similar.
Here is an example of what I used, You can refer to the following link: https://github.com/yyunk/jenkins-x-arm-support/tree/master/doc/pipeline

You should build container images by binary files which are built by source code.

I built these container images:

```
git-init:0.15.2-arm64         
webhook:0.15.2-arm64         
pullrequest-init:0.15.2-arm64         
nop:0.15.2-arm64         
kubeconfigwriter:0.15.2-arm64                 
imagedigestexporter:0.15.2-arm64         
controller:0.15.2-arm64         
gcs-fetcher:0.15.2-arm64         
entrypoint:0.15.2-arm64         
creds-init:0.15.2-arm64    
```


# Deploy on cluster

```
$ git clone https://github.com/jenkins-x-charts/tekton 
$ cd tekton/tekton
```

Change the values.yaml, modify that yaml file to use the container image name + tag  for the image you just built on your cluster, such as

```
image:
  upstreamtag: 0.15.2-arm64
  kubeconfigwriter: kubeconfigwriter
  credsinit: creds-init
  gitinit: git-init
  controller: controller
  webhook: webhook
  entrypoint: entrypoint
  pullrequest: pullrequest-init
  imagedigestexporter: imagedigestexporter
  gcsfetcher: gcs-fetcher
```
Here is an example of what I used in my arm64 server. You can refer to the following link: 

https://github.com/yyunk/jenkins-x-arm-support/blob/master/doc/pipeline/myvalues.yaml

`$ helm upgrade tekton tekton --install --values myvalues.yaml`

# Success criteria
When both helm charts can be installed and the `Deployment` resources create pods which startup, start Running and don't fail / restart for 5 minutes. That means that the pods startup and don't fail basically.
You can use command to see it.

`$ kubectl get pod -w` 

```
tekton-pipelines-controller-68487d8d5c-f6q5g                 1/1     Running            0          1h
tekton-pipelines-webhook-6cff968fb9-6b6rm                    1/1     Running            0         1h

```

