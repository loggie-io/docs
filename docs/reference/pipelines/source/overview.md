# Overview

sources字段为数组，一个Pipeline中可填写多个source组件配置。

因此，请注意所有的source中`name`必填，作为pipeline中source的唯一标识。

## Source通用配置

所有Source均可以使用以下配置。

### enabled

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| enabled | bool  |    非必填    |   true  | 表示是否开启该source |


### name

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| name | string  |    必填    |     | 表示source的名称，建议填写有标识意义的词 |

### fields

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| fields | map  |    非必填    |     | 自定义额外添加到event中的字段 |

比如如下配置:

!!! example

    ```
    sources:
    - type: file
      name: access
      paths:
      - /var/log/*.log
      fields:
        service: demo
    ```

会给采集的所有日志上，都加上`service: demo`字段。

### fieldsFromEnv

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| fieldsFromEnv | map  |    非必填    |     | 额外添加到event中的字段，value为env环境变量的key |

比如如下配置:

!!! example

    ```
    sources:
    - type: file
      name: access
      paths:
      - /var/log/*.log
      fieldsFromEnv:
        service: SVC_NAME
    ```

会从Loggie所在的环境变量中，获取`SVC_NAME`的值`${SVC_NAME}`，然后给所有的日志event上添加字段：`service: ${SVC_NAME}`。

### fieldsUnderRoot

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| fieldsUnderRoot | bool  |    非必填    |   false  | 额外添加的fields是否放在event的根部 |

比如，默认情况下，输出的日志格式为：

```json
{
    "body": "hello world",
    "fields": {
        "service": "demo"
    }
}
```

如果设置fieldsUnderRoot=true，输出的日志格式为：

```json
{
    "body": "hello world",
    "service": "demo"
}
```


### fieldsUnderKey

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| fieldsUnderKey | string  |    非必填    |  fields   | 当`fieldsUnderRoot=false`时，字段的名称 |

比如可以修改默认的字段`fields`为`tag`，输出的日志为：

```json
{
    "body": "hello world",
    "tag": {
        "service": "demo"
    }
}
```
