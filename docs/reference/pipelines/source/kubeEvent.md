# kubeEvent

接收Kubernetes events的source

## 配置

### kubeconfig
|  `类型`  |  `是否必填`  |  `默认值`  |  `含义`  |
| ------- | ----------- | --------- | ------- |
| string  |    非必填    |    ~/.kube/config  | 请求Kubernetes的kubeconfig文件 |

### master

|  `类型`  |  `是否必填`  |  `默认值`  |  `含义`  |
| ------- | ----------- | --------- | ------- |
| string  |    非必填    |    ""  | 请求Kubernetes的master地址 |

### bufferSize

|  `类型`  |  `是否必填`  |  `默认值`  |  `含义`  |
| ------- | ----------- | --------- | ------- |
| int  |    非必填    |    1000  | 监听的队列大小，最小为1 |




