# logAlert

用于日志报警检测。  
属于source interceptor。
使用示例请参考[日志报警](../../../user-guide/monitor/service-log-alarm.md)。

!!! example

    ```yaml
    interceptors:
    - type: logAlert
      matcher:
        contains: ["error", "err"]

    ```

## matcher

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| matcher.contains | string数组  |    非必填    |    | 日志数据包含字符串检测 |
| matcher.regexp | string数组  |    非必填    |    | 日志数据正则检测 |
| matcher.target | string  |    非必填    |  body  | 根据日志数据的该字段进行检测，如果进行日志切分或者drop body字段，请填写所需字段 |


