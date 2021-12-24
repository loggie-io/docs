
# Http

Loggie自身提供的Http端口，包含监控metrics，内部运维等接口。

!!! example

    ```yaml
    http:
      enabled: true
      host: "0.0.0.0"
      port: 9196

    ```

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| enabled | bool  |    非必填    |   false   | 是否开启http |
| host | string  |    非必填    |   0.0.0.0   | http监听的host |
| port | http  |    非必填    |   9196   | http监控的端口 |


!!! tips
    一般推荐打开http端口，但是如果Kubernetes或者容器部署时，使用hostNetwork请注意端口冲突，以及监听host是否暴露给公网、是否有安全隐患。