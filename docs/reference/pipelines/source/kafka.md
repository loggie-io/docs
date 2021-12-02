# Kafka source

Kafka source用于接收Kafka数据。

## 配置
### brokers

|  `类型`  |  `是否必填`  |  `默认值`  |  `含义`  |
| ------- | ----------- | --------- | ------- |
| string  |    必填      |    无  | Kafka broker地址 |


### topics

|  `类型`  |  `是否必填`  |  `默认值`  |  `含义`  |
| ------- | ----------- | --------- | ------- |
| string数组  |    必填      |    无  | 接收的topics |

### groupId

|  `类型`  |  `是否必填`  |  `默认值`  |  `含义`  |
| ------- | ----------- | --------- | ------- |
| string  |    非必填      |    loggie  | Loggie消费 kafka client的groupId |


### enableAutoCommit

|  `类型`  |  `是否必填`  |  `默认值`  |  `含义`  |
| ------- | ----------- | --------- | ------- |
| bool    |    非必填    |   true  | 是否开启autoCommit |



### autoCommitInterval

|  `类型`  |  `是否必填`  |  `默认值`  |  `含义`  |
| ------- | ----------- | --------- | ------- |
| int    |    非必填    |     | autoCommit的间隔时间 |


### autoOffsetReset
|  `类型`  |  `是否必填`  |  `默认值`  |  `含义`  |
| ------- | ----------- | --------- | ------- |
| string    |    非必填    | latest  | 没有初始offset时，smallest earliest beginning largest latest end error |


## 示例
