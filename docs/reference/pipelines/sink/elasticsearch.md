# elasticsearch

使用Elasticsearch sink发送数据至Elasticsearch集群。

!!! example

    ```yaml
    sink:
      type: elasticsearch
      hosts: ["elasticsearch1:9200", "elasticsearch2:9200", "elasticsearch3:9200"]
      index: "log-${fields.service}-${+YYYY.MM.DD}"
    ```

## hosts

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| hosts | string数组  |    必填    |   无  | 发送日志至Elasticsearch的地址 |


## index

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| index | string  |    必填    |   无  | 发送至Elasticsearch后，日志数据存储的index |

可以使用`${a.b}`的方式获取日志数据里的字段，或者加上`${+YYYY.MM.DD.hh}`时间戳等方式来动态生成index。

## username

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| username | string  |    非必填    |   无  | 如果Elasticsearch配置了用户名密码验证，需要填写请求的用户名 |



## password

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| password | string  |    非必填    |   无  | 如果Elasticsearch配置了用户名密码验证，需要填写请求的密码 |

## schema

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| schema | string  |    非必填    |   http  | client sniffing时使用 |

## sniff

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| sniff | bool  |    非必填    |   true  | 是否开启sniffer |

## gzip

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| gzip | bool  |    非必填    |   false  | 发送数据是否开启gzip压缩 |



