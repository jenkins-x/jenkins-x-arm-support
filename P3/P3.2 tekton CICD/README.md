# P3.2 tekton CICD

P3.2使用了 GitHub, Lighthouse, Tekton, Kaniko，nexus组成了一套pipeline，目标为用tekton自身部署自身。

## github

首先需要从github上面拉取相对应的代码。但官方仓库的代码拉取下来后，直接编译制作镜像无法适用于arm64架构的机器，因此我们首先fork了一份源码，再更改他的部分内容，例如dockerfile，makefile。最后将我们所fork的仓库作为本次项目的主代码仓。

## lighthouse

首先依照lighthouse官方文档，安装部署lighthouse https://github.com/jenkins-x/lighthouse。

完成安装后，在github项目上设定一个webhook，测试他与github是否能正常的通信。

## tekton

首先依照tekton官方文档，安装部署tekton https://github.com/tektoncd/pipeline/blob/master/docs/install.md 或依照jenkins-x团队提供的charts，部署tekton https://github.com/jenkins-x-charts/tekton

完成安装后，编写一个小demo测试pipeline是否顺利运行。参考官方文档 https://github.com/tektoncd/pipeline/blob/master/docs/tutorial.md

测试成功后，将lighthouse与tekton进行配置，使lighthouse可以顺利运行pipeline。参考官方文档 https://github.com/jenkins-x/lighthouse/blob/master/docs/install_lighthouse_with_tekton.md

## kaniko

使用之前制作的kaniko镜像，完成pipeline的流程。kaniko具体使用方法参考官方文档 https://github.com/GoogleContainerTools/kaniko

## nexus

nexus官方并未提供arm64版本镜像，使用之前制作的镜像，启动nexus。开启了docker registry的功能，使得kaniko制作的镜像能够推送到nexus处。

