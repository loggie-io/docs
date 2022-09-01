# 日志切分处理
> Loggie可使用[transformer interceptor](../../reference/pipelines/interceptor/transformer.md)来进行日志的切分和处理，将日志数据进行结构化的提取，同时可以对提取后的字段进行处理。  
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


## 配置示例

日志切分处理在Loggie Agent端或者Loggie中转机侧均可，取决于我们是否需要中转机，以及希望日志处理这种CPU密集型的计算是分布在Agent上，由各个节点承担，还是希望在中转机集群中集中进行。  

下面以采集tomcat服务的access日志为例，展示如何对access日志进行字段切分。  
  
简单起见，示例使用CRD实例配置下发在Agent，同时使用dev sink直接输出处理结果展示。

### 创建tomcat deployment
请[参考](../use-in-kubernetes/collect-container-logs.md#_3)

### 创建logconfig
配置logconfig如下所示：

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
        sources: |
          - type: file
            name: access
            paths:
            - /usr/local/tomcat/logs/localhost_access_log.*.txt

        interceptors: |
            - type: transformer
              actions:
                - action: regex(body)
                  pattern: (?<ip>\S+) (?<id>\S+) (?<u>\S+) (?<time>\[.*?\]) (?<url>\".*?\") (?<status>\S+) (?<size>\S+)

        sink: |
          type: dev
          printEvents: true
          codec:
            type: json
            pretty: true
    ```

这里我们在transformer interceptors里，配置了regex action，针对access日志进行正则提取。

原始的access日志大概如下所示：
```
10.244.0.1 - - [31/Aug/2022:03:13:40 +0000] "GET / HTTP/1.1" 404 683
```

经过transformer处理后，我们可以通过`kubectl -nloggie logs -f <loggie-pod-name> --tail=100`来查看输出的日志。

转换后的event示例如下：
```json
{
    "status": "404",
    "size": "683",
    "fields": {
        "logconfig": "tomcat",
        "namespace": "test1",
        "nodename": "kind-control-plane",
        "podname": "tomcat-85c84988d8-frs4n",
        "containername": "tomcat"
    },
    "ip": "10.244.0.1",
    "id": "-",
    "u": "-",
    "time": "[31/Aug/2022:03:13:40 +0000]",
    "url": "\"GET / HTTP/1.1\""
}
```

