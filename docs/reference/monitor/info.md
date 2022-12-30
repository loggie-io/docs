# info listener

展示Loggie本身的一些信息。

## 配置
|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| period | time.Duration  |    非必填    |   10s   |  暴露指标的时间间隔 |


## Metrics

#### loggie_info_stat

```
# HELP loggie_info_stat Loggie info
# TYPE loggie_info_stat gauge
loggie_info_stat{version=v1.4} 1
```
其中的version表示Loggie自身的版本号（版本号在Loggie构建的时候被注入，如果未出现正确的版本号，请检查使用go构建编译的参数）。