# 业务日志报警

除了Loggie本身的报警，业务日志本身的监控报警也是一个常用的功能，比如在日志中包含了ERROR日志，可以发送报警，这种报警会更贴近业务本身，是基于metrics报警的一种很好的补充。  

## 使用方式

有以下两种方式可以选择：

- **采集链路检测报警**：Loggie可以在Agent采集日志的时候，或者在中转机转发的时候，检测到匹配的异常日志，然后发送报警
- **独立链路检测报警**：单独部署Loggie，使用Elasticsearch source或者其他的source查询日志，然后匹配检测发送报警

## 采集链路检测

### 原理
采集链路不需要独立部署Loggie，但是由于在采集的数据链路上进行匹配，理论上会对传输性能造成一定影响，但胜在方便简单。  

`logAlert interceptor`用于在日志传输的时候检测异常日志，异常日志会被封装成报警的事件发送至monitor eventbus的`logAlert` topic，由`logAlert listener`来消费。`logAlert listener`支持发送至任意http后端（可以多个）。发送体根据自定义模板进行渲染，若模板未定义，则会发送原始数据。在配置模板前，可以先观察原始数据（设置debug模式启动），再进行模板配置，原始数据可能会根据pipeline配置被其他interceptor改动而与示例不同。

### 配置示例

配置新增`logAlert listener`。详细配置可参考[LogAlert](../../reference/monitor/logalert.md)。

!!! config

    ```yaml
    loggie:
      monitor:
        logger:
          period: 30s
          enabled: true
        listeners:
          logAlert:
            alertManagerAddress: ["http://127.0.0.1:8080/loggie"]
            bufferSize: 100
            batchTimeout: 10s
            batchSize: 10
            linelimit: 10
            template: |
              {
                  "alerts":
                        [
                        {{$first := true}}
                        {{range .Alerts}}
                        {{if $first}}{{$first = false}}{{else}},{{end}}
                        {
                              "labels": {
                                "topic": "{{.fields.topic}}"
                              },
                              "annotations": {
                                "message": "\nNew alert: \nbody:\n{{range .body}}{{.}}\n{{end}}\ncontainerid: {{._meta.pipelineName}}\nsource: {{._meta.sourceName}}\ncontainername: {{.fields.containername}}\nlogconfig: {{.fields.logconfig}}\nname: {{.fields.name}}\nnamespace: {{.fields.namespace}}\nnodename: {{.fields.nodename}}\npodname: {{.fields.podname}}\nfilename: {{.state.filename}}\n",
                                "reason": "{{.reason}}"
                              },
                              "startsAt": "{{._meta.timestamp}}",
                              "endsAt": "{{._meta.timestamp}}"
                        }
                        {{end}}
                        ],
                        {{$first := true}}
                        {{range .Alerts}}
                        {{if $first}}{{$first = false}}{{else}}
                        "commonLabels": {
                          "namespace": "{{._additions.namespace}}",
                          "module": "{{._additions.module}}",
                          "alertname": "{{._additions.alertname}}",
                          "cluster": "{{._additions.cluster}}"
                        }
                        {{end}}
                        {{end}}
              }
          filesource: ~
          filewatcher: ~
          reload: ~
          queue: ~
          sink: ~
      http:
        enabled: true
        port: 9196
    ```

