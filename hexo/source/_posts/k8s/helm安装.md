---
title: helm安装
date: 2020-04-29 16:23:18
tags: helm
categories: k8s
---

我们可以在Helm Realese⻚⾯下载⼆进制⽂件，这⾥下载的v2.10.0版本，解压后将可执⾏⽂件 helm  拷⻉到 /usr/local/bin  ⽬录即可，这样 Helm  客户端就在这台机器上安装完成了。 <https://github.com/helm/helm/releases>

<!--more-->

现在我们可以使⽤ Helm  命令查看版本了，会提示⽆法连接到服务端 Tiller  ：

```sh
$ helm version
Client: &version.Version{SemVer:"v2.16.2", GitCommit:"9ad53aac42165a5fadc6c87be0dea6b115f9
3090", GitTreeState:"clean"}
Error: could not find tiller
```

要安装 Helm 的服务端程序，我们需要使⽤到 kubectl  ⼯具，所以先确保 kubectl  ⼯具能够正常的访问 kubernetes 集群的 apiserver

然后我们在命令⾏中执⾏初始化操作：

```
$ helm init
```

由于 Helm 默认会去 gcr.io  拉取镜像，所以如果你当前执⾏的机器没有配置科学上⽹的话可以实现下⾯的命令代替：

```sh
$ helm init --service-account tiller --upgrade -i registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.16.6 --stable-repo-url https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
$ helm init --tiller-image 192.168.220.101/library/tiller:v2.16.6-1 --upgrade
```

