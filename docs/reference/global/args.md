## 系统参数

- `-config.from`: 默认为file，即默认使用文件的配置方式。可选：env，此时会从环境变量中读取配置（此时不支持reload）。
- `-config.system`: 默认为loggie.yml，表示指定Loggie系统配置的路径和文件名称。  
  （如果`-config.from=env`，则为system配置的环境变量名称）
- `-config.pipeline`: 默认为pipelines.yml，表示Pipeline配置文件所在的路径，需要填写符合glob匹配的路径，比如具体的路径和文件名`/etc/loggie/pipelines.yml`，或者glob匹配的方式，比如`/etc/loggie/*.yml`。  
  （如果`-config.from=env`，则为pipeline配置的环境变量名称）

!!! warning
    值得注意的是，如果`config.pipeline=/etc/loggie`，glob匹配会认为`/etc/loggie`为`/etc`目录下的`loggie`文件，而不是匹配`/etc/loggie`目录下的文件，请避免类似的设置方式

- `-meta.nodeName`：默认情况下会使用系统的hostname，在Kubernetes部署中会使用Downward API来注入nodeName。一般情况下不需要单独配置

## 日志参数:  

- `-log.level`: 日志级别，默认为info，可配置为debug、info、warn和error
- `-log.jsonFormat`: 是否将日志输出为json格式，默认为true
- `-log.enableStdout`: 是否输出标准输出日志，默认为true
- `-log.enableFile`: 是否输出日志文件，默认为false，即不输出日志文件，默认打印到标准输出
- `-log.directory`: 日志文件的路径，默认为/var/log，当log.enableFile=true时生效
- `-log.filename`: 日志文件的名称，默认为loggie.log，一般同log.directory搭配使用
- `-log.maxSize`: 日志轮转的时候，最大的文件大小，默认为1024MB
- `-log.maxBackups`: 日志轮转最多保留的文件个数，默认为3
- `-log.maxAge`: 日志轮转最大保留的天数，默认为7
- `-log.timeFormat`: 每行日志输出的时间格式，默认格式为`2006-01-02 15:04:05`

!!! info
    Loggie的日志轮转使用[lumberjack](`https://github.com/natefinch/lumberjack`)库

