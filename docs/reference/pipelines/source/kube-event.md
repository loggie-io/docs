# kubeEvent

接收Kubernetes events的source。

使用方式可参考[采集Kubernetes Events](../../user-guide/../../user-guide/use-in-kubernetes/kube-event-source.md)。

!!! example
    ```yaml
    sources:
    - type: kubeEvent
      name: event
    ```

## kubeconfig

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| kubeconfig | string  |    非必填    |     | 请求Kubernetes的kubeconfig文件，当Loggie部署在Kubernetes集群中时，无需填写，会进入`in cluster`模式 |


## master

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| master | string  |    非必填    |      | 请求Kubernetes的master地址，当Loggie部署在Kubernetes集群中时，无需填写 |


## bufferSize

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| bufferSize | int  |    非必填    |    1000  | 监听的队列大小，最小为1 |

## watchLatestEvents

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| watchLatestEvents | bool  |    非必填    |    false  | 是否只监听最新的events |

由于Loggie重启后会重新list所有的events，会导致重复发送，如果不希望重复发送，可以设置为true，当然可能导致重启时间段内新产生的events丢失。

## blackListNamespaces

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| blackListNamespaces | string数组  |    非必填    |      | 不接收其中定义的namespaces中产生的events |