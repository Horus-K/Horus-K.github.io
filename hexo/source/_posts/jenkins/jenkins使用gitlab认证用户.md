---
title: jenkins使用gitlab认证用户
date: 2020-05-10 20:00:29
tags:
categories: jenkins
---



进入setting/application

<!--more-->

![1589113026824](jenkins使用gitlab认证用户/1589113026824.png)

![1589119336351](jenkins使用gitlab认证用户/1589119336351.png)

`http://192.168.220.104:8080/securityRealm/finishLogin`   换为jenkins地址

jenkins安装插件

![1589113812654](jenkins%E4%BD%BF%E7%94%A8gitlab%E8%AE%A4%E8%AF%81%E7%94%A8%E6%88%B7/1589113812654.png)

先让用户可以做任何事



![1589161616254](jenkins使用gitlab认证用户/1589161616254.png)

![1589168928317](jenkins使用gitlab认证用户/1589168928317.png)

重新登录后即可使用gitlab用户认证

登录后权限认证可以改为Role-based

# 如果失败

更新 Jenkins 的 config.xml 文件，把 useSecurity 改为 false，重启 Jenkins 生效。Jenkins 就可以正常访问了。