# Loggie的优势与特性


Loggie支持多个Pipeline，每个Pipeline都基于简单直观的source->interceptor->sink的架构。
这样设计带来的好处有：

## 强隔离
多Pipeline设计，减少互相干扰。比如我们可以将重要的业务日志放在一个Pipeline中，其他的不重要的日志配置为另外的Pipeline，不重要的日志配置变动、发生下游堵塞时，不会影响重要日志的采集和发送。

## 通用性更好
在一些场景下，我们可能会将不同类型的服务混合部署在一个节点上，很可能他们的日志会发送到不同的Kafka集群中，如果只有一个全局的输出源，需要在节点上部署两个Agent，如果使用Loggie则只需要使用不同的Pipeline即可，每个Pipeline配置不同的Sink，减少部署成本。  
我们甚至可以采集相同的日志，发送到不同的后端输出源，根据实际需求灵活配置。  

## 灵活、热插拔、可扩展
本质上source->interceptor->sink架构是一个数据流式的设计，不同类型的source/interceptor/sink的排列组合，可以满足日志的不同需求，
Loggie并没有将interceptor更细化的分类成比如Filter/Formater等类型，interceptor承担了除了source读取，sink发送之外的大部分工作，只需要配置不同的interceptor就可以拥有中转、过滤、解析、切分、日志报警等能力。
于是，Loggie可以：

- 独立部署当作日志中转机
- 在Agent端或者中转机端解析处理日志
- 非日志采集场景也可以使用：比如监听k8s events，比如定时同步数据等

在部署和维护层面带来的好处是：  
如果之前采用常规的ELK架构，使用Filebeat采集日志、Logstash中转和解析日志，由于Filebeat和Logstash是两个不同的语言栈，带来排查问题和扩展开发成本均较高。  
改用Loggie后我们无需维护两个项目，甚至如果没有中转机的需求，可以选择在Agent端配置日志解析。  

另外，目前针对日志报警，开源的方案一般为使用elastAlert，但是elastAlert无法直接对接AlertManager，并且在高可用等方面存在问题，同时会强依赖Elasticsearch。所以如果使用Loggie，可以无需引入额外的组件，直接使用Loggie来检测异常日志并接入报警。  


## 可快速方便的写一个组件
Loggie基于微内核的架构，所有的source/interceptor/sink/queue都被抽象成component，只需要在代码中实现Golang接口，即可方便的研发一个component。  
如果Loggie在某些场景下，无法满足你的需求，可以尝试写一个自己的component。  
比如需要Loggie转换成特定的日志格式，可以写一个interceptor去处理；需要Loggie将采集的日志发送至尚未支持的服务，可以写一个sink。  
当然，Loggie使用Golang编写，所以你目前需要用Golang来写component。Golang和Java或者C/C++相比，在性能和研发效率上有一个折中，更适合类似日志Agent的场景。  

## 更方便的Kubernetes容器日志采集，更好的云原生支持
如果你尝试过在Kubernetes环境下采集日志，应该遇到过这些问题：

- 采用sidecar还是daemonset部署日志Agent？
- 使用什么方式挂载日志文件？难道只推荐用stdout输出日志吗？
- 采集的日志如何和Pod、Namespace、Node等信息关联上？
- Kubernetes下Pod会动态的创建和销毁，迁移到了其他的Node，相应的日志Agent配置怎么办呢？
  
如果你使用Loggie，你可以：

1. 快速部署，支持各种部署架构
2. 只需要使用ClusterLogConfig/LogConfig CRD配置管理日志配置，更方便的接入各类容器云平台，对业务无侵入，无需关心Pod迁移等，无需手动在节点上操作配置日志文件，同时可配置注入Namespace/PodName/NodeName等元信息供查询使用


## 更好的稳定性、更详细的监控指标、更方便的排障方式
在实际的生产环境中，日志Agent本身的稳定性很重要，同时不影响业务也很重要。
Loggie可配置限流interceptor，在日志量太大时，可以避免发送日志数据占据了太多网络带宽。  
Loggie有合理的文件句柄处理机制，避免fd被占用的各种异常场景导致节点不稳定。  

另外，Loggie结合我们在各种环境中遇到的各种问题，针对性的检测暴露出相应的指标。  
比如指标支持采集和发送延迟检测。比如针对文件size增长太快，或者文件size太大等场景，支持该类metric上报。  

同时Loggie支持原生Prometheus metric，可避免额外部署exporter带来的部署成本和资源消耗。Loggie还提供了完善的Grafana监控图表，可以方便快速接入使用。  

## 更低的资源占用，更好的性能
Loggie基于Golang编写，在代码层面我们有很多优化，在较少资源占用的同时，还可提供强大的吞吐性能。  

