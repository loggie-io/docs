# Overview

一个Pipeline对应一个Sink。

## Sink通用配置

!!! example

    ```yaml
    sink:
      type: "dev"
      codec:
        type: json
        pretty: true

    ```

### parallelism

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| parallelism | int  |    非必填    |   1  | sink客户端的并发度，可同时启动多个增大发送吞吐量，可设置的最大值为100 |

### codec

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| codec |   |    非必填    |    | sink发送数据给下游时，数据使用的格式 |
| codec.type | string  |    非必填    |   json | codec类型 |

#### type: json

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| codec.pretty |   |    非必填    |  false  | 是否进行json格式美化 |
| codec.beatsFormat |   |    非必填    |  false  | 日志转成类filebeats格式：增加`@timestamp`字段，同时body字段命名为`message` |

#### type: raw

用于发送采集的原始body数据。

!!! example

    ```yaml
    sink:
      type: dev
      codec:
        type: raw
    ```
