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