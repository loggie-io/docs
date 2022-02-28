# 代码规范

## 开发规则

1. 不允许使用`time.after()`。使用`time.NewTicker()`代替，并及时清理ticker
2. 启动任何一个goroutine都要先想好如何退出，以及编码退出逻辑
3. return err的时候想想是否需要清理本无需启动的goroutine
4. 永远记住go中的操作都不保证原子性（例如字符串拼接），除了chan、sync包等
5. for（for each）循环的时候，注意函数变量是共享相同的地址
6. 减少使用全局的done chan（大量goroutine轮询同一个done chan性能会急剧下降）。 更建议使用组件struct独立的done chan和提供Stop()方法来供上层调用来关闭以避免goroutine泄露
7. Component想好如何合理优雅Stop（特别是不要发生panic导致整个进程挂掉）
8. 不要关闭(close)一个用于读写的chan来达到退出goroutine或者停止Component的目标，可能在close chan后还有写操作会造成panic。 使用一个单独的done chan来标示是否退出
9. 关闭(Stop)Component的时候注意关闭的顺序和资源释放， 不要造成其他goroutine夯住(例如供外部写的0容量的chan没有处理，但是读的goroutine退出了，导致外部写的goroutine一直阻塞)
10. 将chan的生命周期封装在chan所有者内部
11. 使用done chan退出goroutine的时候，不要做任何清理资源的操作。清理资源优先使用defer，因为当goroutine panic的时候， defer还能够清理资源，而done chan中的清理逻辑可能永远不会被执行
12. 职责单一原则，一个component只做一件事，且做好一件事

## 日志

* 日志应该被认真对待。在进行修改时，请花时间记录日志，以确保重要的事情被记录下来。特别是pipeline name、source name等组件的关键信息
* 日志语句应该是完整的句子，大小写适当，供不一定熟悉源代码的人阅读
* 打印日志直接使用项目预设的函数，例如`log.Info()`、`log.Error()`。不允许使用`fmt.Printf()`、`fmt.Println()`等go内置fmt方法。项目日志系统采用 **zerolog**

### 日志级别

* INFO级别是你应该假定该程序将以何种级别运行。INFO信息是一些并不坏的东西，但每次出现时用户肯定想知道
* DEBUG级别是当有事情发生，你想弄清楚发生了什么的时候才打开的。DEBUG不应该太细，以至于会严重影响程序的性能
* WARN和ERROR表示某些东西是坏的。如果你不完全确定它是坏的，就使用WARN，如果你确定，就使用ERROR。并且ERROR级别 **默认** 会上报error metric，如果对接了loggie内置的eventBus中的alertListener（参考[metric上报/error metric](../metric/metric-guide.md)），将会直接发出报警

## 监控

* 任何新组件都应该带有适当的指标，以便监控功能正常工作（参考[metric上报](../metric/metric-guide.md)）
* 应该认真对待这些指标，并且只上报有用的指标，这些指标将在生产中用于监测/提醒系统的健康状况，或排查问题

## 单元测试

* 新功能需要包含单元测试（虽然目前loggie本身缺乏单元测试，恶补中🤣）
* 单元测试应该测试尽可能少的代码。不要启动整个服务器，除非没有其他方法可以单独测试单个组件的功能

## 代码风格

* go fmt
* 配置项命名使用驼峰。例如`yaml:"cleanDataTimeout,omitempty"`，代替`yaml:"clean_data_timeout,omitempty"`

## 向后兼容

* 协议应支持向后兼容性以实现无停机升级。这意味着同类source和sink必须能够同时支持来自新版和旧版的请求
* 元数据格式和数据格式应支持向后兼容。例如grcp的protobuf定义

## Copyright profile

!!! Copyright-text
    ```
    /*
    Copyright 2021 Loggie Authors

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

        http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.
    */
    ```

