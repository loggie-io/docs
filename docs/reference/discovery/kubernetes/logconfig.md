# Logconfig 


## spec.selector
表示Pipeline配置适用的范围，可以选择采集一批Pods的日志，或者采集节点的日志等



### type: pod
采集Pods日志

- `labelSelector`: map，通过该label来匹配Pods

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

- `nodeSelector`: map，通过label选择下发配置的node

!!! example 
    ```yaml
    spec: 
      selector:
        type: pod
        nodeSelector:
          nodepool: test
    ```
    表示将配置的Pipelines下发至带有`nodepool: test`的所有node上。


### cluster
表示指定下发的日志集群，非必填，默认为空。仅当部署多套Loggie时，和全局配置`discovery.kubernetes.cluster`配套使用。  


## spec.pipelines
和在配置文件中Pipelines的区别在：
- sources为实际为string，在yaml中使用`｜`表示保留换行符，同时增加了几个特殊的参数。
- 没有sink，只有sinkRef，表示引用的Sink CRD实例
- 没有interceptors，只有interceptorsRef，表示引用的Interceptors CRD实例

### sources
在LogConfig中，新增的参数为：

- containerName: string，非必填，表示指定采集的容器名称，建议在Pod里包含多个容器时填写
- matchFields: map，将Pod中的信息加入到Fields中 
  - labelKey: string数组，指定增加的Pod上的Label Key值，比如Pod上包含Label: `app: demo`，此处填写`labelKey: app`，此时会将Pod上的`app: demo` label增加到file source fields中，采集到的日志会被加上该label信息。适用于匹配的Pod的label存在不一致的场景。  
  - annotationKey: string数组，和上面labelKey类似，注入的为Pod Annoatation的值。  
  - env: string数组，和上面labelKey类似，注入的为Pod Env环境变量的值。  
