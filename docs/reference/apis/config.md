# Pipeline config配置相关接口

## /api/v1/reload/config

#### **URL**

```bash
GET /api/v1/reload/config
```

#### 描述

查看Loggie磁盘上的Pipeline配置文件内容。
所以每次调用该接口都会从Loggie启动参数`-config.pipeline`中指定的配置文件中读取一次。  
但是该配置文件并不一定是当前Loggie内存中所读取到的配置，因为存在reload间隔时间、配置文件格式错误等情况。

#### 请求参数
无

#### 返回

Pipeline配置文件文本内容。

## /api/v1/controller/pipelines

#### **URL**


```bash
GET /api/v1/controller/pipelines
```

#### 描述

查看Loggie内存中被读取到的配置文件，也是正在运行的Pipeline配置。  
该接口返回的配置内容和[help](help.md#apiv1help)接口中的一致。

#### 请求参数
无

#### 返回

Pipeline配置文件文本内容。


