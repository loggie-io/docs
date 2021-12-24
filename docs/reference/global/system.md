启动参数中使用`-config.system`指定全局系统YAML格式的配置，包含如下：

- [**monitor**](monitor.md): 监控相关的配置
- [**discovery**](discovery.md): 服务发现与配置下发
- [**reload**](reload.md): 动态配置热加载
- [**defaults**](defaults.md): 全局默认配置
- [**http**](http.md): 管理和监控使用的http端口


???+ example "全局配置示例"

    ``` yaml
    # loggie.yml
    loggie:
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

      reload:
        enabled: true
        period: 10s

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