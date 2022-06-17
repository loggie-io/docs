
推荐参考已有的组件开发一个新的组件

## 公共接口

```go
// 生命周期接口
type Lifecycle interface {
    // 初始化，例如初始化Kafka连接
    Init(context Context)
    // 启动运行，例如开始消费Kafka
    Start()
    // 停止
    Stop()
}

// 描述接口
type Describable interface {
    // 类别，例如source
    Category() Category
    // 类型，例如kafka
    Type() Type
    // 自定义描述
    String() string
}

// 配置获取接口
type Config interface {
    // 获取配置
    Config() interface{}
}

// 组件接口
type Component interface {
    // 生命周期管理
    Lifecycle
    // 描述管理
    Describable
    // 配置管理
    Config
}
```

    
## source组件
source组件对接数据源输入，开发一个新的source插件需要实现如下接口
```go
// source组件接口
type Source interface {
    Component
    Producer
	// 提交接口，确认sink端成功然后提交
    Commit(events []Event)
}

// 生产接口，source组件需要实现
type Producer interface {
	// 对接数据源
    ProductLoop(productFunc ProductFunc)
}
```

## sink组件
sink组件对接输出端，开发一个新的sink插件需要实现如下接口
```go
 // sink组件接口
type Sink interface {
    Component
    Consumer
}

// 消费接口，sink组件需要实现
type Consumer interface {
	// 对接输出端
    Consume(batch Batch) Result
}
```

## interceptor组件

interceptor组件对事件进行拦截处理，开发一个新的interceptor插件需要实现如下接口
```go

// interceptor组件接口
type Interceptor interface {
    Component
	// 拦截处理
    Intercept(invoker Invoker, invocation Invocation) api.Result
}
```


!!! note
    请注意新增的组件需要放到pkg/include/include.go的import当中注册
