---
layout: post
title: nginx web page 404
date: 2019-12-02 09:10:00
tags: nginx-webpage
---


#### nginx web page 404

```
    location / {
        root   /file_path;
        index  index.html index.htm;
        try_files $uri $uri/ @rewrites;
    }
    location @rewrites {rewrite ^(.+)$ /index/index.html last;}

```
<!-- more -->