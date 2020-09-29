# Create bucketrepo container image

This document describes how to build Bucketrepo-0.1.12 on Arm64.

The major reference is the build instruction in 

https://github.com/jenkins-x/bucketrepo which is written for and verified on x86-64.

# Environment

The build is run natively on aarch64 machines. The server used is:   

- Memory : 32 G  

- CPU: 32 cores  

# prerequisites

- install go version>=1.13.8

  For arm64, follow this link to download Go 1.13.8 

  https://golang.google.cn/dl/go1.13.8.linux-arm64.tar.gz

  Download the archive and extract it.

- install Pre-commit

  I used pip. Other installation methods exist as well.

  `$ pip install pre-commit`

- install Dep

  https://github.com/golang/dep

- update go proxy and environment

  Optionally, set up GOPROXY suitable to your development environment.

  ```
  $ go env -w GO111MODULE=on
  $ go env -w GOPROXY=https://goproxy.io,direct
  ```

- set GOPATH

  `$ export GOPATH=your file`

# Build Bucketrepo

```
$ git clone https://github.com/jenkins-x/bucketrepo.git
$ make build  
CGO_ENABLED=0 GO111MODULE=on go build -ldflags '' -o bin/bucketrepo ./internal
go: downloading gocloud.dev v0.0.0-20200414034852-8076239640d2
go: downloading github.com/TV4/logrus-stackdriver-formatter v0.1.0
go: downloading github.com/sirupsen/logrus v1.4.2
go: downloading k8s.io/helm v2.15.2+incompatible
go: extracting github.com/sirupsen/logrus v1.4.2
go: downloading github.com/julienschmidt/httprouter v1.2.0
go: downloading github.com/spf13/viper v1.4.0
go: extracting github.com/julienschmidt/httprouter v1.2.0
go: extracting github.com/spf13/viper v1.4.0
go: downloading github.com/spf13/pflag v1.0.3
go: extracting github.com/spf13/pflag v1.0.3
go: downloading gopkg.in/yaml.v2 v2.2.4
go: downloading github.com/magiconair/properties v1.8.0
go: extracting gopkg.in/yaml.v2 v2.2.4
go: extracting github.com/magiconair/properties v1.8.0
go: extracting github.com/TV4/logrus-stackdriver-formatter v0.1.0
go: downloading github.com/mitchellh/mapstructure v1.1.2
go: downloading github.com/pelletier/go-toml v1.2.0
go: extracting github.com/mitchellh/mapstructure v1.1.2
go: extracting github.com/pelletier/go-toml v1.2.0
go: downloading golang.org/x/sys v0.0.0-20190826190057-c7b8b68b1456
...
go: finding github.com/cyphar/filepath-securejoin v0.2.2
go: finding github.com/googleapis/gax-go v2.0.2+incompatible
go: finding golang.org/x/net v0.0.0-20200202094626-16171245cfb2
go: finding github.com/golang/groupcache v0.0.0-20190702054246-869f871628b6
go: finding k8s.io/apimachinery v0.0.0-20191102025618-50aa20a7b23f
go: finding k8s.io/client-go v11.0.0+incompatible
go: finding golang.org/x/crypto v0.0.0-20190605123033-f99c8df09eb5
```

Now, the binary files are built in folder `bin`. you can see it.

```
$ ls -l bin
total 28036
-rwxr-xr-x 1 root root 28706255 Sep 22 11:13 bucketrepo
```

Now, bucketrepo binary file has been built.

Next, building container images, then deploying bucketrepo with helm.

# Build container images 

The buckerrepo dockerfile

```dockerfile
FROM alpine:latest
RUN apk --no-cache add ca-certificates
COPY ./config/config.yaml /
COPY ./bin/ /
ENTRYPOINT ["/bucketrepo"]
```

```
$ docker build -t buckerrepo:0.1.12 .
```


Here is an example of what I used, You can refer to the following link: https://github.com/yyunk/jenkins-x-arm-support/tree/master/doc/bucketrepo

# Deploy on cluster

Deployment is based on Helm. Helm chart can be found here: https://github.com/jenkins-x/bucketrepo/tree/master/charts/bucketrepo

Change the `values.yaml`, modify that yaml file to use the container image you build.

Here is an example of what I used in my arm64 server. You can refer to the following link: https://github.com/yyunk/jenkins-x-arm-support/blob/master/doc/bucketrepo/myvalues.yaml

Then run:

`$ helm upgrade buckerrepo buckerrepo --install --values myvalues.yaml`

# Success criteria

When helm charts can be installed, the pod's status is Running and don't fail / restart for 5 minutes. That means that the pods startup and don't fail basically. You can use command to see it

`$ kubectl get pod -w`

```
bucketrepo-bucketrepo-66d8b58975-v52ng                       1/1     Running            1          1h
```





