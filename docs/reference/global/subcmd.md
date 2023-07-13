# 子命令

## genfiles
模拟产生日志。可用于日志采集的压测和测试场景。

### 参数

- totalCount: 总的生成日志行数。
- lineBytes: 每行的日志长度。
- qps: 生成日志的qps。
- 其他log参数: genfiles子命令本质上是使用了loggie本身的日志框架去大量生成日志，所以loggie的[日志参数](./args.md#日志参数)也是genfiles子命令的参数。

### 使用方式

#### 使用loggie可执行文件

```bash
LOG_DIR=/tmp/log ## log directory
LOG_MAXSIZE=10 ## max size in MB of the logfile before it's rolled
LOG_QPS=0 ## qps of line generate
LOG_TOTAL=5 ## total line count
LOG_LINE_BYTES=1024 ## bytes per line
LOG_MAX_BACKUPS=5 ## max number of rolled files to keep

./loggie genfiles -totalCount=${LOG_TOTAL} -lineBytes=${LOG_LINE_BYTES} -qps=${LOG_QPS} \
	-log.maxBackups=${LOG_MAX_BACKUPS} -log.maxSize=${LOG_MAXSIZE} -log.directory=${LOG_DIR} -log.noColor=true \
	-log.enableStdout=false -log.enableFile=true -log.timeFormat="2006-01-02 15:04:05.000"

```

示例：
```bash
# Generate a log file loggie.log under /tmp/log, 
# which contains 1000 logs, each line of log is 1KB, and the total size is about 1.1MB


./loggie genfiles -totalCount=1000 -lineBytes=1024 -qps=0 \
	-log.maxBackups=1 -log.maxSize=1000 -log.directory=/tmp/log -log.noColor=true \
	-log.enableStdout=false -log.enableFile=true -log.timeFormat="2006-01-02 15:04:05.000"
```

在容器中使用，请参考部署catalog中的[deployment](https://github.com/loggie-io/catalog/tree/main/common/genfiles)，并exec到其中的Pod中执行以上命令。

#### 本地测试

示例：

```makefile
make genfiles LOG_TOTAL=1000
```
具体其他参数可以参考项目中的makefiles。




## version
查看Loggie的版本。

### 参数
无

### 使用方式

```bash
./loggie version
```