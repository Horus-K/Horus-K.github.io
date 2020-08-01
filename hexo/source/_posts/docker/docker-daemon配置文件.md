---
title: docker-daemon配置文件
date: 2020-06-12 09:40:14
tags: docker-daemon
categories: docker
---

```
{
    "experimental": false,
    "registry-mirrors": [
        "http://bc437cce.m.daocloud.io",
        "https://docker.mirrors.ustc.edu.cn/",
        "https://reg-mirror.qiniu.com",
        "https://mirror.ccs.tencentyun.com",
        "https://hub-mirror.c.163.com"
  ],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "debug": true
}
```

