# Kubernetes部署

## 部署Loggie DaemonSet

### 下载helm-chart包

```bash
git clone https://github.com/loggie-io/installation.git
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
image: hub.c.163.com/qingzhou/loggie:latest
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
  - mountPath: /data/
    name: data
  - mountPath: /var/log/pods
    name: podlogs
  - mountPath: /var/lib/kubelet/pods
    name: kubelet
  - mountPath: /var/lib/docker
    name: docker


extraVolumes:
  - hostPath:
      path: /data/
      type: DirectoryOrCreate
    name: data
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

- Loggie需要记录采集的文件状态(offset等)，避免重启后从头开始采集文件，造成日志采集重复，默认挂载路径为/data/loggie.db，所以挂载了/data目录。如果需要修改该目录，请同时修改file source配置`db.file`（默认`./data/loggie.db`）。  
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


### 部署

初次部署，我们指定部署在`loggie` namespace下，并让helm自动创建该namespace。
```bash
helm install loggie ./ -nloggie --create-namespace --set image=hub.c.163.com/qingzhou/loggie:v0.1.0-rc1
```

如果你的环境中已经创建了`loggie` namespace，可以忽略其中的`-nloggie`和`--create-namespace`参数。当然，你也可以使用自己的namespace，将其中`loggie`替换即可。  

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
// TODO