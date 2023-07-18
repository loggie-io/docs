# registry持久化存储接口

## /api/v1/source/file/registry
#### **URL**

```bash
GET /api/v1/source/file/registry
```
#### 描述

查询Loggie在本节点下文件的持久化采集进度。  
Loggie会将文件采集了多少的进度等信息持久化到本地，如果Loggie发生了重启，会从文件记录的进度offset开始采集，避免重头开始采集文件。


#### 请求参数
- format: 展示的格式，可以为json或者text，默认为json。

示例：

```
/api/v1/source/file/registry?format=text
```

#### 返回

示例：
```bash
{Id:1 PipelineName:local SourceName:demo Filename:/tmp/log/app2.log JobUid:75064440-16777234 Offset:259 CollectTime:2023-07-17 20:19:04.846 Version:0.0.1 LineNumber:9} 
{Id:2 PipelineName:local SourceName:demo Filename:/tmp/log/app.log JobUid:75610913-16777234 Offset:0 CollectTime:2023-07-17 20:19:12.343 Version:0.0.1 LineNumber:0} 
```

- JobUid：为文件的{inodeId}-{deviceId}拼接组成