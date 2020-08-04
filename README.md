# jenkins-x-arm-support
A project to track progress on supporting ARM64 platforms with Jenkins X.

We value respect and inclusiveness and follow the [CDF Code of Conduct](https://github.com/cdfoundation/toc/blob/master/CODE_OF_CONDUCT.md) in all interactions.

Note: Thoughout this project, Arm Server and Arm64 are used equivalently to refer to the architecture of arm64.

Note: Here is the [Jenkins X Enhancement Proposal](https://github.com/jenkins-x/enhancements/issues/33) which is a high level issue to track things

## Goal
The goals of this project are:

 1. To verify the CDF Components and their dependencies execute on Arm servers (aka. arm64), or in Docker containers on Arm servers.  
 2. If a component or dependency needs to be upgraded from the version supplied by the server’s host operating system distribution, we will build it.   If it does not build, we will port the software to successfully build, and deploy it on the ARM server.
 3. To build two proof of concept CI/CD clusters to run in the project cloud with this software:
	a. One cluster using Prow and Jenkins to demonstrate workloads that require the use of a legacy Jenkins server to automate CI/CD pipelines.
	b. Another cluster using Tekton to demonstrate serverless CI/CD pipelines.
 4. To demonstrate a working CI/CD pipeline for Jenkins X on an ARM server based cloud.

## Minimum Set of Components
(To be clarified) The following components are identified as a minimum set to let Jenkins X to run on Arm server:

 - Kubernetes
 - Docker, Docker Registry
 - Jx
 - Helm 2, Tiller
 - Helm 3, 
 - Prow, Lighthouse, Jenkins, Tekton
 - Vault
 - Nexus Repository OSS
 - ChartMuseum, Monocular

## Milestones
(To be clarified)

 - M1: Initial project setup
 - M2: Tool assessment and prepartion, ensure the CDF components and their depenencies can be built for, deployed and run on arm64.
 - M3: Cluster using Prow and Jenkins
 - M4: Serverless pipeline using Tekton
 - M5: Demonstrate workloads on Arm64 CI cluster.

## Slack

If you want to join us on [CDF slack in #jenkins-x-to-arm](https://cdeliveryfdn.slack.com/join/shared_invite/enQtODM2NDI1NDc0MzIxLTA1MDcxMzUyMGU2NWVlNmQwN2M1N2M4MWJjOWFkM2UzMDY0OWNkNjAzNzM0NzVkNjQ5M2NkMmY2MTRkMWY4MWY#/)
