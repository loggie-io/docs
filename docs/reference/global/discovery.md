
# Discovery
服务发现和配置下发相关的配置。目前主要为Kubernetes相关全局配置。

!!! example

    ```yaml
    discovery:
      enabled: true
      kubernetes:
        # Choose: docker or containerd
        containerRuntime: containerd
        # Collect log files inside the container from the root filesystem of the container, no need to mount the volume
        rootFsCollectionEnabled: false
        # Automatically parse and convert the wrapped container standard output format into the original log content
        parseStdout: false
        # If set to true, it means that the pipeline configuration generated does not contain specific Pod paths and meta information,
	      # and these data will be dynamically obtained by the file source, thereby reducing the number of configuration changes and reloads.
        dynamicContainerLog: false
        # Automatically add fields when selector.type is pod in logconfig/clusterlogconfig
        typePodFields:
          logconfig: "${_k8s.logconfig}"
          namespace: "${_k8s.pod.namespace}"
          nodename: "${_k8s.node.name}"
          podname: "${_k8s.pod.name}"
          containername: "${_k8s.pod.container.name}"
        typeNodeFields:
          nodename: "${_k8s.node.name}"
          clusterlogconfig: "${_k8s.clusterlogconfig}"
          os: "${_k8s.node.nodeInfo.osImage}"

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
| rootFsCollectionEnabled | bool  |    非必填    |  false   | 是否开启采集root filesystem里的日志，用于不挂载日志volume的场景 |
| parseStdout | bool  |    非必填    |  false   | 是否开启自动提取容器标准输出原始内容 |
| dynamicContainerLog | bool  |    非必填    |  false   | 是否开启动态容器日志配置，打开后配置文件不会渲染具体的path和动态fields字段，可以有效避免大规模容器化场景里Pod变动从而导致配置的频繁渲染，显著减少reload次数，特别是在单节点的Pod个数较多和使用clusterlogconfig匹配大量的Pod时。一般建议设置为true。 |
| kubeletRootDir | string  |    非必填    |  /var/lib/kubelet   | kubelet的root路径 |
| podLogDirPrefix | string  |    非必填    |  /var/log/pods   | kubernetes默认放置的pod标准输出路径 |
| typePodFields | map  |    非必填    |    | 当logconfig/clusterlogconfig里selector为`type: pod`时，自动添加的kubernetes相关元信息，key即为添加的元信息key，value请使用${_k8s.XX}的方式指定，同时支持填写固定值的`key:value`字段 |
| typeNodeFields | map  |    非必填    |    | 当logconfig/clusterlogconfig里selector为`type: node`时，自动添加的kubernetes相关元信息，key即为添加的元信息key，value请使用${_k8s.XX}的方式指定，同时支持填写固定值的`key:value`字段 |

### typePodFields支持的变量
"${_k8s.XX}"的方式可填写以下参数：

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| ${_k8s.logconfig}| string  |    非必填    |    | 添加logConfig name作为元信息 |
| ${_k8s.node.name} | string  |    非必填    |    | 添加所在节点node name作为元信息 |
| ${_k8s.node.ip} | string  |    非必填    |    | 添加所在节点node ip作为元信息 |
| ${_k8s.pod.namespace} | string  |    非必填    |    | 添加namespace作为元信息 |
| ${_k8s.pod.name} | string  |    非必填    |    | 添加pod name作为元信息 |
| ${_k8s.pod.ip} | string  |    非必填    |    | 添加pod ip作为元信息 |
| ${_k8s.pod.uid} | string  |    非必填    |    | 添加pod uid作为元信息 |
| ${_k8s.pod.container.name} | string  |    非必填    |    | 添加container name作为元信息 |
| ${_k8s.pod.container.id} | string  |    非必填    |    | 添加container id作为元信息 |
| ${_k8s.pod.container.image} | string  |    非必填    |    | 添加container image作为元信息 |
| ${_k8s.workload.kind} | string  |    非必填    |    | 添加pod归属的Deployment/Statefulset/DaemonSet/Job作为元信息 |
| ${_k8s.workload.name} | string  |    非必填    |    | 添加pod归属的Deployment/Statefulset/DaemonSet/Job名称作为元信息 |

### typeNodeFields支持的变量
"${_k8s.XX}"的方式可填写以下参数：

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| ${_k8s.clusterlogconfig}| string  |    非必填    |    | 添加clusterlogconfig name作为元信息 |
| ${_k8s.node.name} | string  |    非必填    |    | 添加所在节点node name作为元信息 |
| ${_k8s.node.addresses.InternalIP}| string  |    非必填    |    | 添加node InternalIP 作为元信息|
| ${_k8s.node.addresses.Hostname}| string  |    非必填    |    | 添加node Hostname作为元信息 |
| ${_k8s.node.nodeInfo.kernelVersion}| string  |    非必填    |    | 添加node kernelVersion作为元信息 |
| ${_k8s.node.nodeInfo.osImage}| string  |    非必填    |    | 添加node osImage作为元信息 |
| ${_k8s.node.nodeInfo.containerRuntimeVersion}| string  |    非必填    |    | 添加node containerRuntimeVersion作为元信息 |
| ${_k8s.node.nodeInfo.kubeletVersion}| string  |    非必填    |    | 添加node kubeletVersion作为元信息 |
| ${_k8s.node.nodeInfo.kubeProxyVersion}| string  |    非必填    |    | 添加node kubeProxyVersion作为元信息 |
| ${_k8s.node.nodeInfo.operatingSystem}| string  |    非必填    |    | 添加node operatingSystem作为元信息 |
| ${_k8s.node.nodeInfo.architecture}| string  |    非必填    |    | 添加node architecture作为元信息 |
| ${_k8s.node.labels.<key>}| string  |    非必填    |    | 添加node的某个label作为元信息，其中的`<key>`请替换成具体的label key |
| ${_k8s.node.annotations.<key>}| string  |    非必填    |    | 添加node的某个annotation作为元信息，其中的`<key>`请替换成具体的annotation key |
