# addK8sMeta

用于从event中的某些字段（比如日志文件的路径中），获取到：

- `pod.uid`
- `namespace`与`pod.name`
- `container.id`

以上3种任意其一的索引信息，此时Loggie可根据该索引查询到具体的Pod，并添加额外的kubernetes`${node.name}`、`${namespace}`、`${pod.uid}`、`${pod.name}`等元信息作加入到event中，用于后续的分析处理。  
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

## patternFields

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| patternFields | string  |    非必填    |  默认会从event中获取系统字段里的filename，此时需要使用file source    | 从event中用于提取的pattern的字段 |

## fieldsName

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| fieldsName | string  |    非必填    |  kubernetes    | 添加元信息的字段 |

## addFields

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| addFields | map  |    非必填    |      | 需要添加的元信息 |

目前支持添加的元信息字段有：

- `${cluster}`：集群信息，为系统配置中`discovery.kubernetes.cluster`字段。
- `${node.name}`
- `${namespace}`
- `${workload.kind}`：Deployment/StatefulSet/DaemonSet/Job等
- `${workload.name}`：工作负载的名称
- `${pod.uid}`
- `${pod.name}`