# ClusterLogConfig

Cluster级别CRD，可用于：

- 采集任意Namespace的Pod日志
- 采集Node节点上的日志
- 将Pipeline配置下发至指定Loggie集群

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


## spec.namespaceSelector
技术提供方：[<img src="https://www.eoitek.com/static/image/eoiicon.ico" height="18px"/>擎创科技](https://www.eoitek.com/)

表示匹配指定命名空间下的全部POD

  !!! example
    ```yaml
      namespaceSelector:
      - test1
      - test2
    ```

## spec.excludeNamespaceSelector
技术提供方：[<img src="https://www.eoitek.com/static/image/eoiicon.ico" height="18px"/>擎创科技](https://www.eoitek.com/)

表示排除指定命名空间下的全部POD

!!! example
    ```yaml
      excludeNamespaceSelector:
      - test1
      - test2
    ```

### type: pod
通过Pipeline配置选择一批Pod进行日志采集

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| labelSelector | map  |    必填    |      | 通过该label来匹配Pods，支持使用`*`来匹配所有的value，比如`app: '*'` |


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

### type: workload

技术提供方：[<img src="https://www.eoitek.com/static/image/eoiicon.ico" height="18px"/>擎创科技](https://www.eoitek.com/)

通过Pipeline配置选择一批负载进行日志采集

| `字段` | `类型`  | `是否必填` |  `默认值`  |  `含义`  |
|------|-------|-----| --------- | -------- |
| type | array | 非必填  |      | 通过该label来匹配Pods，支持使用`*`来匹配所有的value，比如`app: *` |
| nameSelector | array   | 必填  |      | 通过该label来匹配Pods，支持使用`*`来匹配所有的value，比如`app: *` |
| namespaceSelector | array   | 必填  |      | 通过该label来匹配Pods，支持使用`*`来匹配所有的value，比如`app: *` |
| excludeNamespaceSelector | array   | 必填  |      | 通过该label来匹配Pods，支持使用`*`来匹配所有的value，比如`app: *` |


!!! example

    ```yaml
    apiVersion: loggie.io/v1beta1
    kind:  ClusterLogConfig
    metadata:
        name: globalstdout
    spec:
        selector:
            type: workload
        workload_selector:
          - type:
              - Deployment
            nameSelector:
              - default1
              - default2
            nameSpaceSelector:
              - default1
              - default2

        pipeline:
            sources: |
              - type: file
                name: stdout
                paths:
                  - stdout
                sinkRef: default
    ```
    表示采集default1和default2的Deployment下名字为default1和default2的所有Pod的日志。


### cluster

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| cluster | string  |    非必填    |  ""    | 表示配置指定下发的Loggie集群。当部署多套Loggie时，和全局系统配置`discovery.kubernetes.cluster`配套使用 |



## spec.pipeline

基本配置和LogConfig一致。

### sources
在ClusterLogConfig中，`source`新增以下额外的参数：

#### typeNodeFields
如果当`type: node`时：

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| typeNodeFields | map  |    非必填    |      | 和全局配置discovery.kubernetes中的[typeNodeFields](../../global/discovery.md#typenodefields支持的变量)一样，区别是这里为clusterlogconfig级别生效 |

