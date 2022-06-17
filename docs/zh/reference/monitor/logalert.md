# logAlert listener

用于日志报警的发送。

## 配置

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| alertManagerAddress | string数组  |    必填    |      | alertManager地址 |
| bufferSize | int  |    非必填    |   100   | 日志报警发送的buffer大小，单位为报警事件个数 |
| batchTimeout | time.Duration  |    非必填    |   10s   | 每个报警发送batch的最大发送时间 |
| batchSize | int  |    非必填    |   10   | 每个报警发送batch的最大包含报警请求个数 |

