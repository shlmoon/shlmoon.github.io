---
layout: post
title: kafka-python debug log
date: 2021-12-03 09:10:00
tags: kafka
---

#### python禁掉kafka debug日志

django 中使用kafka，但kafka 的debug日志会无限输出到日志文件中，非常头痛。

```
INFO 2022-07-29 08:46:51,544 0cd0ec8c3b57480093f54a8907cb2a63 [base._handle_group_coordinator_response line:694] Discovered coordinator coordinator-0 for group g1
WARNING 2022-07-29 08:46:51,544 0cd0ec8c3b57480093f54a8907cb2a63 [base.coordinator_dead line:716] Marking the coordinator dead (node coordinator-0) for group g1: Node Disconnected.
WARNING 2022-07-29 08:46:51,544 2d9682ce3ecd4d0481e70f9acd8ebdf2 [base.coordinator_dead line:716] Marking the coordinator dead (node coordinator-0) for group g1: Node Disconnected.
```

根本原因在于，kafka的python包中 `logger = logging.getLogger("kafka")`
如果在django 的setting中没有定义 kafka 的logger，那会直接输出到 ROOT中。

解决方法也很简单，在setting的logging配置中配置下 kafka logger

```
    "loggers": {
        "kafka": {
            "handlers": ["console"],
            "propagate": False,
            "level": "WARNING",
        },
        ...
    },
```

或者
```
logging.getLogger("kafka").setLevel(logging.WARNING)
```

日志level，看需求，一般是INFO，不像看到相关的kafka任何信息直接使用ERROR
