# franz kafka

使用[franz-go kafka](https://github.com/twmb/franz-go)库将日志数据发送至下游Kafka，并且能比较好的支持kerberos认证。  
（本sink和kafka sink的区别一般只在于使用的kafka golang库不同，提供给对franz kafka库有偏好的用户使用）

!!! example

    === "简单"
        ```yaml
        sink:
          type: franzKafka
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
            enabled: true
            mechanism: SCRAM-SHA-512
            username: ***
            password: ***
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

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  | `含义`                                   |
| ---------- | ----------- | ----------- | --------- |----------------------------------------|
| balance | string  |    非必填    |   roundRobin  | 负载均衡策略，可填`roundRobin`、`range`、`sticky`、`cooperativeSticky`  |


## compression

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| compression | string  |    非必填    |   gzip  | 日志发送至Kafka的压缩策略，可填`gzip`、`snappy`、`lz4`、`zstd` |

## batchSize

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| batchSize | int  |    非必填    |   100  | 发送时每个batch最多包含的数据个数 |

## batchBytes

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| batchBytes | int  |    非必填    |   1048576  | 每个发送请求包含的最大字节数 |

## writeTimeout

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| writeTimeout | time.Duration  |    非必填    |   10s  | 写入超时时间 |


## tls

| `字段`                   | `类型`   |  `是否必填`  |  `默认值`  | `含义`                  |
|------------------------|--------| ----------- | --------- |-----------------------|
| tls.enabled            |  bool      |    非必填    |   false  | 是否启用                  |
| tls.caCertFiles        |    string    |    非必填    |     | 证书文件路径                  |
| tls.clientCertFile     | string |    必填    |     | SASL类型，可为：`客户端cert文件` |
| tls.clientKeyFile     | string |    必填    |     | SASL类型，可为：`客户端key文件`  |
| tls.endpIdentAlgo          | bool   |    type=scram时必填    |     | 客户端是否验证服务端的证书名字       |

## sasl

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  | `含义`                                                                               |
| ---------- | ----------- | ----------- | --------- |------------------------------------------------------------------------------------|
| sasl |   |    非必填    |     | SASL authentication                                                                |
| sasl.enabled            |  bool |    非必填    |   false  | 是否启用                                 |
| sasl.mechanism | string  |    必填    |     | SASL类型，可为：`PLAIN`、`SCRAM-SHA-256`、`SCRAM-SHA-512`、`GSSAPI`|
| sasl.username | string  |    必填    |     | 用户名                                                                                |
| sasl.password | string  |    必填    |     | 密码                                                                                 |


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

## security

|    `字段`   | `类型`   |  `是否必填`  |  `默认值` | `含义`                                |
| ---------- |--------| ----------- | ------ |-------------------------------------|
| security | string |    非必填    |     | java格式的安全认证内容，可以自动转化成为franz-go适配的格式 |

!!! example
    ```
    pipelines:
      - name: local
        sources:
          - type: file
            name: demo
            paths:
              - /tmp/log/*.log
        sink:
          type: franzKafka
          brokers:
          - "hadoop74.axrzpt.com:9092"
          topic: loggie
          writeTimeout: 5s
          sasl:
            gssapi:
              kerberosConfigPath: /etc/krb5-conf/krb5.conf
          security:
            security.protocol: "SASL_PLAINTEXT"
            sasl.mechanism: "GSSAPI"
            sasl.jaas.config: "com.sun.security.auth.module.Krb5LoginModule required useKeyTab=true storeKey=true debug=true keyTab=\"/shylock/kerberos/zork.keytab\"  principal=\"zork@AXRZPT.COM\";"
            sasl.kerberos.service.name: "kafka"
    ```


Kubernetes 挂载keytab二进制证书，请参考[官方文档](https://kubernetes.io/zh-cn/docs/concepts/configuration/secret/)。
