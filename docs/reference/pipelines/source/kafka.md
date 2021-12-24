# Kafka source

Kafka source用于接收Kafka数据。

!!! example
    ```yaml
    sources:
    - type: kafka
      brokers: ["kafka1.kafka.svc:9092"]
      topic: log-*
    ```

## brokers

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| brokers | string数组  |    必填      |    无  | Kafka broker地址 |


## topic

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| topic | string  |    必填      |    无  | 接收的topics，可使用正则来匹配多个topic |


## groupId

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| groupId | string  |    非必填      |    loggie  | Loggie消费kafka的groupId |


## autoCommitInterval

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| autoCommitInterval | int    |    非必填    |  0   | autoCommit的间隔时间 |


## autoOffsetReset

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| autoOffsetReset | string    |    非必填    | latest  | 没有offset时，初始的offset采用方式，可为`earliest`和`latest` |