# 数据格式

!!! info
    了解Loggie内部数据格式的设计，能帮助我们配置合适的日志处理和日志格式转换

## 结构设计
在Loggie内部的日志数据，会被存储为如下`key:value`的格式：

!!! example
    ```json
    "body": "xxxxxxx",
    "key1": "value1",
    "key2": "value2"
    
    ```

其中`body`为source接收到的原始内容，比如日志采集source，这里的`body`为文件采集到的原始日志数据。  
其他的字段，在Loggie内部被统一称为header。header里一般包括：

- 系统保留字段。统一system开头，比如systemPipelineName、systemSourceName、systemState等
- 用户使用的字段。比如用户自己添加到source里的fields字段，用户切分日志后得到的字段等

默认Json格式输出示例如下：

!!! example
    ```json
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


## 格式转换
如果以上的格式不满足需求，可以使用sink里的 **codec** 修改输出的字段：

- [配置](../../reference/pipelines/sink/codec.md)
- [使用示例](../best-practice/log-format.md)

## 日志切分
对于原始日志数据的切分与处理，请使用 **normalize interceptor**：

- [配置](../../reference/pipelines/interceptor/normalize.md)
- [使用示例](../best-practice/log-process.md)


!!! caution
    格式转换和日志切分两者区别：

    - normalize interceptor除了可以drop `body`字段外，无法修改系统保留的字段（修改可能影响内部逻辑）。codec.transformer可以任意修改字段，但存在反序列化的性能开销。
    - normalize interceptor可以通过配置`belongTo`指定关联source使用，codec.transformer则为pipeline级别，同时可以在系统配置中使用defaults来全局生效。  