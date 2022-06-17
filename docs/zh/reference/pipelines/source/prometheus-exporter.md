# prometheusExporter

采集Prometheus Metrics的指标数据。

!!! example
    ```yaml
    sources:
    - type: prometheusExporter
      name: metric
      endpoints:
      - "http://127.0.0.1:9196/metrics"
    ```

## endpoints

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| endpoints | string数组  |    必填    |     | 抓取的远端exporter地址，请注意Loggie不会默认在请求路径中添加/metrics |


## interval

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| interval | time.Duration  |    非必填    |  30s   | 定时抓取远端exporter的时间间隔 |

## timeout

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| timeout | time.Duration  |    非必填    |  5s   | 抓取请求的超时时间 |

## toJson

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| toJson | bool  |    非必填    |  false   | 是否将抓取到的prometheus原生指标，转换成JSON格式 |

## labels

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| labels | map  |    非必填    |     | 给所有metrics指标增加额外的label |

labels支持配置${_env.XX}的方式获取环境变量。
例如，配置：
!!! example "labels"

    ```yaml
        sources:
        - type: prometheusExporter
          name: metric
          endpoints:
          - "http://127.0.0.1:9196/metrics"
          labels:
            svc: ${_env.SVC}
    
    ```
假设环境变量SVC=test，会将所有的metrics加上svc=test的label。