
# Monitor
监控事件总线，所有的组件都可以发出自己的metrics指标数据，由listeners消费处理。

具体请参考[这里](../monitor/overview.md)。

!!! example

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