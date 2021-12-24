# sink

表示一个sink配置。

!!! example

    ```yaml
    kind: Sink
    metadata:
      name: default
    spec:
      sink: |
        type: elasticsearch
        index: "loggie"
        hosts: ["elasticsearch-master.default.svc:9200"]

    ```

## spec.sink

使用"|"符号表示一个sink配置，和Pipelines里的配置一致。  

