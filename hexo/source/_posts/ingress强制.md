---
title: ingress强制
date: 2020-08-01 22:04:25
tags:
- Ingress
categories: k8s
---

为ingress配置增加注解（annotations）：nginx.ingress.kubernetes.io/ssl-redirect: 'true' 就可以实现http强制跳转至https

不过默认情况ingress是通过308重定向跳转到https， ie浏览器不一定支持308状态， 可以通过如下方式修改ingress配置，让ingress通过301跳转到https

<!--more-->

1、进入阿里容器服务后台->选择配置项

![img](https://img2018.cnblogs.com/blog/1356274/201906/1356274-20190613202808903-2073258032.png)

 

2、为ingress配置项添加 http-redirect-code = 301 配置。

![img](https://img2018.cnblogs.com/blog/1356274/201906/1356274-20190613202846635-1315563375.png)

 

参考下配置项的文档：https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/configmap/

![img](https://img2018.cnblogs.com/blog/1356274/201906/1356274-20190613202955081-1954547023.png)

 

从上图可以看到这个重定向默认值是 “308”，k8s路由默认http跳转到https， 用的是308跳转，ie浏览器，或者有些低版本的浏览器不支持“308”跳转的，要改成“301”跳转，不然低版本的浏览器会报错。