# rateLimit

用于日志限流。  
属于source interceptor。

!!! example

    ```yaml
    interceptors:
    - type: rateLimit
      qps: 4000
    ```

## qps

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| qps | int  |    非必填    |  2048    | 限流qps |
