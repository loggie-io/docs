
## 配置
Loggie的配置主要分为两类：
### 系统配置
全局的系统配置，启动参数中使用`-config.system`指定，包含如下：

- [**monitor**](global/monitor.md): 监控相关的配置
- [**discovery**](global/discovery.md): 服务发现与配置下发
- [**reload**](global/reload.md): 动态配置热加载
- [**defaults**](global/defaults.md): 全局默认配置
- [**http**](global/http.md): 管理和监控使用的http端口

???+ example "loggie.yml"

    ``` yaml
    # loggie.yml
    loggie:
      monitor:
        logger:
          period: 30s
          enabled: true
        listeners:
          filesource: ~
          filewatcher: ~
          reload: ~
          sink: ~
    
      discovery:
        enabled: false

      reload:
        enabled: true
        period: 10s

      defaults:
        sink:
          type: dev
        sources:
          - type: file
            watcher:
              cleanFiles:
                maxHistory: 1
      http:
        enabled: true
        port: 9196

    ```

### Pipeline配置
Pipeline的配置，通过启动参数`-config.pipeline`指定。表示队列使用的Source、Sink、Queue和Interceptor。

- [**Source**](pipelines/source/overview.md): 每个Pipeline可配置多个Source
- [**Interceptor**](pipelines/interceptor/normalize.md): 每个Pipeline可配置多个Interceptor
- [**Sink**](pipelines/sink/overview.md): 每个Pipeline可配置一个Sink
- [**Queue**](pipelines/queue/channel.md): 默认为channel队列，一般无需配置

???+ example "pipeline.yml"

    ``` yaml
    pipelines:
    - name: demo # pipeline name必填
      sources:
        - type: ${sourceType}
          name: access # source中name必填
          ...
      interceptors:
      - type: ${interceptorType}
        ...
      sink:
        type: ${sinkType}
        ...
    ```

## Kubernetes CRD
Loggie定义了以下几个CRD用于在Kubernetes集群环境里下发配置：  

- [**LogConfig**](discovery/kubernetes/logconfig.md)：namespace级别，表示一个Pipeline配置，可用于采集Pods的容器日志。  

- [**ClusterLogConfig**](discovery/kubernetes/clusterlogconfig.md)：cluster级别，表示一个Pipeline配置，包括集群级别的跨Namespace采集Pod容器日志，采集Node节点上的日志，以及为某个Loggie集群下发通用的pipeline配置。

- [**Sink**](discovery/kubernetes/sink.md)：cluster级别，表示一个sink配置，可以在LogConfig/ClusterLogConfig中引用该Sink。  

- [**Interceptors**](discovery/kubernetes/interceptors.md)：cluster级别，表示一个interceptors组，可以在LogConfig中引用该interceptors组。  

!!! note
    ClusterLogConfig/LogConfig中的pipeline可以定义sink和interceptor，用于该pipeline的sink/interceptor。  
    如果你希望在多个ClusterLogConfig/LogConfig中复用sink或者interceptor，可以创建Sink/Interceptor CR，在ClusterLogConfig/LogConfig中使用sinkRef/interceptorRef进行引用。
