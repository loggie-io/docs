# 日志增加加元信息

> 建议先了解Loggie内部日志数据[schema设计](../architecture/schema.md)。  


## 需求场景
Loggie部署在不同的环境中，如果需要在原始的日志数据里，增加一些元信息，同时兼容已有的格式，可以参考如下的办法。

### 场景1: 发送原始数据
正常情况，Loggie仅将原始数据发送给下游。

如果你配置：

!!! config "pipelines.yml"

    ```yaml
    pipelines:
      - name: local
        sources:
          - type: file
            name: demo
            paths:
              - /tmp/log/*.log
        sink:
          type: dev
          printEvents: true
          codec:
            pretty: true
    ```

sink输出的数据仅为原始数据：

```json
    {
        "body": "01-Dec-2021 03:13:58.298 INFO [main] Starting service [Catalina]"
    }
```

### 场景2: 添加fields自定义元信息

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

```yaml
        fields:
          topic: "loggie"
        fieldsUnderRoot: true
```

```json
    {
        "topic": "loggie",
        "body": "01-Dec-2021 03:13:58.298 INFO [main] Starting service [Catalina]"
    }
```

### 场景3: 添加日志采集file source的状态信息
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


### 场景4: 增加Kubernetes元信息
在Kubernetes的场景中，采集的容器日志，为了在查询的时候，使用namespace/podName等信息进行检索，往往需要增加相关的元数据。

我们可以在系统配置的discovery.kubernetes中，配置fields字段。

可参考[discovery](../../reference/global/discovery.md)。

### 场景5: 添加meta系统内置元信息

有一些Loggie系统内置的元信息，我们也希望发送给下游，这个时候，需要使用normalize interceptor中的addMeta processors。

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

当然，我们可能会觉得这些数据太多了，或者想对字段进行修改。我们就可以使用[normalize interceptor](../../reference/pipelines/interceptor/normalize.md)里的其他processor，进行drop、rename、copy、timestamp转换等操作。  

另外，这里的normalize interceptor配置在某个Pipeline中，如果希望全局生效，避免每个pipeline都配置该interceptor，可以在defaults中配置：

!!! config "loggie.yml"

    ```yaml
    defaults:
      interceptors:
      - type: normalize
        name: global # 用于区分pipelines中的normalize，避免pipeline中定义了normalize会覆盖这里的defaults
        order: 500   # 默认normalize的order值为900，这里定义一个相对较小值，可控制先执行defaults中的normalize
        processor:
        - addMeta: ~
    ```


### 场景6：beatsFormat快速转换

如果你期望的格式为类似：

```json
{
  "@timestamp": "2021-12-12T11:58:12.141267",
  "message": "01-Dec-2021 03:13:58.298 INFO [main] Starting service [Catalina]",
}
```

Loggie提供了一个内置的转换方式，可以在sink codec中设置[beatsFormat](../../reference/pipelines/sink/overview.md)如下:

!!! config 

    ```yaml
    sink:
      type: dev
      printEvents: true
      codec:
        type: json
        beatsFormat: true
    ```

会将默认的`body`改为`message`，同时增加`@timestamp`时间字段。