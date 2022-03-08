# addK8sMeta

用于从event中的某些字段（比如日志文件的路径中），提取到：

- `pod.uid`
- `namespace`与`pod.name`
- `container.id`

以上3种任意其一，此时Loggie可索引到具体的Pod信息，并添加`${node.name}`、`${namespace}`、`${pod.uid}`、`${pod.name}`等元信息作加入到event中，用于后续的分析处理。  
属于source interceptor。

!!! example

    ```yaml
    interceptors:
    - type: addK8sMeta
      pattern: "/var/log/${pod.uid}/${pod.name}/"
      addFields:
        nodename: "${node.name}"
        namespace: "${namespace}"
        podname: "${pod.name}"
    ```

## patternFields

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| patternFields | string  |    非必填    |  默认会从event中获取系统字段里的filename    | 用于提取的字段 |

## pattern

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| pattern | string  |    必填    |      | 提取字段的匹配模型 |

必须包含有：

- `pod.uid`
- `namespace与pod.name`
- `container.id`

其中之一。

比如：`/var/log/${pod.uid}/${pod.name}/`

## fieldsName

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| fieldsName | string  |    非必填    |  kubernetes    | 添加元信息的字段 |

## addFields

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| addFields | map  |    非必填    |      | 需要添加的元信息 |

目前支持添加的元信息字段有：

- `${node.name}`
- `${namespace}`
- `${pod.uid}`
- `${pod.name}`