# Sink

表示一个sink配置。用于在LogConfig/ClusterLogConfig中被引用。

!!! example

    ```yaml
    apiVersion: loggie.io/v1beta1
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

