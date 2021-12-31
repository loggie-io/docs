
# Monitor
监控事件总线，所有的组件都可以发出自己的metrics指标数据，由listeners消费处理。

!!! example

    ```yaml
    monitor:
      logger:
        period: 30s
        enabled: true
      listeners:
        filesource: ~
        filewatcher: ~
        reload: ~
        sink: ~    
    ```

## logger

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| logger | map  |    非必填    |      | 控制监控指标metrics输出到Loggie本身的日志中 |
| logger.enabled | bool  |    非必填    |  false    | 是否开启 |
| logger.period | time.Duration  |    非必填    |  10s    | 指标打印的时间间隔，数据量较大时建议将间隔延长，如30s、5m |


## listeners  

### filesource

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| filesource |   |    非必填    |      | file source实时文件采集的监控指标处理，包括文件的名称、采集进度、文件大小、qps等 |


### filewatcher
  
|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| filewatcher |   |    非必填    |      | 对文件采集情况的定时检查并暴露指标，包括文件名称、ackOffset、修改时间、大小等 |
| filewatcher.period |  time.Duration |    非必填    |   5m   | 定时检查间隔时间 |
| filewatcher.checkUnFinishedTimeout |  time.Duration |    非必填    |   24h   | 检查文件是否采集完毕的超时时间，如果检测到文件的最近修改时间为`checkUnFinishedTimeout`之前，同时文件的并未采集完毕，则会在metrics中被标记为unfinished状态 |


### sink

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| sink |   |    非必填    |      | sink发送端的监控指标处理，包括发送成功的event数量、发送失败的event数量、event qps等 |


### reload

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| reload |   |    非必填    |      | reload的指标处理，目前为总的reload次数 |


### logAlert

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| logAlert |   |    非必填    |      | 日志报警 |
| logAlert.alertManagerAddress | string数组  |    必填    |      | alertManager地址 |
| logAlert.bufferSize | int  |    非必填    |   100   | 日志报警发送的buffer大小，单位为报警事件个数 |
| logAlert.batchTimeout | time.Duration  |    非必填    |   10s   | 每个报警发送batch的最大发送时间 |
| logAlert.batchSize | int  |    非必填    |   10   | 每个报警发送batch的最大包含报警请求个数 |