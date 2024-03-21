---
layout: post
title: git单操作集锦
date: 2017-09-22 09:10:00
tags: linux-tips
---

### git 合并单个文件

```
git checkout -p master account/api/views.py
```

### git 本地忽略跟踪
```
git update-index --assume-unchanged /path/to/file
```
<!-- more -->