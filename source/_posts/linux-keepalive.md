---
layout: post
title: linux close_wait
date: 2019-06-22 09:10:00
tags: linux
---


### 前置环境
* Azure
* centos7.6
* python3.6.5
* tornado==5.0.2
* Tornado-MySQL==0.5.1
* tornado-redis==2.4.18

<!-- more -->
### 事故场景
最终的结果是浏览器所有的请求都是pending的状态。
通过`lsof -i:9000`查看: 

```
python3 105792 apps    5u  IPv4 2782706      0t0  TCP *:cslistener (LISTEN)
python3 105808 apps    5u  IPv4 2782706      0t0  TCP *:cslistener (LISTEN)
python3 105808 apps    9u  IPv4 2791131      0t0  TCP localhost:cslistener->localhost:57550 (CLOSE_WAIT)
python3 105809 apps    5u  IPv4 2782706      0t0  TCP *:cslistener (LISTEN)
python3 105809 apps    9u  IPv4 2791129      0t0  TCP localhost:cslistener->localhost:57548 (CLOSE_WAIT)

```
2个worker进程已经是CLOSE_WAIT的状态了。

### 事故解决

I. Updating Linux

For a Linux client, there are four operating system keepalive parameters to change:

* `tcp_keepalive_probes` - the number of probes that are sent and unacknowledged before the client considers the connection broken and notifies the application layer

* `tcp_keepalive_time` - the interval between the last data packet sent and the first keepalive probe

* `tcp_keepalive_intvl` - the interval between subsequent keepalive probes

* `tcp_retries2` - the maximum number of times a packet is retransmitted before giving up


II. On the Linux operating system, update these parameters using the "echo" command:

```
echo "60" > /proc/sys/net/ipv4/tcp_keepalive_time
echo "1" > /proc/sys/net/ipv4/tcp_keepalive_intvl
echo "10" > /proc/sys/net/ipv4/tcp_keepalive_probes
echo "3" > /proc/sys/net/ipv4/tcp_retries2
```

The `tcp_keepalive_time` and `tcp_keepalive_intvl` values are expressed in seconds. To retain these values after a system restart, they must be added to the `/etc/sysctl.conf` file.

```
net.ipv4.tcp_keepalive_time = 60
net.ipv4.tcp_keepalive_intvl = 1
net.ipv4.tcp_keepalive_probes = 10
net.ipv4.tcp_retries2 = 3

```

### 注
worker出现CLOSE_WAIT，问题一定在代码部分。tornado mysql缺少快速失败机制，始终是一个坑。重构代码，需要集中的时间。更改系统参数，是一个缓冲方案。
