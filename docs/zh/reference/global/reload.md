# Reload

reload会定时检查启动参数`-config.pipeline`指定的配置文件，如果检测到文件内容发生变动，会重启有变动配置的Pipeline，未被修改的Pipeline不会受影响。

!!! example

    ```yaml
      reload:
        enabled: true
        period: 10s
    ```

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| enabled | bool  |    非必填    |  false    | 是否开启reload |
| period | time.Duration  |    非必填    |  10s    | reload检测配置的时间间隔，不建议设置过短，否则可能会增加CPU的消耗 |