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
Loggie支持将metrics指标输出到日志中，可以通过logger配置。


|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| logger.enabled | bool  |    非必填    |  false    | 是否开启 |
| logger.period | time.Duration  |    非必填    |  10s    | 指标打印的时间间隔，数据量较大时建议将间隔延长，如30s、5m |
| logger.pretty | bool  |    非必填    |  false    | 打印的指标json是否需要友好展示 |
| logger.additionLogEnabled | bool  |    非必填    |  false    | 是否需要将打印的指标单独输出到另外的日志文件中，在数据量比较多的情况下，如果我们配置的打印时间间隔较短，可以打开该开关，避免太多的metrics日志干扰 |
| logger.additionLogConfig |   |    非必填    |      | 额外输出的日志配置参数 |
| logger.additionLogConfig.directory | bool  |    非必填    |  /data/loggie/log    | 额外输出的日志目录 |
| logger.additionLogConfig.maxBackups | int  |    非必填    |  metrics.log    | 日志轮转最多保留的文件个数，默认为3 |
| logger.additionLogConfig.maxSize | int  |    非必填    |  1024    | 日志轮转的时候，最大的文件大小，单位为MB |
| logger.additionLogConfig.maxAge | int  |    非必填    |  14    | 日志轮转最大保留的天数 |
| logger.additionLogConfig.timeFormat | string  |    非必填    |  2006-01-02 15:04:05    | 每行日志输出的时间格式 |

## listeners  
表示具体启动的listeners。  
配置不填写即为关闭，不启动该Listener，相关的的指标也不会被处理和暴露。