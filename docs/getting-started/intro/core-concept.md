# 核心概念

Loggie是一个基于Golang的轻量级、高性能、云原生日志采集Agent和中转处理Aggregator，支持多Pipeline和组件热插拔，提供了：

- :hammer:  **一栈式日志解决方案**：同时支持日志中转、过滤、解析、切分、日志报警等
- :cloud: **云原生的日志形态**：快速便捷的容器日志采集方式，原生的Kubernetes动态配置下发
- :key: **生产级的特性**：Loggie吸收了我们长期的大规模运维经验，形成了全方位的可观测性、快速排障、异常预警、自动化运维能力

## 架构

![](../imgs/loggie-arch.png)


## 概念
#### 核心数据流

- **Source**：输入源，表示一个具体的输入源，一个Pipeline可以有多个不同的输入源。比如file source表示日志文件采集源，Kafka source为读取Kafka的输入源。
- **Sink**：输出源，表示一个具体的输出源，一个Pipeline仅能配置一种类型的输出源，但是可以有多个并行实例。比如Elasticsearch sink表示日志数据将发送至远端的Elasticsearch。
- **Interceptor**：拦截器，表示一个日志数据处理组件，不同的拦截器根据实现可以进行日志的解析、切分、转换、限流等。一个Pipeline可以有多个Interceptor，数据流经过多个Interceptor被链式处理。
- **Queue**：队列，目前有内存队列。
- **Pipeline**：管道，source/interceptor/queue/sink共同组成了一个Pipeline，不同的Pipeline数据隔离。


#### 管理与控制

- **Discovery**：动态配置的下发，目前主要为Kubernetes下的日志配置，可以通过创建LogConfig等CRD实例的方式来采集容器日志。后续将陆续支持主机形态下的各种配置中心对接。  
- **Monitor EventBus**：各组件均可以通过publish数据到EventBus Topic中，由特定的Listener监听Topic并进行消费处理。主要用于监控数据的暴露或发送。  
- **Reloader**：用于配置的动态更新。



