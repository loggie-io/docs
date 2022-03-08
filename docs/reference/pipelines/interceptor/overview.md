# Overview

interceptors字段为数组，一个Pipeline中可填写多个interceptor组件配置。

目前，interceptor分为两种类型： 

- source interceptor：运行在source发送数据到queue的过程中，`source -> source interceptor -> queue`。
- sink interceptor：运行在queue到sink的过程中，`queue -> sink interceptor -> sink`。

一个interceptor只属于其中一种。大部分组件为source interceptor类型，可支持配置belongTo被部分source使用。少数通用性质的比如retry interceptor为sink interceptor类型。

## Interceptor通用配置

### name

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| name | string  |    非必填    |     | 表示interceptor的名称。当pipeline里配置相同type interceptor的情况下，必填，用于区分标识 |

### belongTo

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| belongTo | string数组  |    非必填    |     | 仅source interceptor可用，用于指定该interceptor仅被哪些source使用 |


### order

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| order | int  |    非必填    |     | interceptor的排列顺序权重 |
