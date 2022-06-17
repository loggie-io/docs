# 数据格式

!!! info
    了解Loggie内部数据格式的设计，能帮助我们配置合适的日志处理和日志格式转换

## 结构设计
在Loggie内部的日志数据，包括：

- **body**: source接收的原始数据。比如使用file source采集的日志文件里的一行日志数据
- **header**: 用户使用的字段。比如用户自己添加到source里的fields字段，用户切分日志后得到的字段等
- **meta**: Loggie系统内置的元信息，默认不会发送给下游



## 格式转换
如果以上的格式不满足需求，可以参考：

- [日志元信息](../best-practice/log-enrich.md)

## 日志切分
对于原始日志数据的切分与处理，请使用 **normalize interceptor**，请参考：

- [日志切分处理](../best-practice/log-process.md)