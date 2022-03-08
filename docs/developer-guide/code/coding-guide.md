# 代码规范

## 开发规则

### 通用Golang规范
参考：

- [Effective Go](https://go.dev/doc/effective_go)
- [Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments)
- [Uber Go Style Guide](https://github.com/uber-go/guide/blob/master/style.md)

### Loggie的规范
1. 职责单一原则，一个component只做一件事，且做好一件事
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
12. 不允许使用`time.after()`。使用`time.NewTicker()`代替，并及时清理ticker

## 日志

* 日志应该被认真对待。在进行修改时，请花时间记录日志，以确保重要的事情被记录下来。特别是pipeline name、source name等组件的关键信息
* 日志语句应该是完整的句子，大小写适当，供不一定熟悉源代码的人阅读
* 打印日志直接使用项目预设的函数，例如`log.Info()`、`log.Error()`。不允许使用`fmt.Printf()`、`fmt.Println()`等go内置fmt方法。项目日志系统采用 **zerolog**

### 日志级别

* DEBUG：打印一些状态、提示信息，用于开发过程中观察，开发完成、正式上线后需要屏蔽，但在发生难以排查的异常时，可以打开以提供更多的详细内部信息。
* INFO：一些提示性的信息，即使在开发完成、正式上线的系统中，也有保留的价值。
* WARN：属于轻微的警告，程序中出现了一些异常情况，但是影响不大，还可以正常使用。
* ERROR：属于普通的错误，在程序可以控制的范围内，不会造成连锁影响或巨大影响。
* PANIC/FATAL：属于致命错误，会导致系统瘫痪、关闭，必须马上进行处理。一般在main函数启动流程中使用，正常运行过程中谨慎使用。并且请注意Loggie的多Pipeline设计上互相隔离，FATAL会影响所有Pipeline。

!!! note "PANIC和FATAL区别"

    PANIC最终调用的是panic(msg)

    FATAL最终调用的是os.Exit(1)

## 监控

* 任何新组件都应该带有适当的指标，以便监控功能正常工作
* 应该认真对待这些指标，并且只上报有用的指标，这些指标将在生产中用于监测/提醒系统的健康状况，或排查问题

## 单元测试

* 新功能需要包含单元测试
* 单元测试应该测试尽可能少的代码，不要启动整个服务，有外部依赖请补充e2e或者集成测试

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
    Copyright 2022 Loggie Authors

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

