# sys listener

Loggie本身的CPU和Memory指标

## 配置
|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| period | time.Duration  |    非必填    |   10s   |  listener消费处理数据的时间间隔 |


## Metrics

#### cpu_percent

```
# HELP loggie_sys_cpu_percent Loggie cpu percent
# TYPE loggie_sys_cpu_percent gauge
loggie_sys_cpu_percent 0.37
```

* HELP: Loggie的cpu使用率
* TYPE: gauge


#### mem_rss

```
# HELP loggie_sys_mem_rss Loggie memory rss bytes
# TYPE loggie_sys_mem_rss gauge
loggie_sys_mem_rss 2.5853952e+07
```

* HELP: Loggie的内存占用字节数
* TYPE: gauge

