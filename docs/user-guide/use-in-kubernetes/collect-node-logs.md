# 使用Loggie采集Node节点日志

在Kubernetes集群中，除了采集Pod里的日志，还可能有采集Node节点上的一些诸如kubelet日志、系统日志等需求。  


## 配置说明
和采集容器不同的是，节点日志采集需使用集群级别的ClusterLogConfig，selector使用`type: node`，并且填写`nodeSelector`用于选择下发配置到哪些节点，同时需确保Node上包含这些labels。  

示例如下：  
!!! example
    ```yaml
    apiVersion: loggie.io/v1beta1
    kind: ClusterLogConfig
    metadata:
      name: varlog
    spec:
      selector:
        type: node
        nodeSelector:
          nodepool: demo
      pipeline:
        sources: |
          - type: file
            name: varlog
            paths:
              - /var/log/*.log
        sinkRef: default
        interceptorRef: default
    ```

另外应注意的是，如果需要采集Node节点上某路径的日志，需要Loggie同样挂载相同的路径，否则由于容器隔离性Loggie无法获取到节点的日志。  
比如采集Node上`/var/log/`路径下的日志，需要Loggie Agent增加挂载该路径。  