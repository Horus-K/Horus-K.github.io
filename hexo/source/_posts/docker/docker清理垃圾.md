---
title: docker清理垃圾
date: 2020-05-14 13:46:48
tags:
- 垃圾清理
categories: docker
---

#### 列出无用的卷

```
docker volume ls -qf dangling=true
```

#### 清理无用的卷，容器，镜像

```sh
docker volume rm $(docker volume ls -qf dangling=true)

docker rmi $(docker images | grep '^<none>' | awk '{print $3}')

docker rm $(docker ps -a | grep 'Exited' | awk '{print $1}')

docker images --no-trunc | grep '<none>' | awk '{ print $3 }' | xargs docker rmi

docker system prune

docker volume prune

```

