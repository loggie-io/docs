# 日志格式转换

> 在sink的[codec](../../reference/pipelines/sink/codec.md)中对输出的日志格式进行配置。  
> 建议先了解Loggie内部日志数据[schema设计](../architecture/schema.md)。  

## 需求场景
默认情况下，Loggie输出的日志字段格式类似如下所示：
!!! example "Event"
    ```
    {
      "body": "01-Dec-2021 03:13:58.298 INFO [main] Starting service [Catalina]",
      "fields": {
          "test": "demo"
      },
      "systemPipelineName": "svcA",
      "systemSourceName": "accesslog",
      "systemState": {
          "nextOffset": 2790,
          "filename": "/tmp/log/a.log",
          "collectTime": "2021-12-12T11:58:12.141267+08:00",
          "contentBytes": 160,
          "jobUid": "39100914-16777234",
          "lineNumber": 39,
          "offset": 2630
      }
    }
    ```

如果需要兼容现有的格式，可以参考如下的办法。

## 功能与配置

以下默认配置的sink codec是pipeline级别，如果你需要整体替换，请在系统配置defaults中进行全局修改。  

### 1. 常规样式

codec可以配置:

!!! config 
    ```yaml
    sink:
      type: dev
      printEvents: true
      codec:
        type: json
        beatsFormat: true
        prune: true
    ```

#### # codec.beatsFormat
将默认的`body`改为`message`，同时增加`@timestamp`时间字段。

此时日志格式为：

!!! example  "Event"
    ```json
    {
      "@timestamp": "2021-12-12T11:58:12.141267",
      "message": "01-Dec-2021 03:13:58.298 INFO [main] Starting service [Catalina]",
      "fields": {
          "test": "demo"
      },
      "systemPipelineName": "svcA",
      "systemSourceName": "accesslog",
      "systemState": {
          "nextOffset": 2790,
          "filename": "/tmp/log/a.log",
          "collectTime": "2021-12-12T11:58:12.141267+08:00",
          "contentBytes": 160,
          "jobUid": "39100914-16777234",
          "lineNumber": 39,
          "offset": 2630
      }
    }
    ```

#### # codec.prune
精简模式，不发送system开头系统保留字段。

!!! example "Event"
    ```json
    {
      "@timestamp": "2021-12-12T11:58:12.141267",
      "message": "01-Dec-2021 03:13:58.298 INFO [main] Starting service [Catalina]",
      "fields": {
          "test": "demo"
      }
    }
    ```


以上配置在sink端无额外性能开销，可放心使用。  

### 2. 自定义转换

和`normalize interceptor`类似，codec可配置transformer可以用于字段的格式化修改。  

假设我们需要将原始日志，转换成以下的日志格式：
!!! example
    ```json
    {
      "@timestamp": "2021-12-12T11:58:12.141267",
      "message": "01-Dec-2021 03:13:58.298 INFO [main] Starting service [Catalina]",
      "test": "demo"
      "meta": {
          "nextOffset": 2790,
          "filename": "/tmp/log/a.log",
          "collectTime": "2021-12-12T11:58:12.141267+08:00",
          "contentBytes": 160,
          "jobUid": "39100914-16777234",
          "lineNumber": 39,
          "offset": 2630
      }
    }
    ```

相比原始日志格式，需要做如下的几个变动：

**1. body重命名为message**

配置：
!!! config
    ```yaml
    codec:
      type: json
      beatsFormat: true
    ```

**2. 将fields字段的内容放在最外层**

建议直接在source里配置`fieldsUnderRoot`:

!!! config
    ```yaml
    sources:
    - type: file
      fields:
        test: demo
      fieldsUnderRoot: true
    ```


**3. 将systemState改名为meta**
!!! config
    ```yaml
    codec:
      type: json
      codec:
        type: json
        beatsFormat: true
        transformer:
        - rename:
            target:
            - from: systemState
              to: meta
    ```

**4. 去掉systemPipelineName和systemSourceName字段和meta.collectTime字段**
!!! config
    ```yaml
    codec:
      type: json
      codec:
        type: json
        beatsFormat: true
        transformer:
        - rename:
            target:
            - from: systemState
              to: meta
        - drop:
            target: ["systemPipelineName", "systemSourceName", "meta.collectTime"]
        
    ```
其中嵌套的字段名称支持通过`.`来引用。  

