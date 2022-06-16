# 快速上手：节点日志采集

下面我们将演示一个最简单的采集节点日志文件的场景。

### 1. 下载可执行文件
请找一台Linux服务器节点，下载Loggie二进制可执行文件
```shell
curl -LJ https://github.com/loggie-io/installation/releases/download/v1.2.0/loggie-linux-amd64 -o loggie
```
### 2. 添加配置文件

我们先使用dev sink将file source采集的日志文件打印到标准输出，复制以下内容为pipelines.yml文件：
=== "pipelines.yml"
```yaml
cat << EOF > pipelines.yml
pipelines:
  - name: demo
    sources:
      - type: file
        name: mylog
        paths:
          - "/var/log/*.log"
    sink:
      type: dev
      printEvents: true
EOF
```
这里我们创建了一个名称为demo的pipeline，然后定义了一个类型为file的source输入源组件，表示需要采集在`/var/log`目录下满足`*.log`匹配规则的日志文件。文件采集后，文件会被发送至dev sink输出源，该sink仅仅将采集的文件打印到标准输出。

pipeline文件表示我们想要的输入、输出等业务相关的配置，除了pipeline配置文件外，Loggie还需要有一个全局的配置文件。

```yaml
// loggie.yml
cat << EOF > loggie.yml
loggie:
  reload:
    enabled: true
    period: 10s
EOF
```

这里我们只展示了一个比较简单的配置，表示打开loggie的动态配置reload功能，同时间隔检查时间为10s。

在节点上增加以上两个配置文件后，我们就可以开始启动Loggie了。


### 3. 运行
```shell
./loggie -config.system=./loggie.yml -config.pipeline=./pipelines.yml -log.jsonFormat=false
```

启动参数里，填入上面的loggie.yml和pipelines.yml的文件路径。

看到正常的启动日志后，表明Loggie就开始正常的工作了。同时节点/var/log/*.log下的日志文件，都会被打印到标准输出。  
