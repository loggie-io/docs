# 日志采集快速排障指南

!!! question  "为什么我的日志没有采集？"
    日志采集中，最为关键核心的问题是，日志有没有被采集到，为什么我配置的日志没有发送过去？  
    下面提供了核心的排查思路和手段供参考。  
    另外，最重要的是，**在环境里配置Loggie的Prometheus监控和Grafana图表**，可以快速发现问题。


## 日志采集核心机制

了解实现机制是排障的基础：

1. 下发采集任务：创建日志采集任务LogConfig CR至Kubernetes
2. 接收日志配置：节点的Agent Loggie监听到K8s相应事件，将LogConfig转换成Pipelines配置文件
3. 采集日志文件：Loggie会自动Reload然后读取配置文件，然后根据配置发送相应的日志数据到下游服务

（针对非Kubernetes的主机场景，只是少了LogConfig CRD配置下发的步骤，其余类似）

## 排查步骤

排查问题关键先要确定是哪一步出现了问题。

### 日志采集任务排查
查看我们要排查的日志采集任务LogConfig/ClusterLogConfig的Events事件：

```bash
kubectl -n ${namespace} describe lgc ${name}
```

如果没有events，则可能为：

- :question: Pod Label未匹配：  
  logConfig中`labelSelector`指定的label未和我们期望的Pod匹配。通过以下命令查看
  ```bash
  kubectl -n ${namespace} get po -owide -l ${labels}
  ```
  比如`kubectl -n ns1 get po -owide -l app=tomcat,service=web`
  来判断一下是否有匹配的Pod。

如果没有类似sync success的events，可根据events同时结合Loggie日志排查问题：

- :question: 配置问题或者Loggie异常：  
通过下面命令查看
```bash
kubectl -n ${loggie-namespace} logs -f ${loggie-pod-name} —-tail=${N}
```
比如`kubectl -nloggie logs -f loggie-5x6vf --tail=100`。
查看对应节点的Loggie日志，根据日志情况进行处理。  

常见的异常有：

- 找不到日志路径：填写的path没有使用volume挂载，可以仔细检查一下path和volumeMount的路径，path是否包含在volumeMount内部。  
- 日志path未匹配到具体的日志文件：path需要填写glob表达式，例如`/var/log/*.log`。最快速的做法是，在需要采集的业务的Pod里，执行`ls <path>`，因为ls也是使用的glob表达式去匹配日志文件。另外也需额外注意是否配置了`ignoreOlder`/`excludeFiles`等参数，忽略或者排除了我们希望采集的日志文件。

### 节点日志Agent排查

#### 1. :mag_right: 找到logConfig匹配的pod所在节点的日志Agent

根据在logConfig的labelSelector找到一个匹配的业务pod：
```bash
kubectl -n ${namespace} get po -owide -l ${labels}
```
找到任意一个所在的Node节点`${node-name}`，然后通过：
```bash
kubectl -n ${loggie-namespace} get po -owide |grep ${node-name}
```
找到该节点所在的Loggie。

#### 2. :mag_right: 查看对应节点的Loggie日志

检查是否有异常，如果有异常，则需根据异常日志再做分析判断
```bash
kubectl -n ${loggie-namespace} logs -f ${loggie-pod-name} —-tail=${N}
```

#### 3. :mag_right: 查看节点Loggie渲染生成的配置

进入到容器中：
```bash
kubectl -n ${loggie-namespace} exec -it ${loggie-pod-name} bash
```
查看渲染生成的Pipeline配置：
```bash
ls /opt/loggie/pipeline/
```
当然，你也可以选择通过调用Loggie接口来查看渲染生成的Pipeline配置：
```
curl ${loggie-pod-ip}:9196/api/v1/reload/config
```

#### 4. :mag_right: 查看日志采集持久化状态

Loggie会记录每个日志文件的采集状态，这样即使Loggie重启后，也可以继续保持上一次的采集进度，避免重新采集日志文件。
可以通过调用接口来查看：
```
curl ${loggie-pod-ip}:9196/api/v1/source/file/registry?format=text | grep XXX
```
一般返回的类似：
```
  {
    "id": 85,
    "pipelineName": "default/tomcat",
    "sourceName": "tomcat-7d64c4f6c9-cm8jm/tomcat/common",
    "filename": "/var/lib/kubelet/pods/9397b8be-8927-44ba-8b94-73e5a4459377/volumes/kubernetes.io~empty-dir/log/catalina.2022-06-02.log",
    "jobUid": "3670030-65025",
    "offset": 4960,
    "collectTime": "2022-06-06 12:44:12.861",
    "version": "0.0.1"
  },
```
其中的`filename`为Loggie转换后的实际节点上的日志路径，`jobUid`组成为文件的`inode-deviceId`，`offset`为sink发送成功接收到ack后的offset。  
另外，我们可以通过在Loggie容器中执行：
```
stat ${filename}
```
查看文件的size/inode等信息，例如：  
!!! example  "stat"
    ```yaml
      File: /var/lib/kubelet/pods/9397b8be-8927-44ba-8b94-73e5a4459377/volumes/kubernetes.io~empty-dir/log/catalina.2022-06-02.log
      Size: 4960      	Blocks: 16         IO Block: 4096   regular file
    Device: fe01h/65025d	Inode: 3670030     Links: 1
    Access: (0640/-rw-r-----)  Uid: (    0/    root)   Gid: (    0/    root)
    Access: 2022-06-06 12:44:12.859236003 +0000
    Modify: 2022-06-02 08:54:33.177240007 +0000
    Change: 2022-06-02 08:54:33.177240007 +0000
    ```
可通过比较size和offset来判断采集进度。如果size=offset，则说明已经文件已经全部采集并发送成功。
