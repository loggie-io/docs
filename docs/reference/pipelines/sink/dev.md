# dev

dev sink将日志数据打印到控制台，一般可以用于debug或者排查问题。  
配置dev sink后，可以设置printEvents=true，查看在Loggie中发送至sink的日志数据，该数据除了source接收或者采集的原始日志，一般还包含其他元信息。

!!! example

    ```yaml
    sink:
      type: dev
      printEvents: true
      codec:
        type: json
        pretty: true
    ```

## printEvents

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| printEvents | bool  |    非必填    |    false  | 是否打印采集的日志 |

默认情况下Loggie的日志打印为json格式，可以配置启动参数`-log.jsonFormat=false`，便于在Loggie日志上查看输出结果。

## printEventsInterval

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| printEventsInterval | time.Duration  |    非必填    |      | 间隔打印采集的日志，如果需要输出的日志量很大，可填入一个时间间隔，比如`10s`，Loggie只会在每10s打印一次日志，避免太多日志刷屏不便查看 |

## printMetrics

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| printMetrics | bool  |    非必填    |      | 是否打印发送日志统计的一些指标，包括totalCount和qps |

## printMetricsInterval

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| printMetricsInterval | time.Duration  |    非必填    |  1s    | printMetrics开启后，打印的时间间隔 |

## resultStatus

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| resultStatus | string  |    非必填    |  success    | 模拟sink发送端对日志的处理，包括`success`，`fail`，`drop` |

除了在配置中填写外，还可以使用接口动态的进行模拟，比如在Loggie运行时将sink `success`状态置为`fail`状态。

```bash
# 请将下面的<pipelineName>替换为实际配置的pipelineName
curl <ip>:<port>/api/v1/pipeline/<pipelineName>/sink/dev?status=fail
```