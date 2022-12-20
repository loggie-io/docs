# loki

loki sink用于发送数据至Loki存储。Loki文档可参考[这里](https://grafana.com/docs/loki/latest/)。


!!! example

    ```yaml
    sink:
      type: loki
      url: "http://localhost:3100/loki/api/v1/push"
    ```

## url

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| url | string  |    必填    |      | push loki的api |

## tenantId

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| tenantId | string  |    非必填    |      | 发送使用的租户名称 |

## timeout

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| timeout | time.Duration  |    非必填    |  30s    | 发送的超时时间 |

## entryLine

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| entryLine | string  |    非必填    |      | 发送至Loki的日志内容，默认为loggie event的body |

Loki的日志数据结构大概分为label和主体数据，loggie会默认将header里的元信息字段，转成以下划线`_`连接的label。
另外需要注意的是，由于loki的labels key不支持`.`, `/`, `-`，这里会自动将header里包含这些符号的key转成`_`的形式。

## insecureSkipVerify

|    `字段`   | `类型` |  `是否必填`  | `默认值` | `含义`          |
| ---------- |------| ----------- |-------|---------------|
| insecureSkipVerify | bool |    非必填    | false | https是否跳过证书验证 |