# 2022 Loggie RoadMap

## 更多组件与功能扩展
- 持久化queue
- 流处理能力：聚合、计算等（类似pulsar的funtion，或者理解为轻量级flink）
- source: elasticsearch、http
- sink: file, clickhouse, pulsar, influxdb等
- WASM形态支持自定义日志解析处理
- 支持Knative/KEDA等Serverless形态扩缩容指标，实现中转机解析处理的自动伸缩

## 服务发现与配置中心
- 主机模式的配置下发：支持Consul, Apollo等

## 云原生与Kubernetes
- 自动注入Loggie sidecar形态支持
- opentelemetry兼容与支持
