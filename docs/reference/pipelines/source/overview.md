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

### codec

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| codec |   |    非必填    |    | source接收到数据的时候用于解析预处理 |
| codec.type | string  |    非必填    |   无 |  |

请注意：目前仅`file source`支持source codec。

#### type: json

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| codec.bodyFields |   |    必填    |    | 使用解析读取到的json数据中的该字段作为`body` |

配置示例：
!!! example "type: json"

    ```yaml
      sources:
      - type: file
        name: nginx
        paths:
        - /var/log/*.log
        codec:
          type: json
          bodyFields: log
    ```

如果采集到的日志为：
```json
{"log":"I0610 08:29:07.698664 Waiting for caches to sync\n", "stream":"stderr", "time:"2021-06-10T08:29:07.698731204Z"}
```
则codec后得到的event为：
```
body: "I0610 08:29:07.698664 Waiting for caches to sync"
```

请注意：目前非bodyFields的字段均会被丢弃。


#### type: regex

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| codec.pattern |   |    必填    |    | 正则表达式 |
| codec.bodyFields |   |    必填    |    | 使用正则提取到的该字段作为`body` |

配置示例：
!!! example "type: regex"

    ```yaml
      sources:
      - type: file
        name: nginx
        paths:
        - /var/log/*.log
        codec:
          type: regex
          pattern: ^(?P<time>[^ ^Z]+Z) (?P<stream>stdout|stderr) (?P<logtag>[^ ]*) (?P<log>.*)$
          bodyFields: log
    ```

如果采集到的日志为：
```
2021-12-01T03:13:58.298476921Z stderr F INFO [main] Starting service [Catalina]
```
则codec后得到的event为：
```
body: "INFO [main] Starting service [Catalina]"
```

请注意：目前非bodyFields的字段均会被丢弃。