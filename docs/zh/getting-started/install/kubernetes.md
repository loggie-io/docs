# Kubernetes部署

## 部署Loggie DaemonSet

请确保本地有kubectl和helm可执行命令。

- kubectl（[下载](https://github.com/kubernetes/kubernetes/tree/master/CHANGELOG)）
- helm（[下载](https://helm.sh/docs/intro/install/)）

### 下载helm-chart包

```bash
helm pull https://github.com/loggie-io/installation/releases/download/v1.2.0/loggie-v1.2.0.tgz && tar xvzf loggie-v1.2.0.tgz
```

### 修改配置
进入chart目录：
```bash
cd installation/helm-chart
```

查看values.yml，并根据实际情况进行修改。  

目前可配置的参数如下所示：

#### 镜像
```yaml
image: hub.c.163.com/loggie/loggie:v1.2.0
```
loggie的镜像。

#### 资源
```yaml
resources:
  limits:
    cpu: 2
    memory: 2Gi
  requests:
    cpu: 100m
    memory: 100Mi
```
Loggie Agent的limit/request资源，可以根据实际情况修改。  

#### 额外启动参数
```yaml
extraArgs: {}
```
Loggie的额外启动参数，比如希望修改配置使用debug的日志级别，不使用json格式的日志打印形式，可以修改为:
```yaml
extraArgs:
  log.level: debug
  log.jsonFormat: false
```

#### 额外挂载
```yaml
extraVolumeMounts:
  - mountPath: /var/log/pods
    name: podlogs
  - mountPath: /var/lib/kubelet/pods
    name: kubelet
  - mountPath: /var/lib/docker
    name: docker


extraVolumes:
  - hostPath:
      path: /var/log/pods
      type: DirectoryOrCreate
    name: podlogs
  - hostPath:
      path: /var/lib/kubelet/pods
      type: DirectoryOrCreate
    name: kubelet
  - hostPath:
      path: /var/lib/docker
      type: DirectoryOrCreate
    name: docker
```
建议根据实际情况默认挂载以上目录。  

- Loggie需要记录采集的文件状态(offset等)，避免重启后从头开始采集文件，造成日志采集重复，默认挂载路径为/data/loggie.db，所以挂载了`/data/loggie--{{ template "loggie.name" . }}`目录。
- 如果业务Pod使用hostPath挂载日志到节点，需要Loggie挂载相同的路径。  
- 因为Loggie需要采集容器标准输出的路径，所以需要从节点/var/lib/docker或者/var/log/pods下采集日志，如果环境中部署的docker修改了该路径，请同步修改该挂载路径。如果非docker运行时，比如使用containerd，无需挂载/var/lib/docker，Loggie会从/var/log/pods中寻找实际的标准输出路径。  
- 如果业务Pod使用emtpyDir挂载日志到节点，默认emtpyDir会在节点的/var/lib/kubelet/pods路径下，所以需要Loggie挂载该路径。如果环境的kubelet修改了该路径配置，这里需要同步修改。  


#### 调度
```yaml
nodeSelector: {}

affinity: {}
# podAntiAffinity:
#   requiredDuringSchedulingIgnoredDuringExecution:
#   - labelSelector:
#       matchExpressions:
#       - key: app
#         operator: In
#         values:
#         - loggie
#     topologyKey: "kubernetes.io/hostname"
```
可使用nodeSelector和affinity来控制Loggie Pod的调度，具体请参考Kubernetes文档。  

```yaml
tolerations: []
# - effect: NoExecute
#   operator: Exists
# - effect: NoSchedule
#   operator: Exists
```
如果节点有自己的taints，会导致Loggie Pod无法调度到该节点，如果需要忽略taints，可以加上对应的tolerations。

#### 更新策略
```yaml
updateStrategy:
  type: RollingUpdate
```
可为`RollingUpdate`或者`OnDelete`。  

#### 全局配置
```yaml
config:
  loggie:
    reload:
      enabled: true
      period: 10s
    monitor:
      logger:
        period: 30s
        enabled: true
      listeners:
        filesource: ~
        filewatcher: ~
        reload: ~
        sink: ~
    discovery:
      enabled: true
      kubernetes:
        containerRuntime: containerd
        fields:
          container.name: containername
          logConfig: logconfig
          namespace: namespace
          node.name: nodename
          pod.name: podname
    http:
      enabled: true
      port: 9196
```
具体参数说明可参考[组件配置](../../reference/index.md)。
需要注意的是，如果你在本地使用Kind等工具部署Kubernetes，Kind默认会使用containerd runtime，此时需要在discovery.kubernetes中增加        `containerRuntime: containerd`，指定容器运行时。  


#### service
如果Loggie希望接收其他服务发送的数据，需要将自身的服务通过service暴露出来。  

正常情况下，使用Agent模式的Loggie只需要暴露自身管理端口。  

```yaml
servicePorts:
  - name: monitor
    port: 9196
    targetPort: 9196
```

### 部署

初次部署，我们指定部署在`loggie` namespace下，并让helm自动创建该namespace。
```bash
helm install loggie ./ -nloggie --create-namespace
```

如果你的环境中已经创建了`loggie` namespace，可以忽略其中的`-nloggie`和`--create-namespace`参数。当然，你也可以使用自己的namespace，将其中`loggie`替换即可。  

!!! caution "Kubernetes版本问题"
    ```
    failed to install CRD crds/crds.yaml: unable to recognize "": no matches for kind "CustomResourceDefinition" in version "apiextensions.k8s.io/v1"
    ```
    如果你在helm install的时候出现类似的问题，说明你的Kubernetes版本较低，不支持apiextensions.k8s.io/v1版本CRD。Loggie暂时保留了v1beta1版本的CRD，请删除charts中v1beta1版本，`rm loggie/crds/crds.yaml`，重新install。


### 查看部署状态
执行完后，通过使用helm命令来查看部署状态：
```
helm list -nloggie
```
类似如下所示：
```
NAME  	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART        	APP VERSION
loggie	loggie   	1       	2021-11-30 18:06:16.976334232 +0800 CST	deployed	loggie-v0.1.0	v0.1.0
```

同时也可以通过kubectl命令查看Pod是否已经被创建。  
```
kubectl -nloggie get po
```
类似如下所示：
```
loggie-sxxwh   1/1     Running   0          5m21s   10.244.0.5   kind-control-plane   <none>           <none>
```



## 部署Loggie Aggregator

部署Aggregator基本和Agent一致，在helm chart中我们提供了默认注释掉的`Aggregator global config`部分，只需使用该配置同时去掉Agent的配置即可。  

同时，请注意在values.yaml中根据情况增加：

- nodeSelector或者affinity，根据node是否有污点增加tolerations。使得Aggregator DaemonSet只调度在某几个节点上
- service增加接收的端口，比如使用Grpc source，需要填写默认的6066端口：
  ```yaml
  servicePorts:
  - name: grpc
    port: 6066
    targetPort: 6066
  ```
- discovery.kubernetes中增加cluster字段，表示中转机集群名称，用于区别Agent或者其他的Loggie集群，如下所示：
  ```yaml
  config:
    loggie:
      discovery:
        enabled: true
        kubernetes:
          cluster: aggregator
  ```

执行部署命令参考：
```
helm install loggie-aggregator ./ -nloggie-aggregator --create-namespace
```

!!! note
    Loggie中转机同样可以使用Deployment或者StatefulSet来部署，请参考DaemonSet自行修改helm chart。
