# 日志切分处理
> Loggie可使用[normalize interceptor](../../reference/pipelines/interceptor/normalize.md)来进行日志的切分和处理，将日志数据进行结构化的提取，同时可以对提取后的字段进行处理。  
> 建议先了解Loggie内部日志数据[schema设计](../architecture/schema.md)。  

## 需求场景

最主要的是对日志进行切分解析提取和处理。  

比如以下日志：

```
01-Dec-2021 03:13:58.298 INFO [main] Starting service [Catalina]
```

我们可能会需要将其中的日期、日志级别解析出来，最终形成：

```json
{
   "time": "01-Dec-2021 03:13:58.298",
   "level": "INFO",
   "message": "[main] Starting service [Catalina]"
}
```

这种结构化的数据，存储的时候便于过滤查询，或者根据日志里的时间来排序，而不是采集的时间戳，或者根据日志级别进行一些过滤，可以方便查询到ERROR级别的日志等等。  
当然不仅仅是像以上tomcat的运维类日志，还有诸如业务的一些订单等等日志，都有类似的需求和使用场景。  

!!! caution "关于stdout日志的解析提取"
    以下示例仅提供日志切分处理的参考思路，如果你需要提取容器标准输出的原始日志，请参考[采集容器日志](../use-in-kubernetes/collect-container-logs.md#_5)。

## 功能
目前已经支持的功能有：

- regex: 正则切分提取日志
- jsonDecode: 解析提取json格式的日志
- split: 通过分隔符来提取日志
- add/rename/drop/copy/underRoot指定字段
- convert类型转换
- timestamp: 转换指定字段的时间格式

normalize可以配置内部的processor顺序执行。

## 配置示例

日志切分处理在Loggie Agent端或者Loggie中转机侧均可，取决于我们是否需要中转机，以及希望日志处理这种CPU密集型的计算是分布在Agent上，由各个节点承担，还是希望在中转机集群中集中进行。  

下面以采集tomcat服务的标准输出和access日志为例，展示如何对标准输出格式进行处理和access日志进行字段切分。  
  
简单起见，示例使用CRD实例配置下发在Agent，同时使用dev sink直接输出处理结果展示。

### sink
创建如下的sink用于演示，将采集处理后的日志打印在所属节点Loggie的日志中。  

!!! example

    ```yaml
    apiVersion: loggie.io/v1beta1
    kind: Sink
    metadata:
      name: default
    spec:
      sink: |
        type: dev
        printEvents: true
        codec:
          type: json
          pretty: true
    ```

### interceptor
根据实际环境创建如下的interceptor。  

!!! example

    === "docker"

        ```yaml
        apiVersion: loggie.io/v1beta1
        kind: Interceptor
        metadata:
          name: tomcat
        spec:
          interceptors: |
            - type: normalize
              name: stdproc
              belongTo: ["stdout"]
              processors:
              - jsonDecode: ~
              - drop:
                  targets: ["stream", "time", "body"]
              - rename:
                  convert:
                  - from: "log"
                    to: "message"
            - type: normalize
              name: accproc
              belongTo: ["access"]
              processors:
              - regex:
                  pattern: '(?<ip>\S+) (?<id>\S+) (?<u>\S+) (?<time>\[.*?\]) (?<url>\".*?\") (?<status>\S+) (?<size>\S+)'
        ```

    === "containerd"

        ```yaml
        apiVersion: loggie.io/v1beta1
        kind: Interceptor
        metadata:
          name: tomcat
        spec:
          interceptors: |
            - type: normalize
              name: stdproc
              belongTo: ["stdout"]
              processors:
              - split:
                  separator: ' '
                  max: 4
                  keys: ["time", "std", "F", "message"]
              - drop:
                  targets: ["time", "std", "F", "body"]
            - type: normalize
              name: accproc
              belongTo: ["access"]
              processors:
              - regex:
                  pattern: '(?<ip>\S+) (?<id>\S+) (?<u>\S+) (?<time>\[.*?\]) (?<url>\".*?\") (?<status>\S+) (?<size>\S+)'

        ```

如果使用docker运行时，默认采集的标准输出为json格式。  
类似：
```json
{
"log":"I0610 08:29:07.698664 Waiting for caches to sync\n",
"stream":"stderr", 
"time":"2021-06-10T08:29:07.698731204Z"
}
```
我们需要将json格式的原始日志解析，一般还需要drop `stream`和`time`字段，并将其中的`log`字段key改为和其他格式一致的`body`或者`message`。  

使用filesource采集后，在Loggie里的原始格式为：
```
"body": '"log":"I0610 08:29:07.698664 Waiting for caches to sync\n","stream":"stderr", "time":"2021-06-10T08:29:07.698731204Z"'
...
```

可在normalize interceptors里，配置以下processor:  

1. **`jsonDecode`**:  
解析并提取字段，将Loggie里存储的日志格式变成：
```
"body": '"log":"I0610 08:29:07.698664 Waiting for caches to sync\n","stream":"stderr", "time":"2021-06-10T08:29:07.698731204Z"'
"log": "I0610 08:29:07.698664 Waiting for caches to sync\n"
"stream":"stderr"
"time":"2021-06-10T08:29:07.698731204Z"
...
```

1. **`drop`**:  
将`body`、`stream`和`time`丢弃。（`body`为内置字段表示原始日志数据，仅能丢弃，不能修改）

3. **`rename`**:  
将`log`改名为统一的字段比如`message`。 
 
最终发送的日志格式变成：  
```
"message": "I0610 08:29:07.698664 Waiting for caches to sync\n"
...
```


在runtime为containerd时，原始日志如下所示：

```
2021-12-01T03:13:58.298476921Z stderr F INFO [main] Starting service [Catalina]
```

会默认加上类似`2021-12-01T03:13:58.298476921Z stderr F`的前缀，一般我们并不需要这个日志，发送的时候只保留后面数据。  
和docker的方式类似，我们可以配置split等方式来切分处理。  

另外，由于stdout的日志和access日志格式不一样，所以使用了两个不同的normalize interceptor，配置了不同的processor，需要添加`name`来区分。  
最重要的是，这里使用了`belongTo`指定关联的`source`，让stdout日志和access日志分别采用不同的处理逻辑。  

### logconfig
配置source如下所示：

!!! example

    ```yaml
    apiVersion: loggie.io/v1beta1
    kind: LogConfig
    metadata:
      name: tomcat
      namespace: default
    spec:
      selector:
        labelSelector:
          app: tomcat
        type: pod
      pipeline:
        interceptorRef: tomcat
        sinkRef: dev
        sources: |
          - type: file
            name: stdout
            paths:
            - stdout
          - type: file
            name: access
            paths:
            - /usr/local/tomcat/logs/localhost_access_log.*.txt
    ```

注意这里的source.name和上面interceptor配置的belongTo需要关联上。  

创建完以上的cr后，便可以采集指定的Pod日志，并进行切分处理。  




