# 开源项目对比


|                        | Loggie                                                       | Filebeat                                        | Fluentd          | Logstash | Flume    |
| ---------------------- | ------------------------------------------------------------ | ----------------------------------------------- | ---------------- | -------- | -------- |
| 开发语言               | Golang                                                       | Golang                                          | Ruby             | JRuby     | Java     |
| 多Pipeline             | 支持                                                         | 单队列                                          | 单队列           | 支持     | 支持     |
| 多输出源           | 支持                                                         | 不支持，仅一个Output                          |        配置copy          | 支持     | 支持     |
| 中转机                 | 支持                                                         | 不支持                                          | 支持             | 支持     | 支持     |
| 日志报警               | 支持                                                         | 不支持                                          | 不支持           | 不支持   | 不支持   |
| Kubernetes容器日志采集 | 支持容器的stdout和容器内部日志文件                           | 只支持容器stdout                                | 只支持容器stdout | 不支持   | 不支持   |
| 配置下发               | Kubernetes下可通过CRD配置，主机场景配置中心陆续支持中 | 手动配置                                        | 手动配置         | 手动配置 | 手动配置 |
| 监控               | 原生支持Prometheus metrics，同时可配置单独输出指标日志文件、发送metrics等方式 | API接口暴露，接入Prometheus需使用额外的exporter |        支持API和Prometheus metrics          |   需使用额外的exporter       |      需使用额外的exporter    |
| 资源占用               | 低                                                           | 低                                              | 一般             | 较高     | 较高     |


## 基准性能测试与对比

**测试环境：**

- 物理机 48C，256G
- Kafka 3 Broker，挂载SSD盘
- Filebeat v7.8版本，无配置processor等处理；Loggie默认包含cost、retry、metric interceptor；

**测试目的：**

Filebeat和Loggie的性能对比

**测试思路：**

Filebeat和Loggie，均采集日志发送至Kafka，观察相应的资源占用和发送吞吐量

**测试详情：**

单文件自动生成5000000行日志，每行内容如下所示：

```
[13/May/2021:10:20:29 +0800] 0.015 10.200.170.107 "GET /static/3tJHS3Ubrf.html?activity_channel_id=22=1_00000&fromMiniapp=1&miniapp_uuid=uEd93lG2eG8Qj5fRXuiJwNt4bmiylkmg HTTP/1.1" 200 138957 "110.183.45.54, 10.200.151.37" act.you.163.com "" "Mozilla/5.0 (Linux; Android 8.1.0; PADM00Build/O11019; wv) AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Chrome/67.0.3396.87 XWEB/568 MMWEBSDK/190102 Mobile Safari/537.36 MMWEBID/6881 MicroMessenger/7.0.3.1400(0x2700033B) Process/appbrand0 NetType/WIFI Language/zh_CN miniProgram" "" [127.0.0.1:8990] [0.014] [] [] immsg={"st":1553307293614,"sb":138963,"rc":200,"cf":{"sr":1},"if":"default","ut":14,"sv":"static","pd":"activity","qb":764}
```

配置Filebeat和Loggie采集日志，并发送至Kafka某个Topic，不使用客户端压缩，Kafka Topic配置Partition为3。

在保证Agent规格资源充足的情况下，修改采集的文件个数、发送客户端并发度（配置Filebeat worker和Loggie parallelism)，观察各自的CPU、Memory和Pod网卡发送速率。

测试得到如下数据：

| Agent    | 文件大小     | 日志文件数 | 发送并发度 | CPU  | MEM (rss)        | 网卡发包速率 |
| -------- | -------- | ---------- | ---------- | -------- | ----------- | ------------------- |
| Filebeat | 3.2G     | 1          | 3          | 7.5~8.5c | 63.8MiB | 75.9MiB/s            |
| Filebeat | 3.2G     | 1          | 8          | 10c      | 65MiB   | 70MiB/s              |
| Filebeat | 3.2G     | 10         | 8          | 11c      | 65MiB   | 80MiB/s              |
|          |          |          |            |            |          |             |                     |
| Loggie   | 3.2G     | 1          | 3          | 2.1c     | 60MiB   | 120MiB/s             |
| Loggie   | 3.2G     | 1          | 8          | 2.4c     | 68.7MiB | 120MiB/s             |
| Loggie   | 3.2G     | 10         | 8          | 3.5c     | 70MiB   | 210MiB/s             |

**测试结论：**

相同压测条件和场景下：

- Loggie和Filebeat消耗的CPU相比，大概仅为后者的1/4，同时发送吞吐量为后者的1.6～2.6倍。

- Memory相当，均处于较低的水准。

- Filebeat的极限吞吐量存在瓶颈，80MB/s后很难提升，而Loggie则可以达到200MiB/s以上。

