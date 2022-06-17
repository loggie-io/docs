# sls

sls sink用于将日志发送至[阿里云可观测统一存储SLS](https://www.aliyun.com/product/sls)。  

!!! example

    ```yaml
    sink:
      type: sls
      name: demo
      endpoint: cn-hangzhou.log.aliyuncs.com
      accessKeyId: ${id}
      accessKeySecret: ${secret}
      project: test
      logstore: test1
      topic: myservice
    ```

## endpoint

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| endpoint | string  |    必填    |      | SLS存储的访问域名 |

你可以在具体project页面的项目概览中查看到。

## accessKeyId

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| accessKeyId | string  |    必填    |      | 访问的accessKeyId，请查看阿里云账号的访问凭证管理 |

建议使用阿里云的子账号，子账号需要有对应project、logstore的权限。

## accessKeySecret

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| accessKeySecret | string  |    必填    |      | 访问的accessKeySecret，请查看阿里云账号的访问凭证管理 |

## project

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| project | string  |    必填    |      | SLS存储的project名称 |

## logstore

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| logstore | string  |    必填    |      | SLS存储的logstore名称 |

## topic

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| topic | string  |    非必填    |      | 日志主题（Topic）是日志服务的基础管理单元。您可在采集日志时指定Topic，用于区分日志  |




