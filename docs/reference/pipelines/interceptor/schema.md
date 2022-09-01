# schema
专注于日志格式转换与适配的interceptor。  
属于source interceptor。

## 使用场景
对于大部分日志对接的场景中，我们要求的日志格式可能有一些差异，这些差异主要体现在时间字段、body字段等。  
默认情况下，Loggie只会把source采集或者接收到的原始数据放到body字段中，以最简单的方式发送：
```json
{
    "body": "this is raw data"
}
```

但是，一些场景下我们需要：

- 增加时间字段
- 修改`body`等字段

!!! example
    === "增加@timestamp，body修改为message"

        ```yaml
        interceptors:
          - type: schema
            addMeta:
              timestamp:
                key: "@timestamp"
            remap:
              body:
                key: message
        ```

        转换后的event:
        ```json
        {
          "message": "this is raw data"
          "@timestamp": "2022-08-30T06:58:49.545Z",
        }
        ```

    === "增加_timestamp_，修改时区和格式，body修改为_log_"

        ```yaml
        interceptors:
          - type: schema
            addMeta:
              timestamp:
                key: "_timestamp_"
                location: Local
                layout: 2006-01-02T15:04:05Z07:00
            remap:
              body:
                key: _log_
        ```


## 配置

### addMeta
|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| addMeta |   |    非必填    |      | 增加系统元信息字段 |

#### timestamp
|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| addMeta.timestamp |   |    非必填    |      | 增加系统时间字段（source采集到数据的时间） |
| addMeta.timestamp.key | string  |    必填    |      | 系统时间的key |
| addMeta.timestamp.location | string  |    非必填    |   默认为空，即为UTC时间   | 增加的时间时区，另外还支持`Local` |
| addMeta.timestamp.layout |  string |    非必填    |  "2006-01-02T15:04:05.000Z"    | golang的时间类型layout，可参考https://go.dev/src/time/format.go |

#### pipelineName
|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| addMeta.pipelineName |   |    非必填    |      | 将pipelineName加入到event中 |
| addMeta.pipelineName.key |  string |    必填    |      | 增加后的字段的key |

!!! example

    ```yaml
    interceptors:
      - type: schema
        addMeta:
          pipelineName:
            key: pipeline
    ```
    转换后的event:
    ```json
    {
      "pipeline": "demo"
      ...
    }
    ```


#### sourceName

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| addMeta.sourceName |   |    非必填    |      | 将sourceName加入到event中 |
| addMeta.sourceName.key | string  |    必填    |      | 增加后的字段的key |

!!! example

    ```yaml
    interceptors:
      - type: schema
        addMeta:
          sourceName:
            key: source
    ```
    转换后的event:
    ```json
    {
      "source": "local"
      ...
    }
    ```

### remap
|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| remap |  map |    非必填    |      | 对字段进行转换，目前支持重命名 |
| remap.[originKey] |  string |    非必填    |      | 原始字段key |
| remap.[originKey].key |  string |    非必填    |      | 转换后的字段key |

!!! example

    ```yaml
    interceptors:
      - type: schema
        remap:
          body:
            key: msg
          state:
            key: meta
    ```
    转换前的event:
    ```json
    {
      "body": "this is log"
      "state": "ok",
    }
    ```

    转换后的event:
    ```json
    {
      "msg": "this is log"
      "meta": "ok",
    }

    ```
