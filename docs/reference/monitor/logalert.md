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

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| addr | string数组  |    必填    |      | 发送alert的http地址 |
| bufferSize | int  |    非必填    |   100   | 日志报警发送的buffer大小，单位为报警事件个数 |
| batchTimeout | time.Duration  |    非必填    |   10s   | 每个报警发送batch的最大发送时间 |
| batchSize | int  |    非必填    |   10   | 每个报警发送batch的最大包含报警请求个数 |
| template | string  |    非必填    |     | 渲染发送的alert结构体的go template模板 |
| timeout | int  |    非必填    |   30   | 发送alert的http timeout，单位为秒 |
| headers | map  |    非必填    |      | 发送alert的http header |
| lineLimit | int  |    非必填    |   10   | 多行日志采集情况下，每个alert中包含的最大日志行数 |

