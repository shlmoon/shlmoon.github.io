---
layout: post
title: client online 监听
date: 2022-08-22 09:10:00
tags: websocket ping
---

### 用户状态监测

根据client的ping消息，监听用户状态


### 分时采集ping 消息

每隔5S，采集10S内的ping消息

```
time_now, ms_delta, listen_list = int(time.time()), 0, []
while ms_delta < 10:
    value = MQ.message
    if value:
        listen_list.append(value)
    ms_delta = int(time.time()) - time_now
func(listen_list)
time.sleep(5)

```
这里面有一个小技巧，采集10S内的ms，可以用时间差作为循环控制条件


### 差值思想，获取未ping的客户端（下线状态）

上述步骤获取当前10S内的ping消息，online_list

关键：通过redis获取上次前10S内的ping消息，last_online
* 如果last_online为空，说明当前offline_list不存在。将当前的online_list存入到redis，并作为下次的last_online
	```
	cache_key = ""
    if not value_list:
        redis_client.delete(cache_key)

    with redis_client.pipeline() as pipe:
        pipe.multi()
        pipe.delete(cache_key)
        pipe.sadd(cache_key, *value_list)
        pipe.expireat(cache_key, expired_at)
        pipe.execute()

	```
* 如果last_online不为空，则当前不在线状态为：`offline_list = last_online - online_list`

