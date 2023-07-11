# kafka

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

## queueCapacity

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| queueCapacity | int  |    非必填      |    100  | 内部发送的队列容量 |

## minAcceptedBytes

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| minAcceptedBytes | int  |    非必填      |    1  | 最小接收的batch字节数 |

## maxAcceptedBytes

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| maxAcceptedBytes | int  |    非必填      |    1e6（1MB)  | 最大接收的消息字节数，如果超过会被truncate，可以设置为一个能容忍的较大的值 |

## readMaxAttempts

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| readMaxAttempts | int  |    非必填      |    3  | 最大的重试次数 |

## maxPollWait

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| maxPollWait | time.Duration  |    非必填      |    10s  | 接收的最长等待时间 |

## readBackoffMin

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| readBackoffMin | time.Duration  |    非必填      |    100ms  | 在接收新的消息前，最小的时间间隔 |

## readBackoffMax

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| readBackoffMax | time.Duration  |    非必填      |    1s  | 在接收新的消息前，最大的时间间隔 |

## enableAutoCommit

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| enableAutoCommit | bool  |    非必填      |    false  | 是否开启autoCommit |

## autoCommitInterval

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| autoCommitInterval | time.Duration    |    非必填    |  1s   | autoCommit的间隔时间 |


## autoOffsetReset

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| autoOffsetReset | string    |    非必填    | latest  | 没有offset时，初始的offset采用方式，可为`earliest`和`latest` |

## sasl

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| sasl |   |    非必填    |     | SASL authentication |
| sasl.type | string  |    必填    |     | SASL类型，可为：`plain`、`scram` |
| sasl.userName | string  |    必填    |     | 用户名，请注意在v1.4和之前使用的名称为`userName`，当然后续版本也对此进行了兼容 |
| sasl.password | string  |    必填    |     | 密码 |
| sasl.algorithm | string  |    type=scram时必填    |     | type=scram时使用的算法，可选`sha256`、`sha512` |
