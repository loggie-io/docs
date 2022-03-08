# queue listener

## 配置
|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| period | time.Duration  |    非必填    |   10s   |  listener消费处理数据的时间间隔 |

## Metrics

* LABELS:
    * pipeline: 表示所在的pipeline名称
    * type: queue队列的类型

#### capacity

```
# HELP queue capacity
# TYPE loggie_queue_capacity gauge
loggie_queue_capacity{pipeline="xxx", type="channel"} 2048
```

* HELP: 当前队列的容量
* TYPE: gauge

#### size

```
# HELP queue size
# TYPE loggie_queue_size gauge
loggie_queue_size{pipeline="xxx", type="channel"} 2048
```

* HELP: 当前队列被使用的长度
* TYPE: gauge

#### fill_percentage

```
# HELP how full is queue
# TYPE loggie_queue_fill_percentage gauge
loggie_queue_fill_percentage{pipeline="xxx", type="channel"} 50
```

* HELP: 当前队列使用的百分比
* TYPE: gauge