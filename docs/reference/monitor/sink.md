# sink listener

sink发送端的监控指标处理，包括发送成功的event数量、发送失败的event数量、event qps等

## 配置
|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| period | time.Duration  |    非必填    |   10s   |  listener消费处理数据的时间间隔 |


## Metrics

* LABELS:
    * pipeline: 表示所在的pipeline名称
    * source: 表示所在的source名称

#### success_event

```
# HELP send event success count
# TYPE loggie_sink_success_event gauge
loggie_sink_success_event{pipeline="xxx", source="access"} 2048
```

* HELP: 在`period`时间段内，发送成功的event个数
* TYPE: gauge

#### failed_event

```
# HELP send event failed count
# TYPE loggie_sink_failed_event gauge
loggie_sink_failed_event{pipeline="xxx", source="access"} 2048
```

* HELP: 在`period`时间段内，发送失败的event个数
* TYPE: gauge

#### event_qps

```
# HELP send success event failed count
# TYPE loggie_sink_event_qps gauge
loggie_sink_event_qps{pipeline="xxx", source="access"} 2048
```

* HELP: 在`period`时间段内，发送的event QPS
* TYPE: gauge