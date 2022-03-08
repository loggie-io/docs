# filewatcher listener

对文件采集情况的定时检查并暴露指标，包括文件名称、ackOffset、修改时间、大小等。

## 配置

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| period |  time.Duration |    非必填    |   5m   | 定时检查间隔时间 |
| checkUnFinishedTimeout |  time.Duration |    非必填    |   24h   | 检查文件是否采集完毕的超时时间，如果检测到文件的最近修改时间为`checkUnFinishedTimeout`之前，同时文件的并未采集完毕，则会在metrics中被标记为unfinished状态，可用于检查是否有长时间未被采集的日志文件 |


## Metrics

### 全局级别
#### total_file_count

```
# HELP file count total
# TYPE loggie_filewatcher_total_file_count gauge
loggie_filewatcher_total_file_count{} 20
```

* HELP: 表示当前被Loggie检测到活跃的文件个数总和
* TYPE: gauge

#### inactive_file_count

```
# HELP inactive file count
# TYPE loggie_filewatcher_inactive_file_count gauge
loggie_filewatcher_inactive_file_count{} 20
```

* HELP: 表示当前被Loggie检测到的不活跃的文件个数总和
* TYPE: gauge

### 文件级别

文件级别包括了以下prometheus labels:  

* LABELS:
    * pipeline: 表示所在的pipeline名称
    * source: 表示所在的source名称
    * filename: 表示文件名
    * status: 表示文件状态。
        * pending: 已被检测到，可能采集中或者采集完毕
        * unfinished: 文件的modify time距当前时间已经超过`checkUnFinishedTimeout`时间
        * ignored: 文件被忽略，可能超过`ignore_older`时间

由于定时扫描的时间间隔`period`默认为5min，以下指标可能存在一定程度的延迟。

#### file_size

```
# HELP file size
# TYPE loggie_filewatcher_file_size gauge
loggie_filewatcher_file_size{pipeline="xxx", source="access", filename="/var/log/a.log", status="pending"} 2048
```

* HELP: 表示该文件的总大小
* TYPE: gauge


#### file_ack_offset

```
# HELP file ack offset
# TYPE loggie_filewatcher_file_ack_offset gauge
loggie_filewatcher_file_ack_offset{pipeline="xxx", source="access", filename="/var/log/a.log", status="pending"} 1024
```

* HELP: 表示该文件被采集后，并且已经接收到ack的offset，可被理解为已经发送成功的文件offset进度
* TYPE: gauge

#### file_last_modify

```
# HELP file last modify timestamp
# TYPE loggie_filewatcher_file_last_modify gauge
loggie_filewatcher_file_last_modify{pipeline="xxx", source="access", filename="/var/log/a.log", status="pending"} 2343214422
```

* HELP: 文件的最新被修改时间
* TYPE: gauge

