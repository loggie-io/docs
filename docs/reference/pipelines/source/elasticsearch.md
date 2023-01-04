# elasticsearch

消费elasticsearch的数据。

!!! example

    === "常规"

        ```yaml
        pipelines:
          - name: local
            sources:
              - type: elasticsearch
                name: elastic
                hosts: ["localhost:9200"]
                indices: ["blog*"]
                size: 10 # data size per fetch
                interval: 30s # pull data frequency
        ```


    === "高级"

        ```yaml
        pipelines:
          - name: local
            sources:
              - type: elasticsearch
                name: elastic
                hosts:
                  - "localhost:9200"
                  - "localhost:9201"
                indices: ["blog*"]
                username: "bob"
                password: "bob"
                schema: ""
                sniff: false
                gzip: true
                includeFields: # pull selected field
                  - Title
                  - Content
                  - Author
                excludeFields: # exclude selected field
                  - Content
                query: | # elastic query phrases
                  {
                    "match": {"Title": "bob"}
                  }
                size: 10 # data size per fetch
                interval: 30s # pull data frequency
                timeout: 5s # pull timeout
                db: 
                  flushTimeout: 2s # persistent the elastic pull location frequency
                  cleanInactiveTimeout: 24h # delete the db record after the time
                  cleanScanInterval: 1h # check the expired db record frequency
        ```

## hosts

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| hosts | string数组  |    必填    |     | 消费的elasticsearch url地址 |

## indices

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| index | string数组  |    必填    |     | 查询elasticsearch的index名称 |

## username

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| username | string  |    非必填    |     | 消费elasticsearch的用户名 |

## password

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| password | string  |    必填    |     | 消费elasticsearch的密码 |

## schema

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| schema | string  |    非必填    |  http   | HTTP scheme(http/https)，sniff的时候使用 |

## gzip

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| gzip | bool  |    非必填    |  false   | 是否开启gzip压缩 |

## includeFields

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| includeFields | string数组  |    非必填    |     | 只返回指定的_source字段 |

## excludeFields

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| excludeFields | string数组  |    非必填    |     | 排除指定的_source字段 |

## query

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| query | string  |    非必填    |     | 查询elasticsearch的表达式 |


## size

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| size | int  |    非必填    |  100   | 每次请求得到hits返回的个数 |

## interval

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| interval | time.Duration  |    非必填    |  30s   | 定时请求elasticsearch的时间间隔 |

## timeout

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| timeout | time.Duration  |    非必填    |  5s   | 请求的超时时间 |

## db

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| db |   |    非必填    |     | 持久化记录查询elasticsearch请求的进度，会存储至elasticsearch中，避免Loggie重启后重复消费数据 |
| db.indexPrefix | string  |    非必填    |  .loggie-db   | 默认情况下，loggie会将持久化的数据定时写入格式为`${indexPrefix}-${pipelineName}-${sourceName}`的index中 |
| db.flushTimeout |  time.Duration |    非必填    |  2s   | 持久化数据写入的间隔时间 |
| db.cleanInactiveTimeout |  time.Duration |    非必填    |  504h (21day)   | 清理过期的持久化数据超时时间 |
| db.cleanScanInterval |  time.Duration |    非必填    |  1h   | 检查过期时间间隔 |

