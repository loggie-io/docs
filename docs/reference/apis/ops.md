# 运维类接口

## 查询日志采集状态

#### URL

GET /api/v1/help/log

#### 描述

查询该Loggie Agent的日志采集状态

#### 请求参数

- pipeline: 表示只查询某个pipeline的状态
- status: 如果status=pending，表示只返回正在采集的日志文件状态（包括0%），忽略采集进度100%、文件不存在NaN%和ignored状态

示例：

```
/api/v1/help/log?pipeline=test&status=pending
```
表示只返回pipeline为test的正在采集的所有日志文件状态

#### 返回参数

| 参数名称 | 说明     | 参数类型  | 备注 |
| -------- | -------- | -------- | -------- |
|  fdStatus    | 文件句柄状态   |       |          |
|  fdStatus.activeFdCount    | 活跃的fd个数   |   int    |          |
|  fdStatus.inActiveFdCount    | 不活跃的fd个数   |  int     |          |
|  fileStatus    | 文件采集状态   |       |          |
|  fileStatus.pipeline.`<name>`    | 管道状态，对应配置中的pipeline name，参考下面的[pipeline](ops.md#pipeline)参数   |   map    |          |

##### pipeline

| 参数名称 | 说明     | 参数类型  | 备注 |
| -------- | -------- | -------- | -------- |
|  source.`<name>`    | pipeline中source的状态，参考下面的[source](ops.md#source)参数   |   map    |          |

##### source

| 参数名称 | 说明     | 参数类型  | 备注 |
| -------- | -------- | -------- | -------- |
|  paths    | 配置文件source中定义的path   |   string数组    |          |
|  detail    | pipeline中source的状态   |   数组    |          |
|  detail[n].filename    | 文件名称   |   string    |          |
|  detail[n].offset    | 采集进度offset   |   int    |          |
|  detail[n].size    | 文件大小   |   int    |          |
|  detail[n].modify    | 文件最近的更新时间   |   int    |      unix milliseconds    |
|  detail[n].ignored    | 文件是否被忽略（由file source中的ignoreOlder配置决定）   |   bool    |          |


!!! example

    ```json
    {
        "fdStatus": {
            "activeFdCount": 0,
            "inActiveFdCount": 1
        },
        "fileStatus": {
            "pipeline": {
                "local": {
                    "source": {
                        "demo": {
                            "paths": [
                                "/tmp/log/*.log"
                            ],
                            "detail": [
                                {
                                    "filename": "/tmp/log/access.log",
                                    "offset": 469,
                                    "size": 469,
                                    "modify": 1673436846523,
                                    "ignored": false
                                }
                            ]
                        }
                    }
                }
            }
        }
    }
    ```