模版使用go template。可参考[GO Template](https://pkg.go.dev/text/template)，
[GO Template教学](https://cloud.tencent.com/developer/article/1683688)。

其中：

!!! 原始alert数据

  ```json
    {
      "Alerts": [
        {
          "_meta": {
            "pipelineName": "default/spring",
            "sourceName": "loggie-source-756fd6bb94-4skqv/loggie-alert/common",
            "timestamp": "2022-10-28T13:12:30.528824+08:00"
          },
          "body": [
            "2022-10-28 01:48:07.093 ERROR 1 --- [nio-8080-exec-1] o.a.c.c.C.[.[.[/].[dispatcherServlet]    : Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed; nested exception is java.lang.ArithmeticException: / by zero] with root cause",
            "",
            "java.lang.ArithmeticException: / by zero"
          ],
          "fields": {
            "containerid": "0dc5f07983bfdf7709ee4fce752679983c4184e94c70dab5fe6df5843d5cbb68",
            "containername": "loggie-alert",
            "logconfig": "spring",
            "name": "loggie-source",
            "namespace": "default",
            "nodename": "docker-desktop",
            "podname": "loggie-source-756fd6bb94-4skqv",
            "topic": "loggie"
          },
          "reason": "matches some rules",
          "state": {
            "bytes": 6913,
            "filename": "/var/log/pods/default_loggie-source-756fd6bb94-4skqv_9da3e440-e749-4930-8e4d-41e0d5b66417/loggie-alert/1.log",
            "hostname": "docker-desktop",
            "offset": 3836,
            "pipeline": "default/spring",
            "source": "loggie-source-756fd6bb94-4skqv/loggie-alert/common",
            "timestamp": "2022-10-28T13:12:30.527Z"
          },
          "_additions": {
            "namespace": "default",
            "cluster": "local",
            "alertname": "loggie-test",
            "module": "loggie"
          }
        }
      ]
    }
  ```

原始alert数据，为一个json，其中`Alerts`为固定的key，其value为alert列表。

**每个alert字段解释**：

| `字段`               | `是否自带` | `含义`                  |
|--------------------|--------|-----------------------|
| _meta              | 是      | alert元数据              |
| _meta.pipelineName | 是      | alert元数据              |
| _meta.sourceName   | 是      | alert元数据              |
| _meta.timestamp    | 是      | alert元数据              |
| body               | 是      | logBody               |
| fields             | 否      | field字段，由其余配置添加       |
| reason             | 是      | 匹配成功原因                |
| state              | 是      | 采集信息，一般为file source自带 |
| _additions         | 否      | 由配置指定                 |



`_meta` 为固定的字段，包含`pipelineName`，`sourceName`，`timestamp`。

增加`logAlert interceptor`，同时在ClusterLogConfig/LogConfig中引用。其中`additions`为给alert额外添加的字段，会放入alert原始数据的`_addtions`字段中，可用做模板渲染。

建议先使用debug模式（`-log.level=debug`）观察原始alert数据格式，再配置模板进行渲染，原始数据会受到其他配置的影响，这里仅展示一个示例。

详细配置可参考[LogAlert Interceptor](../../reference/pipelines/interceptor/logalert.md)。

!!! config

    ```yaml
    apiVersion: loggie.io/v1beta1
    kind: Interceptor
    metadata:
      name: logalert
    spec:
      interceptors: |
        - type: logAlert
          matcher:
            contains: ["ERROR"]
          sendOnlyMatched: true
          additions:
            module: "loggie"
            alertname: "loggie-test"
            cluster: "local"
            namespace: "default"
    ```

可以配置接收方发送至其他服务来报警。

接收方接收到类似的报警：

!!! example

    ```json
    {
    "alerts": [
        {
            "labels": {
                "topic": "loggie"
            },
            "annotations": {
                "message": "\nNew alert: \nbody:\n2022-10-28 01:48:07.093 ERROR 1 --- [nio-8080-exec-1] o.a.c.c.C.[.[.[/].[dispatcherServlet]    : Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed; nested exception is java.lang.ArithmeticException: / by zero] with root cause\n\njava.lang.ArithmeticException: / by zero\ncontainerid: 0dc5f07983bfdf7709ee4fce752679983c4184e94c70dab5fe6df5843d5cbb68\nsource: loggie-source-756fd6bb94-4skqv/loggie-alert/common\ncontainername: loggie-alert\nlogconfig: spring\nname: loggie-source\nnamespace: default\nnodename: docker-desktop\npodname: loggie-source-756fd6bb94-4skqv\nfilename: /var/log/pods/default_loggie-source-756fd6bb94-4skqv_9da3e440-e749-4930-8e4d-41e0d5b66417/loggie-alert/1.log\n",
                "reason": "matches some rules"
            },
            "startsAt": "2022-10-28T13:12:30.527Z",
            "endsAt": "2022-10-28T13:12:30.527Z"
        }
    ],
    "commonLabels": {
        "namespace": "default",
        "module": "loggie",
        "alertname": "loggie-test",
        "cluster": "local"
      }
    }
    ```

我们收到类似报警：
```
内容 : New alert:
body:
2022-12-09 09:48:55.190 ERROR 1 --- [nio-8080-exec-7] o.a.c.c.C.[.[.[/].[dispatcherServlet]    : Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed; nested exception is java.lang.ArithmeticException: / by zero] with root cause

java.lang.ArithmeticException: / by zero

containerid: 0dc5f07983bfdf7709ee4fce752679983c4184e94c70dab5fe6df5843d5cbb68
source: loggie-source-756fd6bb94-4skqv/loggie-alert/common
containername: loggie-alert
logconfig: spring
name: loggie-source
namespace: default
nodename: docker-desktop
podname: loggie-source-756fd6bb94-4skqv
filename: /var/log/pods/default_loggie-source-756fd6bb94-4skqv_9da3e440-e749-4930-8e4d-41e0d5b66417/loggie-alert/1.log
```




## 独立链路检测

### 原理
Loggie配置source采集日志，经过`logAlert interceptor`匹配时，可配置`sendOnlyMatched`仅将匹配成功的日志发送至`alertwebhook sink`，匹配失败的日志看作正常日志被忽略。建议在使用`alertwebhook sink`时，同时开启`logAlert interceptor`, 设置`sendOnlyMatched`为`true`搭配使用。


### 配置示例

配置新增`alertwebhook sink`。详细配置可参考[AlertWebhook Sink](../../reference/pipelines/sink/webhook.md)。

!!! config
  ```yaml
      sink:
        type: webhook
        addr: http://localhost:8080/loggie
        linelimit: 10
        template: |
              {
                  "alerts":
                        [
                        {{$first := true}}
                        {{range .Alerts}}
                        {{if $first}}{{$first = false}}{{else}},{{end}}
                        {
                              "labels": {
                                "topic": "{{.fields.topic}}"
                              },
                              "annotations": {
                                "message": "\nNew alert: \nbody:\n{{range .body}}{{.}}\n{{end}}\ncontainerid: {{._meta.pipelineName}}\nsource: {{._meta.sourceName}}\ncontainername: {{.fields.containername}}\nlogconfig: {{.fields.logconfig}}\nname: {{.fields.name}}\nnamespace: {{.fields.namespace}}\nnodename: {{.fields.nodename}}\npodname: {{.fields.podname}}\nfilename: {{.state.filename}}\n",
                                "reason": "{{.reason}}"
                              },
                              "startsAt": "{{._meta.timestamp}}",
                              "endsAt": "{{._meta.timestamp}}"
                        }
                        {{end}}
                        ],
                        {{$first := true}}
                        {{range .Alerts}}
                        {{if $first}}{{$first = false}}{{else}}
                        "commonLabels": {
                          "namespace": "{{._additions.namespace}}",
                          "module": "{{._additions.module}}",
                          "alertname": "{{._additions.alertname}}",
                          "cluster": "{{._additions.cluster}}"
                        }
                        {{end}}
                        {{end}}
              }
  ```

`logAlert Interceptor`配置和接收方收到的报警与采集链路检测报警类似。