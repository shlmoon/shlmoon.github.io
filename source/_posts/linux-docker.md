---
layout: post
title: django-docker镜像实践
date: 2018-11-25 10:10:00
tags: centos-docker
---


#### linux server安装docker ce
参考：[阿里云安装Docker CE](https://yq.aliyun.com/articles/110806?spm=5176.8351553.0.0.37161991kq7Vrm)
推荐手动安装，按需决定是否更改镜像源。
<!-- more -->

#### linux server安装docker-compose

centos 安装docker-compose，建议通过pip安装。当然亦可以直接下载，安装docker-compose注意版本。
```
sudo curl -L https://github.com/docker/compose/releases/download/1.13.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
```


#### 镜像打包
  * 本地下载镜像 `docker pull centos`
  * 创建项目Dockerfile
  ```
  FROM centos
  RUN mkdir -p /apps/web/leetee
  ADD . /apps/web/leetee
  ```
  * 进入容器 `docker run -it dockerimage_id /bin/bash`
  * 容器内部安装项目需要的环境，创建非root用户，并设置权限。requirements.txt
    ```
    amqp==2.3.2
    anyjson==0.3.3
    appnope==0.1.0
    backcall==0.1.0
    billiard==3.5.0.4
    celery==4.2.0
    certifi==2018.4.16
    chardet==3.0.4
    decorator==4.3.0
    Django==2.0.4
    django-extensions==2.1.3
    django-grappelli==2.12.1
    djangorestframework==3.8.2
    docutils==0.14
    gevent==1.3.7
    greenlet==0.4.15
    idna==2.6
    ipython==6.4.0
    ipython-genutils==0.2.0
    jedi==0.12.0
    kombu==4.2.0
    meld3==1.0.2
    parso==0.2.1
    pexpect==4.6.0
    pickleshare==0.7.4
    Pillow==5.3.0
    prompt-toolkit==1.0.15
    ptyprocess==0.5.2
    Pygments==2.2.0
    pytz==2018.4
    redis==2.10.6
    requests==2.18.4
    simplegeneric==0.8.1
    simplejson==3.16.0
    six==1.11.0
    supervisor==4.0.0.dev0
    traitlets==4.3.2
    urllib3==1.22
    uWSGI==2.0.17.1
    uwsgitop==0.10
    vine==1.1.4
    wcwidth==0.1.7

    ```
  * 删除项目文件 `rm -rf /apps/web/leetee/*`
  * 镜像打包 `docker commit -a '.' -m '.' container_id shllmoon/leetee`
  * 镜像上传 `docker push shllmoon/leetee`



#### 镜像生成
  * 个人使用的是docker官方地址`https://registry.docker-cn.com`。镜像源：`shllmoon/leetee`

  * uwsgi.ini文件
    ```
    [uwsgi]
    chdir=/apps/web/leetee
    module=leetee.wsgi
    socket=0.0.0.0:8000
    chmod-socket=666
    gevent=100
    max-requests=2000
    master=true
    workers=2
    pidfile=/apps/web/data/uwsgi-leetee.pid
    stats=/apps/web/data/sock-leetee.backend.sock
    disable-logging=true
    vacuum=true
    ```

  * supervisord.conf文件
    ```
    [program:django]
    directory=/apps/web/leetee
    command=/apps/python3/bin/uwsgi --ini /apps/web/leetee/uwsgi.ini
    priority=1
    numprocs=1
    autostart=true
    startsecs=3
    autorestart=unexpected
    exitcodes=0
    ;redirect_stderr=true
    stdout_logfile_maxbytes=500MB
    stdout_logfile_backups=20
    stderr_logfile=/apps/web/data/logs/web.log
    stdout_logfile=/apps/web/data/logs/web.log

    [group:web_server]
    programs=django

    [unix_http_server]
    file=/apps/web/data/supervisor.sock
    chmod=0777
    username=apps
    password=apps123

    [supervisord]
    logfile=/apps/web/data/logs/supervisord_shop_log.log
    stderr_logfile=/apps/web/data/logs/supervisord_err.log
    stdout_logfile=/apps/web/data/logs/supervisord_out.log
    pidfile=/apps/web/data/supervisord.pid

    [supervisorctl]
    serverurl=unix:///apps/web/data/supervisor.sock

    [rpcinterface:supervisor]
    supervisor.rpcinterface_factory=supervisor.rpcinterface:make_main_rpcinterface

    ```

  * Dockerfile文件
    ```
    FROM shllmoon/leetee

    ```

  * docker-compose.yml文件
    ```
    version: '2'
    services:
      python:
        build: .
        privileged: true
        user: ubuntu

        ports:
          - "8000:8000"

        volumes:
          - "/apps/web/www/leetee/leetee:/apps/web/leetee"
          - "/apps/web/www/leetee/logs:/apps/web/data/logs"

        command: "/apps/python3/bin/supervisord -n -c /apps/web/leetee/supervisord.conf"
        restart: always
    ```

ps：
使用该镜像需要注意：
  1. 系统存在ubuntu用户，并且ubuntu用户的uid和gid都为500
  ```
  usermod -u 500 ubuntu
  groupmod -g 500 ubuntu
  ```
  2. ubuntu用户需要有写log目录的权限 `sudo chown -R ubuntu:ubuntu /apps/web/www/leetee/logs`


