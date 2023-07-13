# 编译构建

## 持久化引擎选择

Loggie在v1.5及之后版本，提供了除sqlite外的badger引擎。

sqlite和badger两种持久化引擎配置，请参考[这里](../reference/global/db.md)。

为什么引入badger引擎？使用sqlite和CGO的问题有：

- sqlite对环境的c lib库版本有需求，主机部署场景下有很多低版本的机器可能无法运行
- 避免CGO引入对构建的一些坑，比如构建多架构（amd64/arm64）镜像


## 容器镜像构建
默认情况下，Loggie的会给所有的release分支和main分支构建镜像，并推送到[dockerhub](https://hub.docker.com/r/loggieio/loggie/)上。但是从dockerhub上拉镜像可能存在被限流等问题，所以推荐大家自行构建镜像，或者重新推送到自己的镜像仓库中。

Loggie项目下提供了两个Dockerfile:

- Dockerfile：默认的dockerfile，使用了sqlite，所以也会使用CGO构建
- Dockerfile.badger：可在docker build的时候使用`-f`参数指定使用该dockerfile，其中使用了badger作为持久化引擎，存粹的Golang，无CGO。

除了使用docker build外，还可使用make来构建：
```bash
make docker-build REPO=<YourRepoHost>/loggie TAG=v1.x.x
make docker-push REPO=<YourRepoHost>/loggie TAG=v1.x.x
```
或者直接构建和推送多架构镜像：
```bash
make docker-multi-arch REPO=<YourRepoHost>/loggie TAG=v1.x.x
```

注：

- 这里的TAG可不传，默认会根据Git中的tag或者分支来自动生成。

**多架构**

Loggie项目中的Dockerfile均支持多架构的构建。dockerhub上的Loggie镜像为amd64和arm64的多架构，拉取时会自动识别本地架构。  

不过如果你希望修改tag并重新推到自己的仓库，请勿直接docker pull & docker tag & docker push，这会只push你本地架构的镜像，导致仓库中的镜像多架构失效。  
如果希望推送多架构镜像到自己的仓库，可使用其他的一些开源工具，比如[regctl](https://github.com/regclient/regclient/blob/main/docs/regctl.md)，可在本地使用类似`regctl image copy loggieio/loggie:xxx <YourRepoImage>`的命令来推送。

如果接入自己的构建流程或者工具，多架构镜像请使用`make docker-multi-arch`或者`docker buildx`构建。

## 二进制构建
二进制可用于在主机部署的场景。

- sqlite:
  ```bash
  make build
  ```
- badger:
  ```bash
  make build-in-badger
  ```

如果需要交叉编译，请加上GOOS和GOARCH。

如果本地使用`make build`构建失败，可修改makefile中的extra_flags额外构建参数，请尝试将其中的`-extldflags "-static"`去除。