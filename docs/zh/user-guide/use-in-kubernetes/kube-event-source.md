# 采集Kubernetes Events

除了使用Logconfig采集日志外，Loggie同样可以通过CRD配置任意的source/sink/interceptor，本质上Loggie就是一个支持多Pipeline的数据流，集成了通用的诸如队列重试、数据处理、配置下发、监控报警等等功能，减少了类似需求的研发成本。采集Kubernetes的Events就是其中一个很好的例子。  

Kubernetes Events是由Kubernetes本身组件和一些控制器产生的事件，我们常用的`kubectl describe`命令就可以查看关联资源的事件信息，采集记录这些事件可以帮助我们回溯、排查、审计、总结问题，更好的了解Kubernetes集群内部状态。  

## 准备

和Loggie中转机类似，我们可以单独[部署Aggregator](../../getting-started/install/kubernetes.md#loggie-aggregator)集群或者复用现有的中转机集群。  

## 配置示例

配置kubeEvents source，并且使用`type: cluster`下发配置到Aggregator集群即可。  

!!! config
    ```yaml
    apiVersion: loggie.io/v1beta1
    kind: ClusterLogConfig
    metadata:
      name: kubeevent
    spec:
      selector:
        type: cluster
        cluster: aggregator
      pipeline:
        sources: |
          - type: kubeEvent
            name: event
        sinkRef: dev
    ```

默认情况下，不管是发送给Elasticsearch还是其他的sink，输出的是类似如下格式的数据:

!!! example "event"
    ```json
    {
    "body": "{\"metadata\":{\"name\":\"loggie-aggregator.16c277f8fc4ff0d0\",\"namespace\":\"loggie-aggregator\",\"uid\":\"084cea27-cd4a-4ce4-97ef-12e70f37880e\",\"resourceVersion\":\"2975193\",\"creationTimestamp\":\"2021-12-20T12:58:45Z\",\"managedFields\":[{\"manager\":\"kube-controller-manager\",\"operation\":\"Update\",\"apiVersion\":\"v1\",\"time\":\"2021-12-20T12:58:45Z\",\"fieldsType\":\"FieldsV1\",\"fieldsV1\":{\"f:count\":{},\"f:firstTimestamp\":{},\"f:involvedObject\":{\"f:apiVersion\":{},\"f:kind\":{},\"f:name\":{},\"f:namespace\":{},\"f:resourceVersion\":{},\"f:uid\":{}},\"f:lastTimestamp\":{},\"f:message\":{},\"f:reason\":{},\"f:source\":{\"f:component\":{}},\"f:type\":{}}}]},\"involvedObject\":{\"kind\":\"DaemonSet\",\"namespace\":\"loggie-aggregator\",\"name\":\"loggie-aggregator\",\"uid\":\"7cdf4792-815d-4eba-8a81-d60131ad1fc4\",\"apiVersion\":\"apps/v1\",\"resourceVersion\":\"2975170\"},\"reason\":\"SuccessfulCreate\",\"message\":\"Created pod: loggie-aggregator-pbkjk\",\"source\":{\"component\":\"daemonset-controller\"},\"firstTimestamp\":\"2021-12-20T12:58:45Z\",\"lastTimestamp\":\"2021-12-20T12:58:45Z\",\"count\":1,\"type\":\"Normal\",\"eventTime\":null,\"reportingComponent\":\"\",\"reportingInstance\":\"\"}",
    "systemPipelineName": "default/kubeevent/",
    "systemSourceName": "event"
    }
    ```

为了方便分析展示，我们可以添加一些interceptor将采集到的events数据json decode。  

配置示例如下，具体请参考[日志切分与处理](../../user-guide/best-practice/log-process.md)。

!!! config

    === "interceptor"
        ```yaml
        apiVersion: loggie.io/v1beta1
        kind: Interceptor
        metadata:
          name: jsondecode
        spec:
          interceptors: |
            - type: normalize
              name: json
              processors:
              - jsonDecode: ~
              - drop:
                  targets: ["body"]
        ```

    === "clusterLogConfig"
        ```yaml
        apiVersion: loggie.io/v1beta1
        kind: ClusterLogConfig
        metadata:
          name: kubeevent
        spec:
          selector:
            type: cluster
            cluster: aggregator
          pipeline:
            sources: |
              - type: kubeEvent
                name: event
            interceptorRef: jsondecode
            sinkRef: dev
        ```

经过normalize interceptor里jsonDecode后的数据如下所示：

!!! example "event"

    ```json
    {
    "metadata": {
        "name": "loggie-aggregator.16c277f8fc4ff0d0",
        "namespace": "loggie-aggregator",
        "uid": "084cea27-cd4a-4ce4-97ef-12e70f37880e",
        "resourceVersion": "2975193",
        "creationTimestamp": "2021-12-20T12:58:45Z",
        "managedFields": [
            {
                "fieldsType": "FieldsV1",
                "fieldsV1": {
                    "f:type": {

                    },
                    "f:count": {

                    },
                    "f:firstTimestamp": {

                    },
                    "f:involvedObject": {
                        "f:apiVersion": {

                        },
                        "f:kind": {

                        },
                        "f:name": {

                        },
                        "f:namespace": {

                        },
                        "f:resourceVersion": {

                        },
                        "f:uid": {

                        }
                    },
                    "f:lastTimestamp": {

                    },
                    "f:message": {

                    },
                    "f:reason": {

                    },
                    "f:source": {
                        "f:component": {

                        }
                    }
                },
                "manager": "kube-controller-manager",
                "operation": "Update",
                "apiVersion": "v1",
                "time": "2021-12-20T12:58:45Z"
            }
        ]
    },
    "reportingComponent": "",
    "type": "Normal",
    "message": "Created pod: loggie-aggregator-pbkjk",
    "reason": "SuccessfulCreate",
    "reportingInstance": "",
    "source": {
        "component": "daemonset-controller"
    },
    "count": 1,
    "lastTimestamp": "2021-12-20T12:58:45Z",
    "firstTimestamp": "2021-12-20T12:58:45Z",
    "eventTime": null,
    "involvedObject": {
        "kind": "DaemonSet",
        "namespace": "loggie-aggregator",
        "name": "loggie-aggregator",
        "uid": "7cdf4792-815d-4eba-8a81-d60131ad1fc4",
        "apiVersion": "apps/v1",
        "resourceVersion": "2975170"
    },
    }
    ```

如果觉得数据字段太多或者格式不符合需求，还可以配置normalize interceptor进行修改。  

