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