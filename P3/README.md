# JxOnArm

# Introduction of P3.1

P3.1 is a simple test for lighthouse docking with Tekton. The process of testing is to change the code repository and automatically trigger the pipeline. The reference link is as follows https://github.com/jenkins-x/lighthouse/blob/master/docs/install_lighthouse_with_tekton.md

## preparation

* Deploy kubernetes cluster with one master and two nodes.
* Deploy the basic components, such as helm3, local docker registry
* Build the image of lighthouse for arm64 architecture. The reference link is as follows: https://github.com/jenkins-x/jenkins-x-arm-support/blob/master/doc/Build%20lighthouse-0.0.762%20on%20Arm64.md
* Build the image of tekton for arm64 architecture. The reference link is as follows: https://github.com/jenkins-x/jenkins-x-arm-support/blob/master/doc/Build%20tekton-v0.15.2%20on%20Arm64.md
* Build the image of kaniko for arm64 architecture. The reference link is as follows: https://github.com/jenkins-x/jenkins-x-arm-support/blob/master/doc/Build%20kaniko-v0.20.0%20on%20Arm64.md
* Build the image of bucketrepofor arm64 architecture. The reference link is as follows: https://github.com/jenkins-x/jenkins-x-arm-support/blob/master/doc/Build%20bucketrepo-0.1.12%20on%20Arm64.md

## Operation process

According to the official lighthouse documentation, install and deploy the lighthouse. The reference link is as follows https://github.com/jenkins-x/lighthouse#installing. 

According to the official tekton documentation, install and deploy the tekton. The reference link is as follows https://github.com/tektoncd/pipeline/blob/master/docs/install.md. Or deploy tekton according to the charts provided by the jenkins-x team. https://github.com/jenkins-x-charts/tekton.

After the two are deployed, refer to the official lighthouse tutorial to configure the docking. https://github.com/jenkins-x/lighthouse/blob/master/docs/install_lighthouse_with_tekton.md

## Testing process

Create a golang code repository, the code is the simple golang web hello-world.Then writing a pipeline, the process is to pull the code, compile the code, use kaniko to build a container image and push it to a nexus container registry. After completing the configuration and docking, set up webhook in the golang code repository. Then change the configmap of config and plugins which in lighthouse config repository.

After that, perform corresponding operations on the code repository to trigger the pipeline. If the pipeline can run successfully, the P3.1 is finished.

The golang demo pipeline link is as follows https://github.com/jincheng-wu/JxOnArm/tree/main/P3.1%20Simple%20tekton%20CI/tekton-pipeline

# Introduction of P3.2

We now have an t1 environment composed of lighthouse, tekton, kaniko, nexus. We want to use this environment deploy another same t2 environment. Then, we will use t2 environment to run an simple golang demo like P3.1. P3.2 consists of two parts, CI and CD

## Specific process of CI

Write CI pipelines, with these, the arm images of lighthouse, tekton, knaiko, bucketrepo are automatically constructed.

Write general shell script to reuse in pipeline. The pipeline's process is to pull the code, compile the code, use kaniko to build a container image. For example, we clone the lighthouse source code, make build, then use kaniko to build a container image and push the image to nexus docker registry. Then, we deploy the lighthouse in the t2 namespace with helm chart. 

The additional profile contains the path of binary file, the path of dockerfile, the name of container image, they are split by commas. The last but not the least, the path of binary file in the dockerfile, it is in a separate row.

The docker registry url and image tag are in pipeline parameters. The example of additional profile's link is as follows: https://github.com/yyunk/pipeline/edit/modify-release-v0.15.x/arm_config

The first step of the pipeline is to pull the corresponding code from github. But after the code of the official repository is pulled down, it is not suitable for the machine of arm64 architecture to directly compile build the container image. Therefore, we fork a source code, and then change some of it's contents, such as dockerfile and makefile. We use this repository as the main code repository of this project. The github repositorys' links are as follows:

https://github.com/yyunk/lighthouse/tree/modify-0.0.748

https://github.com/yyunk/pipeline/tree/modify-release-v0.15.x

https://github.com/yyunk/bucketrepo/tree/modify-0.1.12

* The lighthouse CI task is as follows: https://github.com/jincheng-wu/JxOnArm/blob/main/P3.2%20tekton%20CICD/task/lighthouse_CD_task.yaml

* The tekton CI task is as follows: https://github.com/jincheng-wu/JxOnArm/blob/main/P3.2%20tekton%20CICD/task/tekton_task.yaml

* The kaniko CI task is as follows: https://github.com/jincheng-wu/JxOnArm/blob/main/P3.2%20tekton%20CICD/task/kaniko_task.yaml

* The bucketrepo CI task is as follows: https://github.com/jincheng-wu/JxOnArm/blob/main/P3.2%20tekton%20CICD/task/bucketrepo_task.yml

## Specific process of CD

On the basis of CI pipelines, writing CD pipeline. So that the components can be deployed in the kubernetes cluster in another namespace. We use an image with helm CLI and kubectl CLI. so we can use helm to deploy the components. 

* The lighthouse CD task is as follows: https://github.com/jincheng-wu/JxOnArm/blob/main/P3.2%20tekton%20CICD/task/lighthouse_CD_task.yaml

* The tekton CD task is as follows: https://github.com/jincheng-wu/JxOnArm/blob/main/P3.2%20tekton%20CICD/task/tekton_CD_task.yaml

* The bucketrepo CDtask is as follows: https://github.com/jincheng-wu/JxOnArm/blob/main/P3.2%20tekton%20CICD/task/bucketrepo_CD_task.yml

## Test

After the new environment deployed, we will test it.

The test process is as the same sa the P3.1 simple demo test. 

## Issues

1. Some resources are global, such as CRD, clusterrole, so if we want to deploy components in two namespace, we should change the charts values.
2. The images which are built by kaniko are not able to use successfully. These storage is less than images which are built by docker, and will miss command line tool such as `git`even if i hava installed it in dockerfile.
