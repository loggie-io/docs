# maxbytes

对原始单行日志的大小进行限制，避免单行日志数据量太大影响Loggie内存和稳定性。   
系统内置，默认加载，属于source interceptor。  

!!! example

    ```yaml
    interceptors:
    - type: maxbytes
      maxBytes: 102400
    ```

## maxBytes

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| maxBytes | int  |    非必填    |    | 单行最大字节个数，超出的部分将会被丢弃 |
