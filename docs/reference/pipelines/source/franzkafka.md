# franz kafka

使用[franz-go kafka](https://github.com/twmb/franz-go)库消费Kafka。

（本sink和kafka sink的区别一般只在于使用的kafka golang库不同，提供给对franz kafka库有偏好的用户使用）

!!! example

简单
``` yaml
    sources:
    - type: franzKafka
      brokers: ["127.0.0.1:9092"]
      topic: log-*
``` 
SASL+TLS认证
```yaml
    sources:
    - type: franzKafka
      brokers: ["127.0.0.1:9092"]
      topic: log-*
      tls:
        enabled: true
        caCertFiles: /data/ca.crt
        endpIdentAlgo: false
      sasl:
        enabled: true
        mechanism: SCRAM-SHA-256
        username: admin
        password: admin

```




## brokers

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| brokers | string数组  |    必填      |    无  | Kafka broker地址 |


## topic

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| topic | string  |    必填      |    无  | 接收的topic，可使用正则来匹配多个topic |

## topics

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| topics | 数组  |    必填      |    无  | 接收的topics，可填多个topic，或者使用正则来匹配多个topic |


## groupId

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| groupId | string  |    非必填      |    loggie  | Loggie消费kafka的groupId |

## clientId

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| clientId | string  |    非必填      |    loggie  | Loggie消费kafka的clientId |

## worker

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| worker | int  |    非必填      |    1  | Loggie消费Kafka的worker线程个数 |

## addonMeta

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| addonMeta | bool  |    非必填      |    true  | 是否增加消费Kafka的一些元信息 |

增加的元信息字段包括：

- offset: 消费的偏移量
- partition: 分区
- timestamp: 消费时打上的时间戳，格式为RFC3339（"2006-01-02T15:04:05Z07:00"）
- topic: 消费的topic

## fetchMaxWait

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| fetchMaxWait | time.Duration  |    非必填      |   5s   | 在返回之前等待获取响应达到所需的最小字节数的最大时间 |

## fetchMaxBytes

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| fetchMaxBytes | int  |    非必填      |   50 << 20 （50MiB）   | 获取的最大字节数量 |

## fetchMinBytes

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| fetchMinBytes | int  |    非必填      |   1   | 获取的最小字节数量 |

## fetchMaxPartitionBytes

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| fetchMaxPartitionBytes | int  |    非必填      |  50 << 20 （50MiB）    | 从单个分区获取的最大字节数量 |

## enableAutoCommit

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| enableAutoCommit | bool  |    非必填      |   false   | 是否开启自动commit到kafka |

## autoCommitInterval

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| autoCommitInterval | time.Duration  |    非必填      |   1s   | 自动commit到kafka的时间间隔 |

## autoOffsetReset

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| autoOffsetReset | string  |    非必填      |   latest   | 如果没有对应的offset，则一开始从什么地方消费topic数据，可为：`latest`或者`earliest` |

## sasl

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  | `含义`                                                                               |
| ---------- | ----------- | ----------- | --------- |------------------------------------------------------------------------------------|
| sasl |   |    非必填    |     | SASL authentication                                                                |
| sasl.enabled            |  bool |    非必填    |   false  | 是否启用                                 |
| sasl.mechanism | string  |    必填    |     | SASL类型，可为：`PLAIN`、`SCRAM-SHA-256`、`SCRAM-SHA-512`、`GSSAPI`|
| sasl.username | string  |    必填    |     | 用户名                                                                                |
| sasl.password | string  |    必填    |     | 密码                                                                                 |

## tls

| `字段`                   | `类型`   |  `是否必填`  |  `默认值`  | `含义`                  |
|------------------------|--------| ----------- | --------- |-----------------------|
| tls.enabled            |  bool      |    非必填    |   false  | 是否启用                  |
| tls.caCertFiles        |    string    |    非必填    |     | 证书文件路径                  |
| tls.clientCertFile     | string |    必填    |     | SASL类型，可为：`客户端cert文件` |
| tls.clientKeyFile     | string |    必填    |     | SASL类型，可为：`客户端key文件`  |
| tls.endpIdentAlgo          | bool   |    type=scram时必填    |     | 客户端是否验证服务端的证书名字       |


## gssapi

| `字段`                           | `类型`   |  `是否必填`  |  `默认值`  | `含义`                                 |
|--------------------------------|--------| ----------- | --------- |--------------------------------------|
| sasl.gssapi                    |        |    非必填    |     | SASL authentication                  |
| sasl.gssapi.authType           | string |    必填    |     | SASL类型，可为：1 使用账号密码、2 使用keytab        |
| sasl.gssapi.keyTabPath         | string |    必填    |     | keytab 文件路径                          |
| sasl.gssapi.kerberosConfigPath | string |    必填    |     | kerbeos 文件路径                         |
| sasl.gssapi.serviceName        | string |    必填    |     | 服务名称                                 |
| sasl.gssapi.userName           | string |    必填    |     | 用户名                                  |
| sasl.gssapi.password           | string |    必填    |     | 密码                                   |
| sasl.gssapi.realm              | string |    必填    |     | 领域                                   |
| sasl.gssapi.disablePAFXFAST                   | bool   |    type=scram时必填    |     |DisablePAFXFAST 用于将客户端配置为不使用 PA_FX_FAST |
