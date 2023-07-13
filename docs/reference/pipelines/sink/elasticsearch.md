# elasticsearch

使用Elasticsearch sink发送数据至Elasticsearch集群。

!!! example

    ```yaml
    sink:
      type: elasticsearch
      hosts: ["elasticsearch1:9200", "elasticsearch2:9200", "elasticsearch3:9200"]
      index: "log-${fields.service}-${+YYYY.MM.DD}"
    ```


!!! caution

    如果elasticsearch版本为v6.x，请加上以下`etype: _doc`参数。
    
    ```yaml
    sink:
      type: elasticsearch
      etype: _doc
      ...
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

## headers

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| headers | map  |    非必填    |   无  | 请求Elasticsearch携带的header |

## parameters

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| parameters | map  |    非必填    |   无  | 请求Elasticsearch的url参数 |

## apiKey

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| apiKey | string  |    非必填    |     | 用于授权的base64编码令牌;如果设置，则覆盖用户名/密码和服务令牌 |


## serviceToken

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| serviceToken | string  |    非必填    |     | 服务令牌用于授权;如果设置，则覆盖用户名/密码 |

## caCertPath

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| caCertPath | string  |    非必填    |     | pem编码的ca证书存放的路径 |

## compress

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| compress | bool  |    非必填    |   false  | 是否压缩请求体 |

## documentId

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| documentId | string  |    非必填    |     | 发送至elasticsearch的id值，可使用`${}`的方式取某个字段 |

## opType

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| opType | string  |    非必填    |   index  | 参考[官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/master/docs-index_.html#docs-index-api-query-params), 如果目标为datastream，则需要设置为create |

## sendBufferBytes

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| sendBufferBytes | int  |    非必填    |   131072  | 发送写入请求体的buffer字节数大小 |
