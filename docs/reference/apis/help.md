# help接口

## /api/v1/help
#### **URL**

```bash
GET /api/v1/help
```
#### 描述

查询该Loggie Agent的内部配置日志采集状态

#### 请求参数
- pipeline: 查看某个pipelineName的配置和日志采集状态 
- source: 查看某个sourceName的配置和日志采集状态
- status: 可为pending，即只展示正在采集的，比如100%，NaN%和ignored的日志文件，但包括0%的

示例：

```
/api/v1/help?pipeline=test&status=pending
```
表示只返回pipeline为test的正在采集的所有日志文件状态

#### 返回

示例如下：

```bash
--------- Usage: -----------------------
|--- view details: /api/v1/help?detail=<module>, module is one of: pipeline/log
|--- query by pipeline name: /api/v1/help?pipeline=<name>
|--- query by source name: /api/v1/help?source=<name>

--------- Pipeline Status: --------------
all 1 pipelines running
  * pipeline: local, sources: [demo]

✅ pipeline configurations consistency check passed

pipelines:
- name: local
  cleanDataTimeout: 5s
  queue:
    type: channel
  interceptors:
  - name: global
    type: schema
    addMeta:
      timestamp:
        key: '@timestamp'
    order: 700
  - type: metric
  - type: maxbytes
  - type: retry
  sources:
  - name: demo
    type: file
    fieldsUnderKey: fields
    fields:
      topic: loggie
    paths:
    - /tmp/log/*.log
    watcher:
      maxOpenFds: 6000
  sink:
    type: dev
    parallelism: 1
    codec:
      type: json
      printEvents: true
      pretty: true

| more details:
|--- pipelines configuration in the path ref: /api/v1/reload/config
|--- current running pipelines configuration ref: /api/v1/controller/pipelines

--------- Log Collection Status: ---------

all activeFdCount: 0, inActiveFdCount: 1

> pipeline * source - filename | progress(offset/size) | modify
  > local
    * demo
      - /tmp/log/app.log | 100.00%(189/189) | 2023-07-17T17:10:33+08:00

| more details:
|--- registry storage ref: /api/v1/source/file/registry
```

- Pipeline Status
  - 表示多少pipeline正在运行
  - `pipeline configurations consistency check passed`：表示检查磁盘上的pipeline配置和Loggie内存中的是否一致
  - pipelines配置：被Loggie最终读取到的配置，包括了全局default字段和默认的字段融合最终的配置
- Log Collection Status
  - 所有active和inactive的日志文件个数：请注意，这里的active和inactive是Loggie内部判断活跃度的机制，和file source中[maxContinueRead](../../reference/pipelines/source/file.md#maxcontinueread)，[maxContinueReadTimeout](../../reference/pipelines/source/file.md#maxcontinuereadtimeout)，[maxEofCount](../../reference/pipelines/source/file.md#maxeofcount)有关系。比如说一个文件1s打印一条日志，默认配置下其实并不是active。
  - 日志采集的进度：包括文件路径，采集进度，上次修改时间等

## /api/v1/help/log

#### **URL**

```bash
GET /api/v1/help/log
```

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
|  fileStatus.pipeline.`<name>`    | 管道状态，对应配置中的pipeline name，参考下面的[pipeline](help.md#pipeline)参数   |   map    |          |

##### pipeline

| 参数名称 | 说明     | 参数类型  | 备注 |
| -------- | -------- | -------- | -------- |
|  source.`<name>`    | pipeline中source的状态，参考下面的[source](help.md#source)参数   |   map    |          |

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
