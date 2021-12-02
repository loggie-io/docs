# Kubernetes下的Loggie配置

Kubernetes下的日志采集和主机下有什么不同，如何采集容器的日志？

Loggie定义了3个CRD：  

## [`LogConfig`](logconfig.md)
namespace级别，表示一个Pipeline配置，包括采集Pods的容器日志，采集Node节点上的日志等  
  
## [`Sink`](sink.md)
表示一个sink配置，可以在LogConfig中引用该Sink  

## [`Interceptors`](interceptors.md)
表示一个interceptors组，可以在LogConfig中引用该interceptors组  


