---
layout: post
title: nginx log tar
date: 2020-02-22 09:10:00
tags: linux-tips
---

### shell script

```
#!/bin/bash
logroot="/usr/servers/nginx/logs"
hisroot=$logroot/log-history
sudo mkdir -p $hisroot
day=$(date '+%Y%m%d')
aclogname="${day}-access.log"
erlogname="${day}-error.log"
cat $logroot/api-access.log > $hisroot/$aclogname
cat $logroot/api-error.log > $hisroot/$erlogname
sudo truncate -s 0 $logroot/api-access.log
sudo truncate -s 0 $logroot/api-error.log

```
<!-- more -->
### crontab 命令
```
59 23 * * * /usr/servers/nginx/logs/log-tar.sh
```
