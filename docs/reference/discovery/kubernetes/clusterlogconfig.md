# ClusterLogConfig

表示选择一批Pod进行日志采集、采集Node节点上的日志或者将pipeline配置下发至指定cluster的Pipeline配置。  

!!! example

    ```yaml
    apiVersion: loggie.io/v1beta1
    kind:  ClusterLogConfig
    metadata:
      name: test
    spec:
      selector:
        type: node
        nodeSelector:
          nodepool: test
      pipeline:
        sources: |
          - type: file
            name: messages
            paths:
            - /var/log/messages
        sinkRef: default

    ```

## spec.selector
表示Pipeline配置适用的范围

### type: pod
通过Pipeline配置选择一批Pod进行日志采集

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| labelSelector | map  |    必填    |      | 通过该label来匹配Pods |


!!! example

    ```yaml
    spec: 
      selector:
        type: pod
        labelSelector:
          app: nginx
    ```
    表示采集带有标签 `app: nginx`的所有Pod的日志。

!!! warning
在`type: pod`时，下面的Pipeline只能使用file source，此时的场景只能是采集日志。

### type: node
下发Pipeline配置至该批节点。

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| nodeSelector | map  |    必填    |      | 通过label选择下发配置的node |


!!! example 
    ```yaml
    spec: 
      selector:
        type: node
        nodeSelector:
          nodepool: test
    ```
    表示将配置的Pipelines下发至带有`nodepool: test`的所有node上。

### type: cluster
下发Pipeline配置至某个Loggie集群，通常需要配合`cluster`字段指定集群名使用。  

!!! example 
    ```yaml
    spec:
      selector:
        cluster: aggregator
        type: cluster
          
    ```
    表示将配置的Pipelines下发至`cluster`为aggregator的Loggie集群。


### cluster

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| cluster | string  |    非必填    |  ""    | 表示配置指定下发的Loggie集群。当部署多套Loggie时，和全局系统配置`discovery.kubernetes.cluster`配套使用 |



## spec.pipelines
和在配置文件中Pipelines的区别在：  

- sources为实际为string，在yaml中使用`｜`表示保留换行符，同时增加了几个特殊的参数。
- 没有sink，只有sinkRef，表示引用的Sink CRD实例
- 没有interceptors，只有interceptorsRef，表示引用的Interceptors CRD实例