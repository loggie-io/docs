# webhook

webhook sink将日志数据发送至http接收方，在配置webhook时，logAlert listener将会仅将匹配的日志发送至sink，其余日志会被丢弃。

!!! example

    ```yaml
    sink:
      type: webhook
      addr: http://localhost:8080/loggie
      headers:
        api: test1
      lineLimit: 10
      template: |
            ******
    ```

## webhook

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| addr | string  |    必填    |      | 发送alert的http地址 |
| template | string  |    非必填    |      | 用来渲染的模板 |
| timeout | int  |    非必填    |    30  | 发送alert的http timeout |
| headers | map  |    非必填    |      | 发送alert的http header |
| lineLimit | int  |    非必填    |    10  | 多行日志采集情况下，每个alert中包含的最大日志行数 |
