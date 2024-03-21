---
layout: post
title: centos 安装mysql
date: 2018-03-22 09:10:00
tags: centos-mysql
---

注：该文档只适用centos7以下（不包括centos7），安装mysql5.6。文档中的mysql5.6相对于mysql5.7存在一些意想不到的问题。比如int类型的获取到undefined会被mysql处理成default值。


#### 检测系统是否安装mysql
`yum list installed | grep mysql`


#### 安装mysql安装包

```
wget http://repo.mysql.com/mysql-community-release-el6-5.noarch.rpm
rpm -ivh mysql-community-release-el6-5.noarch.rpm
```

<!-- more -->

#### 查看可安装的mysql列表

`yum list|grep mysql`选择需要安装，一般只需要安装client和server即可。
```
sudo yum install mysql-community-server.x86_64 -y
sudo yum install mysql-community-client.x86_64 -y
```

#### 启动mysql
`service mysqld start`


#### mysql设置密码

mysql刚刚安装完的时候，root用户的密码默认是空的。需要用mysql的root用户登录，并修改密码：

```
mysql -uroot -p
use mysql;
update user set password=PASSWORD("输入root用户密码") where User='root';
flush privileges;
```

#### 查看mysql是否自启动,并且设置开启自启动

```
chkconfig --list | grep mysqld
chkconfig mysqld on
```

#### mysql安全设置

`mysql_secure_installation`

ps：在mysql安全设置中，不要允许root用户远程登陆。


#### mysql设置远程登陆用户
```
GRANT ALL PRIVILEGES ON *.* TO admin@"%" IDENTIFIED BY 'password123' WITH GRANT OPTION;
update db set host = '%' where user = 'admin';
```
