# Interceptor

表示一个interceptor组。用于在LogConfig/ClusterLogConfig中被引用。

!!! example

    ```yaml
    apiVersion: loggie.io/v1beta1
    kind: Interceptor
    metadata:
      name: default
    spec:
      interceptors: |
        - type: rateLimit
          qps: 90000

    ```

## spec.interceptors

使用"|"符号表示一整段interceptors配置列表，和Pipelines里的配置一致。  