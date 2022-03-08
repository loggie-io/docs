# 业务日志报警

除了Loggie本身的报警，业务日志本身的监控报警也是一个常用的功能，比如在日志中包含了ERROR日志，可以发送报警，这种报警会更贴近业务本身，是基于metrics报警的一种很好的补充。  

## 使用方式

有以下两种方式可以选择：

- **采集链路检测报警**：Loggie可以在Agent采集日志的时候，或者在中转机转发的时候，检测到匹配的异常日志，然后发送报警
- **独立链路检测报警**：单独部署Loggie，使用Elasticsearch source或者其他的source查询日志，然后匹配检测发送报警

## 采集链路检测

### 原理
采集链路不需要独立部署Loggie，但是由于在采集的数据链路上进行匹配，理论上会对传输性能造成一定影响，但胜在方便简单。  

`logAlert interceptor`用于在日志传输的时候检测异常日志，异常日志会被封装成报警的事件发送至monitor eventbus的`logAlert` topic，由`logAlert listener`来消费。目前`logAlert listener`支持发送至Prometheus AlertManager。有发送至其他报警渠道的需求，欢迎提Issues或者PR。  

### 配置示例

配置新增`logAlert listener`：

!!! config

    ```yaml
    loggie:
      monitor:
        logger:
          period: 30s
          enabled: true
        listeners:
          logAlert:
            alertManagerAddress: ["http://127.0.0.1:9093"]
            bufferSize: 100
            batchTimeout: 10s
            batchSize: 10
          filesource: ~
          filewatcher: ~
          reload: ~
          queue: ~
          sink: ~
      http:
        enabled: true
        port: 9196
    ```

增加`logAlert interceptor`，同时在ClusterLogConfig/LogConfig中引用：

!!! config

    ```yaml
    apiVersion: loggie.io/v1beta1
    kind: Interceptor
    metadata:
      name: logalert
    spec:
      interceptors: |
        - type: logAlert
          matcher:
            contains: ["err"]
    ```

可以配置alertManager的webhook发送至其他服务来接收报警。  
!!! config
    ```yaml
    receivers:
    - name: webhook
      webhook_configs:
        - url: http://127.0.0.1:8787/webhook
          send_resolved: true
    ```


发送成功后我们在alertManager里可以查看到类似的日志：
```
ts=2021-12-22T13:33:08.639Z caller=log.go:124 level=debug component=dispatcher msg="Received alert" alert=[6b723d0][active]
ts=2021-12-22T13:33:38.640Z caller=log.go:124 level=debug component=dispatcher aggrGroup={}:{} msg=flushing alerts=[[6b723d0][active]]
ts=2021-12-22T13:33:38.642Z caller=log.go:124 level=debug component=dispatcher receiver=webhook integration=webhook[0] msg="Notify success" attempts=1

```

同时webhook接收到类似的报警：

!!! example

    ```json
    {
    "receiver": "webhook",
    "status": "firing",
    "alerts": [
        {
            "status": "firing",
            "labels": {
                "host": "fuyideMacBook-Pro.local",
                "source": "a"
            },
            "annotations": {
                "message": "10.244.0.1 - - [13/Dec/2021:12:40:48 +0000] error \"GET / HTTP/1.1\" 404 683",
                "reason": "contained error"
            },
            "startsAt": "2021-12-22T21:33:08.638086+08:00",
            "endsAt": "0001-01-01T00:00:00Z",
            "generatorURL": "",
            "fingerprint": "6b723d0e395b14dc"
        }
    ],
    "groupLabels": {},
    "commonLabels": {
        "host": "node1",
        "source": "a"
    },
    "commonAnnotations": {
        "message": "10.244.0.1 - - [13/Dec/2021:12:40:48 +0000] error \"GET / HTTP/1.1\" 404 683",
        "reason": "contained error"
    },
    "externalURL": "http://xxxxxx:9093",
    "version": "4",
    "groupKey": "{}:{}",
    "truncatedAlerts": 0
    }
    ```


## 独立链路检测
!!! info
    马上放出，敬请期待