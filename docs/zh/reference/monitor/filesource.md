# filesource listener

实时文件采集的监控，表示当前日志采集的进度与状态，包括文件的名称、采集进度、QPS等。

## 配置
|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| period | time.Duration  |    非必填    |   10s   |  listener消费处理数据的时间间隔 |

## Metrics

* LABELS:
    * pipeline: 表示所在的pipeline名称
    * source: 表示所在的source名称
    * filename: 表示文件名

#### file_size

```
# HELP file size
# TYPE loggie_filesource_file_size gauge
loggie_filesource_file_size{pipeline="xxx", source="access", filename="/var/log/a.log"} 2048
```

* HELP: 表示当前某个日志文件被采集时的文件大小
* TYPE: gauge


#### file_offset

```
# HELP file offset
# TYPE loggie_filesource_file_offset gauge
loggie_filesource_file_offset{pipeline="xxx", source="access", filename="/var/log/a.log"} 1024
```

* HELP: 表示当前某个日志文件被采集的进度，当前读取文件的offset
* TYPE: gauge


#### line_number

```
# HELP current read line number
# TYPE loggie_filesource_line_number gauge
loggie_filesource_line_number{pipeline="xxx", source="access", filename="/var/log/a.log"} 20
```

* HELP: 表示当前某个日志文件被采集时的当前读取的行数
* TYPE: gauge


#### line_qps

```
# HELP current read line qps
# TYPE loggie_filesource_line_qps gauge
loggie_filesource_line_qps{pipeline="xxx", source="access", filename="/var/log/a.log"} 48
```

* HELP: 表示当前某个日志文件被采集时每秒读取的行数
* TYPE: gauge
