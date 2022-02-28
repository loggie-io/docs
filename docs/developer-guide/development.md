# 本地开发

本文介绍如何在本地开发和调试Loggie工程。

## 本地工程

#### golang开发环境

Loggie使用Golang编写，请确保本地有[Golang](https://go.dev/dl)开发环境。

```bash
go version
```

#### 代码工程
请使用自己的github账号fork Loggie工程。然后git clone到本地。

```bash
git clone git@github.com:<Account>/loggie.git
```

## 本地环境

正常情况下，本地无特殊依赖。

针对不同的功能场景，需要根据具体情况安装或者使用外部的依赖。

建议本地运行Kubernetes进行部署和管理。

### Kubernetes

**搭建Kubernetes**

推荐使用[Kind](https://kind.sigs.k8s.io/docs/user/quick-start/)。

**部署依赖组件**

推荐使用[Helm](https://helm.sh/docs/intro/install/)来部署。相关的组件可使用公开的Helm仓库来添加和部署。

**本地Loggie连接Kubernetes**

如果需要使用LogConfig等CRD，或者调试Kubernetes下的使用方式，需要在系统配置中，enable Kubernetes Discovery。 

!!! config  "loggie.yml"

    ```yaml
      discovery:
        enabled: true
        kubernetes:
          kubeconfig: ${home}/.kube/config
    ```

指定kubeconfig连接本地的Kubernetes APIServer。  

另外，Agent形态的Loggie，会只监听本节点的Kubernetes Pod事件，需要在Loggie启动参数中添加`-meta.nodeName`来模拟所在的节点。  

例如，可以使用`kubectl get node`查看所有节点名称：

```bash
NAME                 STATUS   ROLES    AGE     VERSION
kind-control-plane   Ready    master   2y50d   v1.21.0
```
然后启动参数增加`-meta.nodeName=kind-control-plane`，可以使Loggie启动后被认为部署在该节点。

## 验证与调试
Loggie默认启动后，输出的日志格式为JSON，如果你不习惯，可在启动参数中添加`-log.jsonFormat=false`来关闭。  

本地尽量使用模拟的方式，合理利用dev source和dev sink。比如：

- 开发source，可配置dev sink输出到本地stdout验证
- 开发sink，可配置dev source模拟输入或者配置file source采集本地文件
- 开发interceptor，可同时配置dev source和dev sink用于模拟

示例：在pipelines中，使用dev sink，查看最终输出发送至下游的数据。

!!! config  "pipelines.yml"

    ```yaml
        sink:
          type: "dev"
          printEvents: true
          codec:
            pretty: true
    ```
