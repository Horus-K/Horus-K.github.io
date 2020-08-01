---
title: gitlab克隆项目连接为容器名
date: 2020-05-11 14:05:10
tags:
categories: git
---

用docker启动的gitlab克隆项目连接为容器名

解决：

重新初始化容器

```sh
docker run -d \
    --hostname gitlab.qh.com \
    -p 443:443 -p 8080:80 -p 2222:22 \
    --name gitlab \
    registry.cn-hangzhou.aliyuncs.com/imooc/gitlab-ce:latest
```

改ssh端口

```sh
vim /etc/gitlab/gitlab.rb
gitlab_rails['gitlab_shell_ssh_port'] = 2222
```


