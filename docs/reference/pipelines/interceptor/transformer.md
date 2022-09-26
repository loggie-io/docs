# transformer
带有条件判断的函数式数据处理interceptor。  
属于source interceptor。

## 使用场景
- 日志提取出日志级别level，并且drop掉DEBUG日志
- 日志里混合包括有json和plain的日志形式，可以判断json形式的日志并且进行处理
- 根据访问日志里的status code，增加不同的topic字段
- ...

示例[参考](https://github.com/loggie-io/loggie/blob/main/pkg/interceptor/transformer/example/pipeline.yml)。

## 使用方式

transformer会按照配置的actions里顺序执行所有的action。action类似函数的方式，可以写入参数，参数一般为event里的字段。  
同时，每个action里还可能包括额外的控制字段。比如下面regex(body)，body即为regex的参数，pattern为额外的字段。

```yaml
interceptors:
  - type: transformer
    actions:
      - action: regex(body)
        pattern: ^(?P<time>[^ ^Z]+Z) (?P<level>[^ ]*) (?P<log>.*)$
      - action: add(topic, common)
```

另外，action还支持条件判断`if-then-else`的方式：

```yaml
- if: <condition>
  then:
    - action: funcA()
  else:
    - action: funcB()
```

其中，condition条件判断也为函数的形式。

```yaml
interceptors:
  - type: transformer
    actions:
      - if: equal(status, 404)
        then:
          - action: add(topic, not_found)
          - action: return()
```

## action

### 共有字段

- ignoreError: 表示是否忽略该action处理过程中的错误，并且不会打印错误日志。

!!! example

    ```yaml
    - type: transformer
      actions:
        - action: regex(body)
          pattern: (?<ip>\S+) (?<id>\S+) (?<u>\S+) (?<time>\[.*?\]) (?<url>\".*?\") (?<status>\S+) (?<size>\S+)
          ignoreError: true
    ```
    这里的ignoreError设置为true，表示会忽略该正则匹配的错误，并且会继续执行后续的action。
    

### add(key, value)
给event添加额外的key:value。

!!! example

    ```yaml
    - action: add(topic, loggie)
    ```
    
    input:
    ```json
    {
      "body": "2021-02-16T09:21:20.545525544Z DEBUG this is log body"
    }
    ```
    
    output:
    ```json
    {
      "body": "2021-02-16T09:21:20.545525544Z DEBUG this is log body",
      "topic": "loggie"
    }
    ```

### copy(from, to)
复制event里的字段。

参数：

- from: 原有的key
- to: 复制后的key

!!! example

    ```yaml
    - action: copy(foo, bar)
    ```
    
    input:
    ```json
    {
      "body": "2021-02-16T09:21:20.545525544Z DEBUG this is log body",
      "foo": "loggie"
    }
    ```
    
    output:
    ```json
    {
      "body": "2021-02-16T09:21:20.545525544Z DEBUG this is log body",
      "foo": "loggie",
      "bar": "loggie"
    }
    ```


### move(from, to)
移动/重命名字段。

参数：

- from: 原有的key
- to: 移动后的key

!!! example

    ```yaml
    - action: move(foo, bar)
    ```
    
    input:
    ```json
    {
      "body": "2021-02-16T09:21:20.545525544Z DEBUG this is log body",
      "foo": "loggie"
    }
    ```
    
    output:
    ```json
    {
      "body": "2021-02-16T09:21:20.545525544Z DEBUG this is log body",
      "bar": "loggie"
    }
    ```


### set(key, value)
更新字段key的值为value。

参数：

- key: 需更新的字段
- value: 更新的后的值

!!! example

    ```yaml
    - action: set(foo, test)
    ```
    
    input:
    ```json
    {
      "body": "2021-02-16T09:21:20.545525544Z DEBUG this is log body",
      "foo": "loggie"
    }
    ```
    
    output:
    ```json
    {
      "body": "2021-02-16T09:21:20.545525544Z DEBUG this is log body",
      "foo": "test"
    }
    ```


### del(key1, key2...)
删除字段。可填写多个字段key。

!!! example

    ```yaml
    - action: del(foo)
    ```
    
    input:
    ```json
    {
      "body": "2021-02-16T09:21:20.545525544Z DEBUG this is log body",
      "foo": "loggie"
    }
    ```
    
    output:
    ```json
    {
      "body": "2021-02-16T09:21:20.545525544Z DEBUG this is log body",
    }
    ```

### underRoot(key)
将嵌套的字段放在根部（最外层）。

!!! example

    ```yaml
    - action: underRoot(state)
    ```
    
    input:
    ```json
    {
      "body": "2021-02-16T09:21:20.545525544Z DEBUG this is log body",
      "state": {
        "node": "127.0.0.1",
        "phase": "running"
      }
    }
    ```
    
    output:
    ```json
    {
      "body": "2021-02-16T09:21:20.545525544Z DEBUG this is log body",
      "node": "127.0.0.1",
      "phase": "running"
    }
    ```


### fmt(key)
将某个字段的值重新渲染，可根据其他的字段值组成一个值。如果key不存在则会新增该字段。

额外字段：

- pattern: 必填，表示格式化的规则，比如`${state.node}-${state.phase}`。如果pattern为固定值，则类似set(key, value)。

!!! example

    ```yaml
    - action: fmt(status)
      pattern: ${state.node} is ${state.phase}
    ```
    
    input:
    ```json
    {
      "body": "2021-02-16T09:21:20.545525544Z DEBUG this is log body",
      "state": {
        "node": "127.0.0.1",
        "phase": "running"
      }
    }
    ```
    
    output:
    ```json
    {
      "body": "2021-02-16T09:21:20.545525544Z DEBUG this is log body",
      "state": {
        "node": "127.0.0.1",
        "phase": "running"
      }，
      "status": "127.0.0.1 is running"
    }
    ```


### timestamp(key)
字段的时间格式转换。

额外字段：

- fromLayout: 必填，指定字段的时间格式(golang形式)，也可为`unix`和`unix_ms`
- fromLocation: 指定字段的时区，也可为`UTC`或者`Local`，如果为空，则为`UTC`
- toLayout: 必填，转换后的时间格式(golang形式)，也可为`unix`和`unix_ms`
- toLocation: 转换后的时间时区，也可为`UTC`或者`Local`，如果为空，则为`UTC`

!!! example

    ```yaml
    - action: timestamp(time)
      fromLayout: "2006-01-02 15:04:05"
      fromLocation: Asia/Shanghai
      toLayout: unix_ms
      toLocation: Local
    ```
    
    input:
    ```json
    {
      "body": "2021-02-16T09:21:20.545525544Z DEBUG this is log body",
      "time": "2022-06-28 11:24:35"
    }
    ```
    
    output:
    ```json
    {
      "body": "2021-02-16T09:21:20.545525544Z DEBUG this is log body",
      "time": 1656386675000
    }
    ```

以上的layout参数需要填写golang形式，可参考：
```
const (
	Layout      = "01/02 03:04:05PM '06 -0700" // The reference time, in numerical order.
	ANSIC       = "Mon Jan _2 15:04:05 2006"
	UnixDate    = "Mon Jan _2 15:04:05 MST 2006"
	RubyDate    = "Mon Jan 02 15:04:05 -0700 2006"
	RFC822      = "02 Jan 06 15:04 MST"
	RFC822Z     = "02 Jan 06 15:04 -0700" // RFC822 with numeric zone
	RFC850      = "Monday, 02-Jan-06 15:04:05 MST"
	RFC1123     = "Mon, 02 Jan 2006 15:04:05 MST"
	RFC1123Z    = "Mon, 02 Jan 2006 15:04:05 -0700" // RFC1123 with numeric zone
	RFC3339     = "2006-01-02T15:04:05Z07:00"
	RFC3339Nano = "2006-01-02T15:04:05.999999999Z07:00"
	Kitchen     = "3:04PM"
	// Handy time stamps.
	Stamp      = "Jan _2 15:04:05"
	StampMilli = "Jan _2 15:04:05.000"
	StampMicro = "Jan _2 15:04:05.000000"
	StampNano  = "Jan _2 15:04:05.000000000"
)
```
还可以根据实际情况修改。  

### regex(key)
使用正则的方式切分日志，提取字段。
另外也可以为regex(key, to)。

参数：

- key: 必填，正则提取的字段
- to: 非必填，提取后所有的字段放置到的key。默认为空，表示将字段提取到根部

额外字段：

- pattern: 必填，正则表达式

!!! example

    ```yaml
    - action: regex(body)
      pattern: (?<ip>\S+) (?<id>\S+) (?<u>\S+) (?<time>\[.*?\]) (?<url>\".*?\") (?<status>\S+) (?<size>\S+)
    ```
    
    input:
    ```json
    {
      "body": "10.244.0.1 - - [13/Dec/2021:12:40:48 +0000] 'GET / HTTP/1.1' 404 683",
    }
    ```
    
    output:
    ```json
    {
      "ip":     "10.244.0.1",
	  "id":     "-",
	  "u":      "-",
	  "time":   "[13/Dec/2021:12:40:48 +0000]",
	  "url":    "GET / HTTP/1.1",
	  "status": "404",
	  "size":   "683",
    }
    ```

### jsonDecode(key)
将json文本反序列化。
也可以为jsonDecode(key, to)。

参数：

- key: 必填，对应的字段key
- to: 非必填，提取后所有的字段放置到的key。默认为空，则表示将字段提取到根部

!!! example

    ```yaml
    - action: jsonDecode(body)
    ```
    
    input:
    ```json
    {
      "body": `{"log":"I0610 08:29:07.698664 Waiting for caches to sync", "stream":"stderr", "time":"2021-06-10T08:29:07.698731204Z"}`,
    }
    ```
    
    output:
    ```json
    {
      "log": "I0610 08:29:07.698664 Waiting for caches to sync",
      "stream": "stderr", 
      "time": "2021-06-10T08:29:07.698731204Z"
    }
    ```


### strconv(key, type)
字段值类型转换。

参数：

- key: 目标字段
- type: 转换后的类型，可为`bool`, `int`, `float`

!!! example

    ```yaml
    - action: strconv(code, int)
    ```
    
    input:
    ```json
    {
      "body": "2021-02-16T09:21:20.545525544Z DEBUG this is log body",
      "code": "200"
    }
    ```
    
    output:
    ```json
    {
      "body": "2021-02-16T09:21:20.545525544Z DEBUG this is log body",
      "code": 200
    }
    ```

### print()
打印event。一般用于调试阶段使用。

### return()
控制类型函数，执行到return()后返回，不再继续执行下面的action。

### dropEvent()
控制类型函数，执行到dropEvent()后会将该event直接丢弃。这意味着该条数据会丢失，也不会继续被后续的interceptor或者sink处理消费。

!!! example

    ```yaml
    interceptors:
      - type: transformer
        actions:
          - action: regex(body)
            pattern: ^(?P<time>[^ ^Z]+Z) (?P<level>[^ ]*) (?P<log>.*)$
          - if: equal(level, DEBUG)
            then:
              - action: dropEvent()
    ```
    假设日志为：`2021-02-16T09:21:20.545525544Z DEBUG this is log body`，则满足level字段为DEBUG，会直接丢弃该条日志。

## condition
条件判断类函数。

### 操作符

- AND：表示两个condition执行结果的`和`。

!!! example

    ```yaml
    interceptors:
      - type: transformer
        actions:
          - if: equal(level, DEBUG) AND equal(code, 200)
            then:
              - action: dropEvent()
    ```


- OR：表示两个condition执行结果的`或`。

!!! example

    ```yaml
    interceptors:
      - type: transformer
        actions:
          - if: equal(level, DEBUG) OR equal(level, INFO)
            then:
              - action: dropEvent()
    ```


- NOT：表示对condition执行结果取`反`。

!!! example

    ```yaml
    interceptors:
      - type: transformer
        actions:
          - if: NOT equal(level, DEBUG)
            then:
              - action: dropEvent()
    ```

### equal(key, target)
目标字段值是否和参数值target相等。

### contain(key, target)
字段值是否包含参数值target。

### exist(key)
目标字段是否存在或者是否为空。

### greater(key, value)
目标字段的值是否大于参数值value。

### less(key, value)
目标字段的值是否小于参数值value。

### hasPrefix(key, prefix)
目标字段的值是否包含prefix前缀。

### match(key, regex)
目标字段的值是否和参数regex正则匹配。

### oneOf(key, value1, value2...)
目标字段的值是否是参数值value1...其中之一。
