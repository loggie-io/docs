# Defaults
defaults用于设置Pipelines配置中的默认值。当Pipeline中没有设置值时生效，或者用于覆盖默认的参数。

!!! example

    ```yaml
    defaults:
      sources:
        - type: file
          watcher:
            cleanFiles:
              maxHistory: 10
      sink:
        type: dev
        printEvents: true
    ```

## sources
和Pipeline中的source一致。当Pipelines配置了相同type的source时，会覆盖其中未填写字段的默认值。

比如:
```yaml
    sources:
      - type: file
        watcher:
          cleanFiles:
            maxHistory: 10
```
如果Pipeline配置了file source，此时可以设置全局的文件清理保留天数为10天，而不需要在每个Pipeline的file source中都设置一遍。

## sink
和Pipeline中的sink一致，如果集群只需要设置一个全局的sink输出源，则只需要在这里配置一次，避免在每个Pipeline中填写。

## interceptors
Loggie会默认设置`metric`、`maxbytes`、`retry`3个系统内置interceptors。  
如果需要添加其他的默认interceptors，会覆盖掉以上的内置interceptors，所以强烈建议此时将内置interceptors加上，除非你确认不需要以上系统内置的interceptors。  

## queue
默认为channel queue。
