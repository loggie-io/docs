# Logconfig 

表示一个日志采集任务或者一个Pipeline配置。  

!!! example

    ```yaml
    apiVersion: loggie.io/v1beta1
    kind: LogConfig
    metadata:
      name: nginx
      namespace: default
    spec:
      selector:
        type: pod
        labelSelector:
          app: nginx
      pipeline:
        sources: |
          - type: file
            name: mylog
            paths:
            - stdout
        sinkRef: default

    ```

## spec.selector
表示Pipeline配置适用的范围，可以选择采集一批Pods的日志，或者采集节点的日志等



### type: pod
采集Pods日志

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
    表示采集该namespace下的带有label `app: nginx`的所有Pods日志。

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
        type: pod
        nodeSelector:
          nodepool: test
    ```
    表示将配置的Pipelines下发至带有`nodepool: test`的所有node上。

### type: loggie
下发Pipeline配置至某个Loggie集群，通常需要配合`cluster`字段指定集群名使用。  

!!! example 
    ```yaml
    spec: 
      cluster: aggregator
      selector:
        type: loggie
          
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

### sources
在LogConfig中，如果`type: pod`，`file source`新增有几个专门针对容器化的参数：

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| containerName | string  |    非必填    |      | 表示指定采集的容器名称，建议在Pod里包含多个容器时填写 |
| matchFields |   |    非必填    |      | 将Pod中的信息加入到Fields中  |
| matchFields.labelKey | string数组  |    非必填    |      | 指定增加的Pod上的Label Key值，比如Pod上包含Label: `app: demo`，此处填写`labelKey: app`，此时会将Pod上的`app: demo` label增加到file source fields中，采集到的日志会被加上该label信息。适用于匹配的Pod的label存在不一致的场景。 |
| annotationKey | string数组  |    非必填    |      | 和上面labelKey类似，注入的为Pod Annoatation的值 |
| env | string数组  |    非必填    |      | 和上面labelKey类似，注入的为Pod Env环境变量的值 |


!!! example

    ```yaml
    apiVersion: loggie.io/v1beta1
    kind: LogConfig
    metadata:
      name: nginx
      namespace: default
    spec:
      selector:
        type: pod
        labelSelector:
          app: nginx
      pipeline:
        sources: |
          - type: file
            name: mylog
            containerName: nginx
            matchFields:
              labelKey: app
            paths:
            - stdout

    ```