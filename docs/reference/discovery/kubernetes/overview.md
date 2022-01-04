# Kubernetes下的Loggie配置


Loggie定义了3个CRD用于在Kubernetes集群环境里下发配置：  

## [`LogConfig`](logconfig.md)
namespace级别，表示一个Pipeline配置，包括采集Pods的容器日志，采集Node节点上的日志，将pipeline配置下发至指定cluster的Loggie集群  
  
## [`Sink`](sink.md)
cluster级别，表示一个sink配置，可以在LogConfig中引用该Sink  

## [`Interceptors`](interceptors.md)
cluster级别，表示一个interceptors组，可以在LogConfig中引用该interceptors组  


