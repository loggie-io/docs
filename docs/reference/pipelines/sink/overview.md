# Overview

一个Pipeline对应一个Sink。

Concurrency相关配置使用可参照[自适应sink流量控制](../../../user-guide/best-practice/concurrency.md)。

## Sink通用配置

!!! example

    ```yaml
    sink:
      type: "dev"
      codec:
        type: json
        pretty: true
      parallelism: 16
      concurrency:
        enabled: true
        rtt:
          blockJudgeThreshold: 120%
          newRttWeigh: 0.4
        goroutine:
          initThreshold: 8
          maxGoroutine: 20
          unstableTolerate: 3
          channelLenOfCap: 0.4
        ratio:
          multi: 2
          linear: 2
          linearWhenBlocked: 4
        duration:
          unstable: 15
          stable: 30
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
| codec.printEvents | bool  |    非必填    |  false  | 是否打印出events到Loggie的日志中，可用于临时排查问题 |
| codec.pretty | bool  |    非必填    |   false | 打印出events到Loggie的日志中的时候，是否美化展示，仅type:json时生效，请注意，如果是发送到Elasticsearch等下游，请勿使用pretty，会导致发送异常 |

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

#### printEvents

是否打印出events到Loggie的日志中，可用于临时排查问题。  
此时无需将当前的sink type修改为dev，更加方便。  

!!! example

    ```yaml
    sink:
      type: kafka
      ...
      codec:
        printEvents: true
    ```

### concurrency

!!! example

    ```yaml
    sink:
      type: kafka
      concurrency:
        enabled: true
    ```


|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| enabled | bool  |    非必填    |   false  | 是否开启sink自适应并发度控制 |

注：默认此功能不开启。

#### concurrency.goroutine

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| initThreshold | int  |    非必填    |   16  | 初始阈值，低于该值，协程数指数增长（快启动阶段），高于该值，线性增长 |
| maxGoroutine | int  |    非必填    |   30  | 最大协程数 |
| unstableTolerate | int  |    非必填    |   3  | 进入平稳阶段后，对网络波动的容忍，协程数需减少的情况包括rtt增大，请求失败，协程数需增大的情况包括网络平稳且channel饱和，相同情况出现3次才会触发协程数改变，若出现第四次，将会追加，直到相反的情况触发协程数改变。 |
| channelLenOfCap | float  |    非必填    |   0.4  | channel饱和阈值，超过该值认为协程数需增大，仅在rtt稳定情况下才会计算该值 |

#### concurrency.rtt

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| blockJudgeThreshold | string  |    非必填    |   120%  | 判断rtt增大阈值，当新rtt超过当前平均rtt的到达一定程度，认为协程数需减少 |
| newRttWeigh | float  |    非必填    |   0.5  | 计算新的平均rtt时，新rtt的权重 |

注：blockJudgeThreshold（b）支持百分比和浮点数两种。

若为百分比，则判断是否 (新rtt/平均rtt)>b 。

若为浮点数，则判断是否 (新rtt-平均rtt)>b 。

#### concurrency.ratio

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| multi | int  |    非必填    |   2  | 快启动阶段，协程数指数增长速率 |
| linear | int  |    非必填    |   2  | （快启动之后）协程数线性增长或减少速率 |
| linearWhenBlocked | int  |    非必填    |   4  | channel满时(上游阻塞)，协程数线性增长速率 |

#### concurrency.duration

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| unstable | int  |    非必填    |   15  | 非平稳阶段，收集数据计算协程数的时间间隔，单位秒 |
| stable | int  |    非必填    |   30  | 平稳阶段，收集数据计算协程数的时间间隔，单位秒 |
