# alertWebhook

alertWebhook sink将日志数据发送至http接收方。
使用示例请参考[日志报警](../../../user-guide/monitor/service-log-alarm.md)

!!! example

    ```yaml
    sink:
      type: alertWebhook
      addr: http://localhost:8080/loggie
      headers:
        api: test1
      lineLimit: 10
      template: |
            ******
    ```

## webhook

| `字段`      | `类型`          | `是否必填` | `默认值` | `含义`                                          |
|-----------|---------------|--------|-------|-----------------------------------------------|
| addr      | string        | 非必填    |       | 发送alert的http地址，若为空，则不会发送                      |
| template  | string        | 非必填    |       | 用来渲染的模板                                       |
| timeout   | time.Duration | 非必填    | 30s   | 发送alert的http timeout                          |
| headers   | map           | 非必填    |       | 发送alert的http header                           |
| method    | string        | 非必填    | POST  | 发送alert的http method, 如果不填put(不区分大小写)，都认为是POST |
| lineLimit | int           | 非必填    | 10    | 多行日志采集情况下，每个alert中包含的最大日志行数                   |
