
# Discovery

!!! example

    ```yaml
    discovery:
      enabled: true
      kubernetes:
        cluster: aggregator
        fields:
          container.name: containername
          logConfig: logconfig
          namespace: namespace
          node.name: nodename
          pod.name: podname

    ```

## enabled

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| enabled | bool  |    非必填    |   false   | 是否开启服务发现配置下发模块 |


## Kubernetes

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| cluster | string  |    非必填    |  ""    | 标识Loggie集群名称。Loggie支持在一个Kubernetes集群中部署多套Loggie，可以通过在LogConfig CRD中指定`selector.cluster`，指定配置下发的Loggie集群 |
| kubeconfig | string  |    非必填    |     |  指定请求Kubernetes集群API的kubeconfig文件。通常在Loggie部署到Kubernetes集群中无需填写，此时为inCluster模式。如果Loggie部署在Kubernetes集群外（例如本地调试时），需要指定该kubeconfig文件。 |
| master | string  |    非必填    |     | 指定请求Kubernetes集群API的master地址，inCluster模式一般无需填写 |
| containerRuntime | string  |    非必填    |  docker   | 容器运行时，可选`docker`和`containerd` |
| dockerDataRoot | string  |    非必填    |  /var/lib/docker   | docker的rootfs路径 |
| kubeletRootDir | string  |    非必填    |  /var/lib/kubelet   | kubelet的root路径 |
| podLogDirPrefix | string  |    非必填    |  /var/log/pods   | kubernetes默认放置的pod标准输出路径 |
| fields | map  |    非必填    |    | 自动添加的元信息 |
| fields.node.name | string  |    非必填    |  node.name  | 添加所在节点node name作为元信息，同时使用该值为key |
| fields.node.ip | string  |    非必填    |  node.ip  | 添加所在节点node ip作为元信息，同时使用该值为key |
| fields.namespace | string  |    非必填    |  namespace  | 添加namespace作为元信息，同时使用该值为key |
| fields.pod.name | string  |    非必填    |  pod.name  | 添加pod name作为元信息，同时使用该值为key |
| fields.pod.ip | string  |    非必填    |  pod.ip  | 添加pod ip作为元信息，同时使用该值为key |
| fields.container.name | string  |    非必填    |  container.name  | 添加container name作为元信息，同时使用该值为key |
| fields.logConfig | string  |    非必填    |  logConfig  | 添加logConfig name作为元信息，同时使用该值为key |