启动参数中使用`- config.system`指定全局YAML格式的配置，包含如下：

## Reload
reload会定时检查启动参数`- config.pipeline`指定的配置文件，如果检测到文件内容发生变动，会重启有变动配置的Pipeline，未被修改的Pipeline不会受影响。

- `enabled`: 表示是否开启reload，默认为false，即不会检测配置并reload
- `period`: reload的时间间隔，默认为10s，不建议设置过短，会增加资源的消耗

示例：  
```yaml
  reload:
    enabled: true
    period: 10s
```


## Monitor Eventbus
监控事件总线，所有的组件都可以发出自己的metrics指标数据，由listeners消费处理。

### `logger` 
控制监控指标metrics输出到Loggie本身的日志中
  - `enabled`: 是否开启，默认为false
  - `period`: 指标打印的时间间隔，默认为10s

### `listeners`  
  - `filesource`: file source实时文件采集的监控指标处理，包括文件的名称、采集进度、文件大小、qps等
  - `filewatcher`: 对文件采集情况的定时检查并暴露指标，包括文件名称、ackOffset、修改时间、大小等
    - `period`: 定时检查时间，默认`5m`
    - `checkUnFinishedTimeout`: 检查文件是否采集完毕的超时时间，默认`24h`。如果检测到文件的最近修改时间为24小时之前，同时文件的并未采集完毕，则会在metrics中被标记为unfinished状态
  - `sink`: sink发送端的监控指标处理，包括发送成功的event数量、发送失败的event数量、event qps等
  - `reload`: reload的指标处理，目前为总的reload次数
  - `logAlert`: 日志报警处理，
    - `alertManagerAddress`: alertManager地址, 字符串数组，必填
    - `bufferSize`: 日志报警发送的buffer大小，默认100
    - `batchTimeout`: 每个报警发送batch的最大发送时间，默认10s
    - `batchSize`: 每个报警发送batch的最大包含报警请求个数，默认10
    !!! info
        为了避免太多小的报警发送请求，报警请求会封装成batch发送至alertManager


示例:
```yaml
  monitor:
    logger:
      period: 30s
      enabled: true
    listeners:
      filesource: ~
      filewatcher: ~
      reload: ~
      sink: ~

```

## Discovery
- `enabled`: 是否开启服务发现配置下发模块

目前支持Kubernetes下的配置自动下发。
### Kubernetes
- `cluster`: 标识Loggie集群名称，默认为""。Loggie支持在一个Kubernetes集群中部署多套Loggie，可以通过在LogConfig CRD中指定`selector.cluster`，指定配置下发的Loggie集群。
- `kubeconfig`: 指定请求Kubernetes集群API的kubeconfig文件。通常在Loggie部署到Kubernetes集群中无需填写，此时为inCluster模式。如果Loggie部署在Kubernetes集群外（例如本地调试时），需要指定该kubeconfig文件。
- `master`: 指定请求Kubernetes集群API的master地址，inCluster模式一般无需填写
- `dockerDataRoot`: docker的rootfs路径，默认为`/var/lib/docker`
- `kubeletRootDir`: kubelet的root路径，默认为`/var/lib/kubelet`
- `fields`: 自动添加的元信息  
    - `node.name`:
    - `namespace`:
    - `pod.name`:
    - `container.name`:
    - `logConfig`:


## Defaults
defaults用于设置Pipelines配置中的默认值。当Pipeline中没有设置值时生效，或者用于覆盖默认的参数。

### sources
和Pipeline中的source一致，比如:
```yaml
    sources:
      - type: file
        watcher:
          cleanFiles:
            maxHistory: 10
```
如果Pipeline配置了file source，此时可以设置全局的文件清理保留天数为10天，而不需要在每个Pipeline的file source中都设置一遍。

### sink
和Pipeline中的sink一致，如果集群只需要设置一个全局的sink输出源，则只需要在这里配置一次，避免在每个Pipeline中填写。

### interceptors
Loggie会默认设置`metric`、`maxbytes`、`retry`3个系统内置interceptors。  
如果需要添加其他的默认interceptors，强烈建议将以上3个interceptors加上，除非你确认不需要以上系统内置的interceptors。  

### queue
默认为channel queue。

## Http
- `enabled`: 是否开启http，默认为false。
- `host`: http监听的host，默认`0.0.0.0`。
- `port`: http监控的端口，默认`9196`。

!!! tips
    一般推荐打开http端口，但是如果Kubernetes或者容器部署，打开了hostNetwork的情况，或者主机部署无网络隔离的情况，请注意端口冲突，以及监听host是否暴露给公网、是否有安全隐患。


## 示例

???+ example "全局配置示例"

    ``` yaml
    # loggie.yml
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
        enabled: false
        kubernetes:
          kubeconfig: ~/.kube/config
    
      configAdapter:
        enabled: false
        configDir: "/tmp/logconf/filebeat/*.yml"
    
      defaults:
        sink:
          type: dev
        sources:
          - type: file
            watcher:
              cleanFiles:
                maxHistory: 1
      http:
        enabled: true
        port: 9196

    ```