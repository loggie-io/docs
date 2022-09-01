# 日志格式与元信息字段

> 建议先了解Loggie内部日志数据[schema设计](../architecture/schema.md)。  

Loggie部署在不同的环境中，如果需要在原始的日志数据里，增加一些元信息，同时兼容已有的格式，可以参考如下的办法。

## 字段格式转换

### 使用schema interceptor

使用schema interceptor可以增加时间字段，以及pipelineName与sourceName字段。另外还可以对字段进行重命名，比如修改`body`为`message`。  
请参考[schema interceptor](../../reference/pipelines/interceptor/schema.md)。  

由于大部分情况下，我们需要全局生效，而不是仅仅只在某个pipeline里添加该interceptor，所以建议在系统配置的defaults中添加schema interceptor，
这样可以避免每个pipeline均需配置该interceptor。  

!!! config "loggie.yml"

    ```yaml
    loggie:
      defaults:
        interceptors:
          - type: schema
            name: global
            order: 700
            addMeta:
              timestamp:
                key: "@timestamp"
            remap:
              body:
                key: message
    
    ```


这里的name是为了增加标识，避免如果在pipeline中又新增schema interceptor会导致校验不通过。另外增加order字段为一个较小的值（默认为900)，这样default里的interceptor会优先于pipeline里定义的其他interceptor执行。

### 使用transformer interceptor

tranformer提供了更丰富的功能，可以应对复杂日志的场景。  
具体请参考[transformer interceptor](../../reference/pipelines/interceptor/transformer.md)。

## 添加元信息

### 添加fields自定义元信息

如果我们在source上配置了一些自定义的fields。

!!! config "pipelines.yml"

    ```yaml
    pipelines:
      - name: local
        sources:
          - type: file
            name: demo
            paths:
              - /tmp/log/*.log
            fields:
              topic: "loggie"
    
        sink:
          type: dev
          printEvents: true
          codec:
            pretty: true
    ```

那么sink输出的为：

```json
{
    "fields": {
        "topic": "loggie",
    },
    "body": "01-Dec-2021 03:13:58.298 INFO [main] Starting service [Catalina]"
}
```

当然我们也可以配置`fieldsUnderRoot: true`，让fields里的`key:value`和body同一层级。
!!! config "pipelines.yml"

    ```yaml
    pipelines:
      - name: local
        sources:
          - type: file
            fields:
              topic: "loggie"
            fieldsUnderRoot: true
    ...
    ```

```json
{
    "topic": "loggie",
    "body": "01-Dec-2021 03:13:58.298 INFO [main] Starting service [Catalina]"
}
```

### 添加日志采集file source的状态信息
在我们使用file source时，可能希望自动在日志原始数据里，增加一些日志采集的状态，比如采集的文件名称、采集的文件offsest等，file source提供了一个`addonMeta`配置，可快速enable。

示例：添加如下`addonMeta`，并设置为true。

!!! config "file source"

    ```yaml
    sources:
    - type: file
      paths:
      - /var/log/*.log
      addonMeta: true
    ```
    
此时，采集的event会变成类似如下：

!!! example 

    ```json
    {
      "body": "this is test",
      "state": {
        "pipeline": "local",
        "source": "demo",
        "filename": "/var/log/a.log",
        "timestamp": "2006-01-02T15:04:05.000Z",
        "offset": 1024,
        "bytes": 4096,
        "hostname": "node-1"
      }
    }
    ```

具体字段含义可参考[file source](../../reference/pipelines/source/file.md)


### 增加Kubernetes元信息
在Kubernetes的场景中，采集的容器日志，为了在查询的时候，使用namespace/podName等信息进行检索，往往需要增加相关的元数据。

我们可以在系统配置的discovery.kubernetes中，配置额外的k8s fields字段。

可参考[discovery](../../reference/global/discovery.md)。

### 添加meta系统内置元信息
有一些Loggie系统内置的元信息，我们也希望发送给下游，这个时候，需要使用normalize interceptor中的addMeta processors。
（需要注意的是，该操作会对采集传输性能有一定影响，正常情况下，并不建议使用该方式）

!!! config "pipelines.yml"

    ```yaml
    pipelines:
      - name: local
        sources:
          - type: file
            name: demo
            paths:
              - /tmp/log/*.log
            fields:
              topic: "loggie"
        interceptors:
          - type: normalize
            processors:
              - addMeta: ~
    
        sink:
          type: dev
          printEvents: true
          codec:
            pretty: true
    
    ```

配置了addMeta processor之后，默认会把所有的系统内置元信息输出。

默认Json格式输出示例如下：

!!! example

    ```json
    {
        "fields": {
            "topic": "loggie"
        },
        "meta": {
            "systemState": {
                "nextOffset": 720,
                "filename": "/tmp/log/a.log",
                "collectTime": "2022-03-08T11:33:47.369813+08:00",
                "contentBytes": 90,
                "jobUid": "43772050-16777231",
                "lineNumber": 8,
                "offset": 630
            },
            "systemProductTime": "2022-03-08T11:33:47.370166+08:00",
            "systemPipelineName": "local",
            "systemSourceName": "demo"
        },
        "body": "01-Dec-2021 03:13:58.298 INFO [main] Starting service [Catalina]"
    }
    ```

当然，我们可能会觉得这些数据太多了，或者想对字段进行修改。我们就可以使用[transformer interceptor](../../reference/pipelines/interceptor/transformer.md)里的action进行操作。  

