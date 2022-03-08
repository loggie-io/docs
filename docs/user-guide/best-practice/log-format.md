# 日志格式转换

> 建议先了解Loggie内部日志数据[schema设计](../architecture/schema.md)。  


日志格式转换，主要是对Loggie内置的一些字段进行处理和转换，一般会依赖[日志切分处理](log-process.md)。

## 需求场景
Loggie部署在不同的环境中，如果需要兼容已有的格式，可以参考如下的办法。

### 示例1: 发送原始数据
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

### 示例2: 添加fields自定义元信息

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

### 示例3: 添加meta系统内置元信息

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


### 示例4：beatsFormat快速转换

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