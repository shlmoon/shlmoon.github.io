---
layout: post
title: GitLab Community Edition
date: 2019-05-12 19:10:00
tags: centos-gitlab
---

注：[gitlab 备份、恢复、迁移](https://gitlab.com/help/raketasks/backup_restore.md)

<!-- more -->

### docker-compose.yml

```
version: '3.6'
services:
  gitlab:
    container_name: gitlab
    image: gitlab/gitlab-ce:latest
    restart: always
    environment:
      GITLAB_OMNIBUS_CONFIG: |
            external_url 'http://gitlab.deno.vip'
    ports:
      - "80:80"
      - "443:443"
      - "22:22"
    volumes:
      - "/data/gitlab/config:/etc/gitlab"
      - "/data/gitlab/logs:/var/log/gitlab"
      - "/data/gitlab/data:/var/opt/gitlab"
```
 * /data 最好是数据盘挂载路径
 * gitlab与宿主机ssh端口最好设置为非22，当然迁移除外。