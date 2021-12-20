# 使用Loggie中转机

Loggie可以部署为Agent，同时支持独立部署，进行聚合、转发和处理。  

## 准备：选择架构
使用中转机架构的方式一般有多种，常见的有：

- `Agent -> Aggregator`: Agent直接发送至Aggregator，Aggregator再发送至后端存储。  
- `Agent -> MQ -> Aggregator`: Agent发送至消息队列，比如Kafka，然后Aggregator再消费Kafka消息发送至后端。  

是否引入Kafka等消息队列，主要取决于自身的场景需求和数据的量级。  

## 准备：部署

1. 部署Agent

2. 部署[Aggregator](../../getting-started/install/kubernetes.md#loggie-aggregator)

部署为Aggregator类型时，请务必在Kubernetes配置中指定`cluster`集群名称。  

为了查看输出日志更加方便，可以在Loggie的启动参数中加上`-log.jsonFormat=false`，使用非json格式的日志打印。  

## 配置使用

### Agent

采集日志的LogConfig配置无区别，只需要修改sink发送至Loggie Aggregator或者Kafka即可。  

如果仅仅修改`sink`或者`interceptor` CR本身不会立即同步到已经使用的LogConfig里，只有等待某些事件触发后（LogConfig本身被修改、关联Pod有重建等）才会被刷新配置。如果需要立刻更新，可以考虑重启Loggie本身。  

被采集的容器创建以及匹配的LogConfig，请参考[Loggie采集容器日志](./collect-container-logs.md)。

这里我们将其中的sink修改为以下示例：  

!!! example
    
    === "Aggregator"
        ```yaml
        apiVersion: loggie.io/v1beta1
        kind: Sink
        metadata:
          name: aggregator
        spec:
          sink: |
            type: grpc
            host: "loggie-aggregator.loggie-aggregator:6166"
        ```

    === "Kafka"
        ```yaml
        apiVersion: loggie.io/v1beta1
        kind: Sink
        metadata:
          name: kafka
        spec:
          sink: |
            type: kafka
            brokers: ["127.0.0.1:6400"]
            topic: "log-${fields.topic}"
        ```

### Aggregator

配置LogConfig下发至Aggregator本质上和Agent侧无区别，只需要修改其中`selector`部分。  

类似如下：
!!! example
    
    ```yaml
    apiVersion: loggie.io/v1beta1
    kind: LogConfig
    metadata:
      name: aggre
      namespace: default
    spec:
      selector:
        type: loggie
        cluster: aggregator
      pipeline:
        sources: |
          - type: grpc
            name: rec1
            port: 6066
        sinkRef: dev
    ```

    ```yaml
    apiVersion: loggie.io/v1beta1
    kind: Sink
    metadata:
      name: dev
    spec:
      sink: |
        type: dev
        printEvents: true
        codec:
          type: json
          pretty: true
    ```

`type: loggie`表示选择下发配置到`cluster`指定的Loggie集群，即我们刚部署的中转机集群，如果不填写会将配置指定到默认的Agent集群，导致无法生效。  

这里的source为Grpc，接收Agent Grpc sink发出的数据，然后转发至自身sinkRef指定的sink中。这里我们创建了一个dev sink用于查看中转机输出的数据。  

## 查看日志

通过`kubectl -nloggie-aggregator logs -f <podName> --tail=200`命令。  
可以在中转机节点上查看到类似如下日志：

!!! example "events"
    ```json
    2021-12-20 09:58:50 INF go/src/loggie.io/loggie/pkg/sink/dev/sink.go:98 > event: {
        "body": "14-Dec-2021 06:19:58.306 INFO [main] org.apache.catalina.startup.Catalina.start Server startup in [141] milliseconds",
        "systemSourceName": "rec1",
        "fields": {
            "podname": "tomcat-684c698b66-gkrfs",
            "containername": "tomcat",
            "logconfig": "tomcat",
            "namespace": "default",
            "nodename": "kind-control-plane"
        },
        "systemState": {
            "collectTime": "2021-12-20T09:58:46.346170897Z",
            "contentBytes": 117,
            "jobUid": "1320468-64769",
            "lineNumber": 43,
            "offset": 6028,
            "nextOffset": 6145,
            "filename": "/var/lib/kubelet/pods/89772369-2fd6-4b64-909f-aa10d81326c5/volumes/kubernetes.io~empty-dir/log/catalina.2021-12-14.log"
        },
        "systemPipelineName": "default/aggregator/"
    }
    ```
