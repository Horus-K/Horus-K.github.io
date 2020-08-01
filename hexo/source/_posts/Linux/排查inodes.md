---
title: 排查inodes
date: 2020-03-24 18:27:41
tags:
categories:
---

```
for i in ./*; do echo $i; find $i |wc -l; done
```

循环查询目录文件个数