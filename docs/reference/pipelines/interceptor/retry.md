# retry

用于给下游发送失败时重试。  
系统内置，默认加载，属于sink interceptor。

!!! example

    ```yaml
    interceptors:
    - type: retry

    ```

## retryMaxCount

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| retryMaxCount | int  |    非必填    |  0    | 最大的重试次数 |