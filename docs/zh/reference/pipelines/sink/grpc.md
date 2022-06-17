# grpc

使用grpc sink发送至下游，需要下游支持相同格式的grpc协议。可以使用该sink发送至grpc source的Loggie中转机。

!!! example

    ```yaml
    sink:
      type: grpc
      host: "loggie-aggregator.loggie-aggregator:6166"
    ```

## host
|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| host | string  |    必填    |   无  | 发送至下游的host地址，多个ip使用`,`逗号分隔 |


## loadBalance
|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| loadBalance | string |    非必填    |   round_robin  | grpc负载均衡策略 |


## timeout
|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| timeout | time.Duration  |   非必填    |   30s  | 发送超时时间 |


## grpcHeaderKey
|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| grpcHeaderKey | string  |    非必填    |   无  |  |
