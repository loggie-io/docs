# Dev sink

dev sink一般可以用于debug或者排查问题，配置dev sink后，可以设置printEvents=true，查看在Loggie中发送至sink的日志数据，该数据除了source接收或者采集的原始日志，一般还包含其他元信息

## 配置

### printEvents

|  `类型`  |  `是否必填`  |  `默认值`  |  `含义`  |
| ------- | ----------- | --------- | ------- |
| bool  |    非必填    |    false  | 是否打印采集的日志 |

