# Build JX-2.1.132  on Arm64

This document describes how to build JX v2.1.132 (the Jenkins X client) on Arm64. The main reference is the build instruction in  [https://jenkins-x.io/community/code/](https://jenkins-x.io/community/code/) , which is written for and verified on x86-64.


# Environment

The build is run natively on Arm64 machines. The server used is : 

- Memory : 64 G
- CPU: 32 cores

# Install Go 1.13.8

This 1.13.8 version is recommended by JX https://jenkins-x.io/community/code/   
For arm64, follow this link to download Go 1.13.8 and get an installation instruction:
https://golang.google.cn/dl/go1.13.8.linux-arm64.tar.gz               
`$  wget -c https://golang.google.cn/dl/go1.13.8.linux-arm64.tar.gz`              
Download the archive and extract it.

# Install Pre-commit

I used pip. Other installation methods exist as well.          

```
$ pip install pre-commit
$ pre-commit --version
pre-commit 1.21.0
```

# Install Dep

https://github.com/golang/dep

`$ sudo apt-get install go-dep`

# Update Go Proxy and Environment
update go proxy and environment.

Optionally, set up GOPROXY suitable to your development environment.

`$ go env -w GO111MODULE=on`               
`$ go env -w GOPROXY=https://goproxy.io,direct`                                

# Set GOPATH

$ export GOPATH=your file

# Build JX

`$ git clone --branch v2.1.132 https://github.com/jenkins-x/jx.git `   
In folder [GOPATH]/src/github.com/jenkins-x/jx, run:   
`$ make build`        

  ```
CGO_ENABLED=0 GO111MODULE=on go build -v -ldflags " -X github.com/jenkins-x/jx/v2/pkg/version.Version=2.1.133-dev+8d57ed445 -X github.com/jenkins-x/jx/v2/pkg/version.Revision=8d57ed445 -X github.com/jenkins-x/jx/v2/pkg/version.BuildDate=2020-08-27T15:58:32Z -X github.com/jenkins-x/jx/v2/pkg/version.GoVersion=1.13.8 -X github.com/jenkins-x/jx/v2/pkg/version.GitTreeState=dirty """ -o build/jx cmd/jx/jx.go
go: downloading github.com/jenkins-x/jx-logging v0.0.10
go: downloading github.com/heptio/sonobuoy v0.16.0
go: downloading github.com/jenkins-x/golang-jenkins v0.0.0-20180919102630-65b83ad42314
go: downloading github.com/banzaicloud/bank-vaults v0.0.0-20191212164220-b327d7f2b681
go: downloading knative.dev/serving v0.16.0
go: downloading k8s.io/apimachinery v0.16.5
go: downloading k8s.io/client-go v0.16.5
go: downloading k8s.io/helm v2.7.2+incompatible
go: extracting github.com/jenkins-x/golang-jenkins v0.0.0-20180919102630-65b83ad42314
go: downloading github.com/google/uuid v1.1.1
go: extracting github.com/jenkins-x/jx-logging v0.0.10
go: extracting k8s.io/helm v2.7.2+incompatible
go: extracting github.com/google/uuid v1.1.1
go: extracting k8s.io/apimachinery v0.16.5
go: downloading github.com/pkg/errors v0.9.1
go: extracting k8s.io/client-go v0.16.5
go: extracting knative.dev/serving v0.16.0
go: downloading github.com/tektoncd/pipeline v0.14.2
go: downloading github.com/rickar/props v0.0.0-20170718221555-0b06aeb2f037
go: downloading golang.org/x/sync v0.0.0-20200317015054-43a5402ce75a
go: downloading github.com/hashicorp/vault v1.2.0-beta2.0.20190725165751-afd9e759ca82
go: downloading golang.org/x/net v0.0.0-20200324143707-d3edc9973b7e
go: extracting github.com/pkg/errors v0.9.1
go: extracting github.com/rickar/props v0.0.0-20170718221555-0b06aeb2f037
go: extracting golang.org/x/net v0.0.0-20200324143707-d3edc9973b7e
go: extracting golang.org/x/sync v0.0.0-20200317015054-43a5402ce75a
go: extracting github.com/tektoncd/pipeline v0.14.2
...
github.com/jenkins-x/jx/v2/pkg/cmd/get
github.com/jenkins-x/jx/v2/pkg/cmd/stop
github.com/jenkins-x/jx/v2/pkg/cmd/step/verify
github.com/jenkins-x/jx/v2/pkg/cmd/deletecmd
github.com/jenkins-x/jx/v2/pkg/cmd/step/create
github.com/jenkins-x/jx/v2/pkg/cmd/gc
github.com/jenkins-x/jx/v2/pkg/cmd/boot
github.com/jenkins-x/jx/v2/pkg/cmd/controller/pipeline
github.com/jenkins-x/jx/v2/pkg/cmd/controller
github.com/jenkins-x/jx/v2/pkg/cmd/step/e2e
github.com/jenkins-x/jx/v2/pkg/cmd/step/bdd
github.com/jenkins-x/jx/v2/pkg/cmd
github.com/jenkins-x/jx/v2/cmd/jx/app
***********
  ```

Now, build finished. You can enter `build` folder to verify it.    
```
$ cd build
$ ./jx version
Version 2.1.132-dev+8d57ed445
Commit 8d57ed445
Build date 2020-08-21T13:47:08Z
Go version 1.13.8
Git tree state dirty
```

# Notes:

1.  You can change the Makefile, add  parameters “-v” to see which package is being compiled:

	```
	BUILDFLAGS := -v -ldflags \
	" -X $(ROOT_PACKAGE)/pkg/version.Version=$(VERSION)\
	-X $(ROOT_PACKAGE)/pkg/version.Revision=$(REV)\
	-X $(ROOT_PACKAGE)/pkg/version.BuildDate=$(BUILD_DATE)\
	-X $(ROOT_PACKAGE)/pkg/version.GoVersion=$(GO_VERSION)\
	-X $(ROOT_PACKAGE)/pkg/version.GitTreeState=$(GIT_TREE_STATE)\
	$(BUILD_TIME_CONFIG_FLAGS)"
	```
	
	
