# Logconfig 

namespace级别CRD，表示一个日志采集任务，用于采集Pod容器日志。  

!!! example

    === "直接定义sink/interceptor方式"

        ```yaml
        apiVersion: loggie.io/v1beta1
        kind: LogConfig
        metadata:
          name: tomcat
          namespace: default
        spec:
          selector:
            type: pod
            labelSelector:
              app: tomcat
          pipeline:
            sources: |
              - type: file
                name: common
                paths:
                  - stdout
            sink: |
              type: dev
              printEvents: false
            interceptors: |
              - type: rateLimit
                qps: 90000
        ```

    === "引用sink和interceptor方式"

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
            interceptorRef: default 
        ```


## spec.selector
表示Pipeline配置适用的范围，可以选择采集一批Pods的日志


### type: pod
采集Pods日志

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
    表示采集该namespace下的带有label `app: nginx`的所有Pods日志。

!!! warning
    在`type: pod`时，下面的Pipeline只支持使用file source，此时的场景只能是采集日志。

### cluster

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| cluster | string  |    非必填    |  ""    | 表示配置指定下发的Loggie集群。当部署多套Loggie时，和全局系统配置`discovery.kubernetes.cluster`配套使用 |



## spec.pipeline

表示一个Pipeline，不支持填写多个Pipeline。  

和在配置文件中Pipelines的区别在：  

- sources为实际为string，在yaml中使用`｜`表示保留换行符
- 没有sink，只有sinkRef，表示引用的Sink CRD实例
- 没有interceptors，只有interceptorRef，表示引用的Interceptor CRD实例

### sources
在LogConfig中，如果`type: pod`，`file source`新增几个专门针对容器化的参数：

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| containerName | string  |    非必填    |      | 表示指定采集的容器名称，建议在Pod里包含多个容器时填写 |
| excludeContainerPatterns | string数组  |    非必填    |      | 排除的容器名称，使用正则表达式形式 |

#### sources.matchFields
非必填, 将Pod中的信息加入到Fields中

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| labelKey | string数组  |    非必填    |      | 指定增加的Pod上的Label Key值，比如Pod上包含Label: `app: demo`，此处填写`labelKey: app`，此时会将Pod上的`app: demo` label增加到file source fields中，采集到的日志会被加上该label信息。适用于匹配的Pod的label存在不一致的场景。支持配置为"*"的方式获取所有的label |
| annotationKey | string数组  |    非必填    |      | 和上面labelKey类似，注入的为Pod Annoatation的值，支持配置为"*"的方式获取所有的annotation |
| env | string数组  |    非必填    |      | 和上面labelKey类似，注入的为Pod Env环境变量的值，支持配置为"*"的方式获取所有的env |
| reformatKeys |   |    非必填    |      | 重新格式化key |
| reformatKeys.label | fmt参数数组  |    非必填    |      | 重新格式化label key |
| reformatKeys.annotation | fmt参数数组  |    非必填    |      |  重新格式化annotation key|
| reformatKeys.env | fmt参数数组  |    非必填    |      | 重新格式化env key |


**fmt参数**

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| regex | string  |    非必填    |      | 匹配的正则表达式 |
| replace | string  |    非必填    |      | 重新渲染的格式 |

!!! example "reformatKeys"

    假设pod labels为`aa.bb/foo=bar`
    配置reformatKeys如下：
    ```
    matchFields:
	 reformatKeys:
	   label:
	   - regex: aa.bb/(.*)
	     replace: pre-${1}
    ```
    最终添加到日志的元信息为：`pre-foo=bar`


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
              labelKey: ["app"]
            paths:
            - stdout

    ```

### interceptors
|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| interceptors | string  |    非必填    |      | 表示该Pipeline的interceptor，使用方式和以上sources类似 |

### sink
|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| sink | string  |    非必填    |      | 表示该Pipeline的sink，使用方式和以上的sources类似 |


如果你希望sink和interceptor可以在不同的ClusterLogConfig/LogConfig间复用，则可以使用以下ref的方式：

### sinkRef

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| sinkRef | string  |    非必填    |      | 表示该Pipeline引用的Sink CR |


### interceptorRef

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| interceptorRef | string  |    非必填    |      | 表示该Pipeline引用的Interceptor CR |
