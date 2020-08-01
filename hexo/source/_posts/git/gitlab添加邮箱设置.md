---
title: gitlab添加邮箱设置
date: 2020-05-12 17:34:07
tags:
categories: gitlab
---

## 修改配置文件

```
[root@localhost opt]# grep -rn "^[^#]" /etc/gitlab/gitlab.rb 
external_url 'http://192.168.0.8:8080' #如果使用容器要对应端口
gitlab_rails['gitlab_email_from'] = '16253173030@qq.com'
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.qq.com"
gitlab_rails['smtp_port'] = 465
gitlab_rails['smtp_user_name'] = "16253173030@qq.com"
gitlab_rails['smtp_password'] = "###授权码###"
gitlab_rails['smtp_domain'] = "smtp.qq.com"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = true
```

<!--more-->

## 加载配置文件并重启gitlab

```sh
gitlab-ctl reconfigure
gitlab-ctl restart
```

## 验证

```
irb(main):002:0> Notify.test_email('1625317303@qq.com', 'GitLab email', 'Hellow world').deliver_now

Notify#test_email: processed outbound mail in 0.7ms

Sent mail to 1625317303@qq.com (790.4ms)
Date: Tue, 12 May 2020 09:34:05 +0000
From: GitLab <1625317303@qq.com>
Reply-To: GitLab <noreply@192.168.220.100>
To: 1625317303@qq.com
Message-ID: <5eba6d8dbcbca_d9eb3fd2410c70d07652e@gitlab.qh.com.mail>
Subject: GitLab email
Mime-Version: 1.0
Content-Type: text/html;
 charset=UTF-8
Content-Transfer-Encoding: 7bit
Auto-Submitted: auto-generated
X-Auto-Response-Suppress: All

<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.0 Transitional//EN" "http://www.w3.org/TR/REC-html40/loose.dtd">
<html><body><p>Hellow world</p></body></html>

=> #<Mail::Message:70172128827780, Multipart: false, Headers: <Date: Tue, 12 May 2020 09:34:05 +0000>, <From: GitLab <1625317303@qq.com>>, <Reply-To: GitLab <noreply@192.168.220.100>>, <To: 1625317303@qq.com>, <Message-ID: <5eba6d8dbcbca_d9eb3fd2410c70d07652e@gitlab.qh.com.mail>>, <Subject: GitLab email>, <Mime-Version: 1.0>, <Content-Type: text/html; charset=UTF-8>, <Content-Transfer-Encoding: 7bit>, <Auto-Submitted: auto-generated>, <X-Auto-Response-Suppress: All>>
irb(main):003:0> exit
```

