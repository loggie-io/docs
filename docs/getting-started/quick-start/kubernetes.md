# 快速上手：采集Kubernetes Pod日志

下面将带你演示在一个Kubernetes集群中，通过创建LogConfig CRD快速采集Pod的日志。

## 1. 准备Kubernetes环境

可以使用现有Kubernetes集群，或者[部署Kubernetes](https://kubernetes.io/zh/docs/tasks/tools/)。本地推荐使用[Kind](https://kind.sigs.k8s.io/)搭建Kubernetes集群。  

本文的操作需要在本地使用:

- kubectl（[下载](https://github.com/kubernetes/kubernetes/tree/master/CHANGELOG)）
- helm（[下载](https://helm.sh/docs/intro/install/)）

请确保本地有kubectl和helm可执行命令。

## 2. 部署Loggie DaemonSet

你可以在 [installation](https://github.com/loggie-io/installation/releases) 页面查看所有发布的部署chart。

可以选择：

#### 下载chart再部署

```bash
helm pull https://github.com/loggie-io/installation/releases/download/v1.2.0/loggie-v1.2.0.tgz && tar xvzf loggie-v1.2.0.tgz
```
尝试修改一下其中的values.yaml。


然后部署安装：

```bash
helm install loggie ./loggie -nloggie --create-namespace
```

当然你也可以：
#### 直接部署：

```bash
helm install loggie -nloggie --create-namespace https://github.com/loggie-io/installation/releases/download/v1.2.0/loggie-v1.2.0.tgz
```

!!! node "想使用其他版本镜像？"

    为了方便体验最新的Fix和特性，我们提供了main分支每次合并后的镜像版本，可通过 [这里](https://hub.docker.com/r/loggieio/loggie/tags) 进行选择。  
    同时你可以在helm install命令中增加`--set image=loggieio/loggie:vX.Y.Z`来指定具体的Loggie镜像。


!!! caution "部署有问题？"

    如果尝试部署后出现问题，或者在你的环境中以下演示操作未成功，请参考[Kubernetes下部署Loggie](../install/kubernetes.md)，修改相关配置。


## 3. 采集日志

Loggie定义了Kubernetes CRD LogConfig，一个LogConfig表示采集一类Pods的日志采集任务。

### 3.1 创建被采集的Pods

我们先创建一个Pod用于被采集日志的对象。
```shell
kubectl create deploy nginx --image=nginx
```
接下来将采集这个Nginx Pod的标准输出stdout日志。  

### 3.2 定义输出源Sink

接着，我们创建一个Loggie定义的CRD Sink实例，表明日志发送的后端。  
为了方便演示，这里我们将日志发送至Loggie Agent自身的日志中并打印。  

```yaml
cat << EOF | kubectl apply -f -
apiVersion: loggie.io/v1beta1
kind: Sink
metadata:
  name: default
spec:
  sink: |
    type: dev
    printEvents: true
EOF
```

可以通过`kubectl get sink`查看到已创建的Sink。

### 3.3 定义采集任务

Loggie定义CRD LogConfig，表示一个日志采集任务。我们创建一个LogConfig示例如下所示：  

```yaml
cat << EOF | kubectl apply -f -
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
EOF
```

可以看到，上面使用了sinkRef引用了刚才创建的`sink default CR`。当然，我们还可以直接在Logconfig中使用sink字段，示例如下：

```yaml
cat << EOF | kubectl apply -f -
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
    sink: |
      type: dev
      printEvents: true
      codec:
        type: json
        pretty: true
EOF
```

创建完之后，我们可以使用`kubectl get lgc`查看到创建的CRD实例。

同时，我们还可以通过`kubectl describe lgc nginx`查看LogConfig的事件，以获取最新的状态。

```
Events:
  Type    Reason       Age   From                       Message
  ----    ------       ----  ----                       -------
  Normal  syncSuccess  52s   loggie/kind-control-plane  Sync type pod [nginx-6799fc88d8-5cb67] success
```

上面的nginx LogConfig通过其中的spec.selector来匹配采集哪些Pod的日志，这里我们使用`app: nginx`选择了刚才创建的nginx Pod。  
spec.pipeline则表示Loggie的Pipeline配置，我们只采集容器标准输出的日志，所以在paths中填写`stdout`即可。  

## 4. 查看日志
首先找到所在的nginx pod节点：
```bash
kubectl get po -owide -l app=nginx
```

然后我们找到该节点的Loggie：
```bash
kubectl -nloggie get po -owide |grep ${node}
```
可以通过：
```bash
kubectl -nloggie logs -f ${logge-pod}
```
查看Loggie打印出的日志，里面展示了采集到的nginx标准输出日志。  

## 更多

上文只是一个简单的快速演示，部署出现问题或者想了解更多Kubernetes下Loggie如何使用？

- 更全面的部署介绍：[Kubernetes下部署Loggie](../install/kubernetes.md)
- Kubernetes下日志采集最佳实践：[Kubernetes下的日志采集](../../user-guide/use-in-kubernetes/collect-container-logs.md)
