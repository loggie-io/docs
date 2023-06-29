# rocketmq

rocketmq sink 可以将日志数据发送至下游 [RocketMQ](https://github.com/apache/rocketmq) 。

!!! example

    ```yaml
    sink:
      type: rocketmq
      nameServer: 
        - 127.0.0.1:9876
      topic: "log-${fields.topic}"
    ```

## nameServer

|    `字段`   | `类型`      | `是否必填` |  `默认值`  | `含义`                        |
| ---------- |-----------|--------| --------- |-----------------------------|
| nameServer | string 数组 | 非必填    |   无  | RocketMQ nameserver addr 列表 |

## nsResolver

|    `字段`   | `类型`      |  `是否必填`  | `默认值` | `含义`                                |
| ---------- |-----------| ----------- |-------|-------------------------------------|
| nsResolver | string 数组 |    非必填    | 无     | 用于获取 nameserver addr 的  reslover 列表 |

> 注意：nameServer 和 nsResolver 二选一，必须要设置一个

## topic

|    `字段`   |    `类型`    |  `是否必填`  | `默认值`  | `含义`                   |
| ---------- | ----------- | ---------- |--------|------------------------|
| topic | string  |    必填    | loggie | 发送日志至 RocketMQ 的 topic |

可使用 `${a.b}` 的方式，获取 event 里的字段值作为具体的 topic 名称。

比如，一个 event 为：

```json
{
  "topic": "loggie",
  "hello": "world"
}
```
可配置 `topic: ${topic}`，此时该 event 发送到 RocketMQ 的 topic 为 "loggie"。

同时支持嵌套的选择方式：

```json
{
  "fields": {
    "topic": "loggie"
  },
  "hello": "world"
}
```

可配置 `topic: ${fields.topic}`，同样也会发送到 topic "loggie"。

## ifRenderTopicFailed

| `字段`                 | `类型`   |  `是否必填`  | `默认值` | `含义`                  |
|----------------------|--------| ----------- |-------|-----------------------|
| ifRenderTopicFailed  | object |    非必填    |       | 配置 topic 动态渲染失败的动作参数  |
| ifRenderTopicFailed.dropEvent | bool   |    非必填    | true  | 是否丢弃此消息               |
| ifRenderTopicFailed.ignoreError | bool   |    非必填    | false | 是否忽略错误                |
| ifRenderTopicFailed.defaultValue | string |    非必填    | 无     | 渲染失败时使用的默认配置的 topic 值 |

## group

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  | `含义`                                        |
| ---------- | ----------- | ----------- | --------- |---------------------------------------------|
| group | string  |    非必填    |   DEFAULT_PRODUCER  | 同一类 Producer 的集合，这类 Producer 发送同一类消息且发送逻辑一致 |

## tag

| `字段` |    `类型`    |  `是否必填`  | `默认值` | `含义`                                                        |
|------| ----------- | ----------- |-------|-------------------------------------------------------------|
| tag  | string  |    非必填    | 无     | 为消息设置的标志，用于同一主题下区分不同类型的消息，可理解为二级消息类型，用来进一步区分某个 Topic 下的消息分类 |

RocketMQ 有对 tag 能力的支持，因此，类似 topic，tag 也可使用 `${a.b}` 的方式，获取 event 里的字段值作为具体的 tag 名称。

比如，一个 event 为：

```json
{
  "tag": "loggie",
  "hello": "world"
}
```
可配置 `tag: ${tag}`，此时该 RocketMQ 的消息的 tag 为 "loggie"。

同时支持嵌套的选择方式：

```json
{
  "fields": {
    "tag": "loggie"
  },
  "hello": "world"
}
```

可配置 `tag: ${fields.tag}`，同样也会配置消息的 tag 为 "loggie"。

## ifRenderTagFailed

| `字段`                 | `类型`   |  `是否必填`  | `默认值` | `含义`                |
|----------------------|--------| ----------- |-------|---------------------|
| ifRenderTagFailed  | object |    非必填    |       | 配置 tag 动态渲染失败的动作参数  |
| ifRenderTagFailed.dropEvent | bool   |    非必填    | true  | 是否丢弃此消息             |
| ifRenderTagFailed.ignoreError | bool   |    非必填    | false | 是否忽略错误              |
| ifRenderTagFailed.defaultValue | string |    非必填    | 无     | 渲染失败时使用的默认配置的 tag 值 |

> 注意：因为 tag 是非必须参数，默认为空，因此，只有配置了 tag 并且渲染失败时，ifRenderTagFailed 参数才会生效

## messageKeys

|    `字段`   | `类型`      |  `是否必填`  | `默认值` | `含义`                                                     |
| ---------- |-----------| ----------- |-------|----------------------------------------------------------|
| messageKeys | string 数组 |    非必填    | 无     | 消息的业务标识，由消息生产者（Producer）设置，`唯一` 标识某个业务逻辑，比如以订单 ID 为  key |

## retry

|    `字段`   |    `类型`    |  `是否必填`  | `默认值` | `含义` |
| ---------- | ----------- | ----------- |-------|------|
| retry | int  |    非必填    | 2     | 重试次数 |

## sendMsgTimeout

|    `字段`   |    `类型`    |  `是否必填`  | `默认值` | `含义`      |
| ---------- | ----------- | ----------- |-------|-----------|
| sendMsgTimeout | time.Duration  |    非必填    | 3s    | 消息发送的超时时间 |

## compressLevel

|    `字段`   | `类型` |  `是否必填`  | `默认值` | `含义`          |
| ---------- |------| ----------- |-------|---------------|
| compressLevel | int  |    非必填    | 5     | 消息的压缩登记，取值范围：0 1 2 3 4 5 6 7 8 9 |

## compressMsgBodyOverHowmuch

|    `字段`   | `类型` |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- |------| ----------- | --------- | -------- |
| compressMsgBodyOverHowmuch | int  |    非必填    |   4096  | 消息Body超过多大开始压缩（Consumer收到消息会自动解压缩），单位字节 |

## topicQueueNums

|    `字段`   | `类型` |  `是否必填`  | `默认值` |  `含义`  |
| ---------- |------| ----------- |-------| -------- |
| topicQueueNums | int  |    非必填    | 4     | 在发送消息，自动创建服务器不存在的topic时，默认创建的队列数 |

## credentials

> credentials 用于 RocketMQ 开启了 ACL 权限控制的场景

|    `字段`   | `类型`   | `是否必填` | `默认值` | `含义`                               |
| ---------- |-------|--------|-------|------------------------------------|
| credentials |  object | 非必填    | 无     | 客户端用于身份验证的凭证信息，仅在服务端开启身份识别和认证时需要传输 |
| credentials.accessKey  | string | 必填     | 无     | accessKey |
| credentials.accessKey  | string | 必填     | 无     | secretKey |
| credentials.securityToken  | string | 非必填    | 无     | secretKey |
