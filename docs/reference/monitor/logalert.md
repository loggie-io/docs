# logAlert listener

用于日志报警的发送。

!!! example

    ```yaml
    logAlert:
        addr: [ "http://127.0.0.1:8080/loggie" ]
        bufferSize: 100
        batchTimeout: 10s
        batchSize: 1
        lineLimit: 10
        template: |
            *****
    ```

## 配置

| `字段`         | `类型`          | `是否必填` | `默认值` | `含义`                                          |
|--------------|---------------|--------|-------|-----------------------------------------------|
| addr         | string数组      | 必填     |       | 发送alert的http地址                                |
| bufferSize   | int           | 非必填    | 100   | 日志报警发送的buffer大小，单位为报警事件个数                     |
| batchTimeout | time.Duration | 非必填    | 10s   | 每个报警发送batch的最大发送时间                            |
| batchSize    | int           | 非必填    | 10    | 每个报警发送batch的最大包含报警请求个数                        |
| template     | string        | 非必填    |       | 渲染发送的alert结构体的go template模板                   |
| timeout      | time.Duration | 非必填    | 30s   | 发送alert的http timeout                          |
| headers      | map           | 非必填    |       | 发送alert的http header                           |
| method       | string        | 非必填    | POST  | 发送alert的http method, 如果不填put(不区分大小写)，都认为是POST |
| lineLimit    | int           | 非必填    | 10    | 多行日志采集情况下，每个alert中包含的最大日志行数                   |

