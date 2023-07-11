# kafka

使用sink kafka将日志数据发送至下游Kafka。

!!! example

    === "简单"
        ```yaml
        sink:
          type: kafka
          brokers: ["127.0.0.1:6400"]
          topic: "log-${fields.topic}"
        ```

    === "SASL认证"
        ```yaml
        sink:
          type: kafka
          brokers: ["127.0.0.1:6400"]
          topic: "demo"
          sasl:
            type: scram
            userName: ***
            password: ***
            algorithm: sha512
        ```


## brokers

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| brokers | string数组  |    必填    |   无  | 发送日志至Kafka的brokers地址 |

## topic

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| topic | string  |    非必填    |   loggie  | 发送日志至Kafka的topic |

可使用`${a.b}`的方式，获取event里的字段值作为具体的topic名称。

比如，一个event为：

```json
{
  "topic": "loggie",
  "hello": "world"
}
```
可配置`topic: ${topic}`，此时该event发送到Kafka的topic为"loggie"。

同时支持嵌套的选择方式：

```json
{
  "fields": {
    "topic": "loggie"
  },
  "hello": "world"
}
```
可配置`topic: ${fields.topic}`，同样也会发送到topic "loggie"。

## ifRenderTopicFailed

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| ifRenderTopicFailed |   |    非必填  |     | 如果使用动态的规则渲染topic，比如`topic: ${fields.topic}`，有可能渲染失败（比如日志没有fields.topic字段），下面配置指示失败后的动作。 |
| ifRenderTopicFailed.dropEvent |   |    非必填  |  true   | 默认为丢弃 |
| ifRenderTopicFailed.ignoreError |   |    非必填  |     | 忽略错误日志，请注意，这里只是不打印错误日志 |
| ifRenderTopicFailed.defaultTopic |   |    非必填  |     |  发送至该默认的topic，配置后dropEvent不生效 |

!!! example
    
    === "1"

        ```yaml
        sink:
          type: kafka
          brokers: ["127.0.0.1:6400"]
          topic: "log-${fields.topic}"
          ifRenderTopicFailed:
            dropEvent: true
        ```

    === "2"

        ```yaml
        sink:
          type: kafka
          brokers: ["127.0.0.1:6400"]
          topic: "log-${fields.topic}"
          ifRenderTopicFailed:
            ignoreError: true
            defaultTopic: default
        ```

## ignoreUnknownTopicOrPartition

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| ignoreUnknownTopicOrPartition |   |    非必填  |     | 用于当发送的topic不存在时，忽略Kafka返回UNKNOWN_TOPIC_OR_PARTITION的报错 |

- 这种情况一般发生在使用动态渲染的topic，但是环境里的Kafka关闭了自动创建topic，导致无法发送至渲染出来的topic。默认情况Loggie会一直不停的重试，从而无法发送新的日志。
- 开启ignoreUnknownTopicOrPartition后，会直接丢弃发送的日志，避免影响其他正常包含已存在topic的日志发送。
- 请注意和上面`ifRenderTopicFailed`的区别，`ifRenderTopicFailed`是动态渲染不出来topic或者渲染的为空值，而`ignoreUnknownTopicOrPartition`则是渲染成功，但是topic在Kafka中实际不存在。


## balance

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| balance | string  |    非必填    |   roundRobin  | 负载均衡策略，可填`hash`、`roundRobin`、`leastBytes` |


## compression

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| compression | string  |    非必填    |   gzip  | 日志发送至Kafka的压缩策略，可填`gzip`、`snappy`、`lz4`、`zstd` |

## maxAttempts

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| maxAttempts | int  |    非必填    |   10  | 发送最多重试次数 |

## batchSize

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| batchSize | int  |    非必填    |   100  | 发送时每个batch最多包含的数据个数 |

## batchBytes

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| batchBytes | int  |    非必填    |   1048576  | 每个发送请求包含的最大字节数 |

## batchTimeout

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| batchTimeout | time.Duration  |    非必填    |   1s  | 形成每个发送batch的最长时间 |

## readTimeout

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| readTimeout | time.Duration  |    非必填    |   10s  | 读取超时时间 |

## writeTimeout

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| writeTimeout | time.Duration  |    非必填    |   10s  | 写入超时时间 |

## requiredAcks

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| requiredAcks | int  |    非必填    |   0  | 等待ack参数，可为`0`、`1`、`-1` |

- `0`: 不要求ack
- `1`: 等待leader partition ack
- `-1`: 等待ISR中所有replica ack

## sasl

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| sasl |   |    非必填    |     | SASL authentication |
| sasl.type | string  |    必填    |     | SASL类型，可为：`plain`、`scram` |
| sasl.userName | string  |    必填    |     | 用户名 |
| sasl.password | string  |    必填    |     | 密码 |
| sasl.algorithm | string  |    type=scram时必填    |     | type=scram时使用的算法，可选`sha256`、`sha512` |


## partitionKey

|    `字段`   | `类型`   |  `是否必填`  | `默认值` | `含义`            |
| ---------- |--------| ----------- |-------|-----------------|
| partitionKey | string |    非必填    | 无     | 控制发送至topic下哪个分区 |

与topic相似，可使用`${a.b}`的方式，获取event里的字段值作为具体的topic名称。
