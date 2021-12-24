# grpc

Grpc source用于接收Loggie Grpc格式的数据请求。  
一般用在[中转机](../../../user-guide/use-in-kubernetes/aggregator.md)场景，接收其他Loggie集群发送的日志。  

!!! example

    ```yaml
    sources:
    - type: grpc
      name: aggre
      port: 6066
    ```


## bind

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| bind | string  |    非必填    |   0.0.0.0   | 提供server绑定的host |


## port

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| port | string  |    非必填    |   6066   | 提供服务的端口号 |

## timeout

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| timeout | time.Duration  |    非必填    |   20s   | 超时时间 |

## maintenanceInterval





