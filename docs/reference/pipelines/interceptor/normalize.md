# normalize

用于日志切分处理。  
属于source interceptor。可指定只被某些source使用。  
具体使用示例可参考[日志切分与处理](../../../user-guide/best-practice/log-process.md)。

## processors

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| processors | 数组  |    必填    |  无    | 所有的处理processor列表 |

配置的processor将按照顺序依次执行。

!!! tips

    Loggie支持使用`a.b`的形式引用嵌套的字段。
    比如数据为:
    ```json
    {
      "fields": {
        "hello": "world"
      }
    }
    ```
    下面的processor配置中均可以使用`fields.hello`指定嵌套在`fields`里的`hello: world`。

### addMeta
默认情况下，Loggie不会添加任何的系统内部信息到原始数据中。  
可通过addMeta添加系统内置字段发送给下游。  

!!! note
    请注意，在pipeline中配置addMeta，只会影响该pipeline发送的所有数据，如果需要全局生效，请在[defaults](../../global/defaults.md)中配置normalize.addMeta。

    ```
    loggie:
      defaults:
        interceptors:
        - type: normalize
          name: global
          processors:
            addMeta: ~

    ```


|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| target | string  |    非必填    |  meta    | 系统内置字段添加到event中的字段名 |

### regex
将指定字段进行正则提取。  

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| regex.pattern | string  |    必填    |  无    | 正则解析规则 |
| regex.target | string  |    非必填    |  body    | 正则解析的目标字段 |

!!! example
    ```yaml
    interceptors:
    - type: normalize
      processors:
      - regex:
          pattern: '(?<ip>\S+) (?<id>\S+) (?<u>\S+) (?<time>\[.*?\]) (?<url>\".*?\") (?<status>\S+) (?<size>\S+)'
    ```

    使用以上的正则表达式，可以将以下示例的日志：
    ```
    10.244.0.1 - - [13/Dec/2021:12:40:48 +0000] "GET / HTTP/1.1" 404 683
    ```
    转换成：
    ```
    "ip": "10.244.0.1",
    "id": "-",
    "u": "-",
    "time": "[13/Dec/2021:12:40:48 +0000]",
    "url": "\"GET / HTTP/1.1\"",
    "status": "404",
    "size": "683"
    ```

具体配置的时候，建议先使用一些正则调试工具 (https://regex101.com/) 验证是否可以匹配。  

### jsonDecode
将指定字段json解析提取。

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| jsonDecode.target | string  |    非必填    |  body    | json decode的目标字段 |

!!! example
    ```yaml
    interceptors:
    - type: normalize
      processors:
      - jsonDecode: ~
    ```


### split
将指定字段通过分隔符进行提取。

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| split.target | string  |    非必填    |  body    | split的目标字段 |
| split.separator | string  |    必填    |  无    | 分隔符 |
| split.max | int  |    非必填    |  -1    | 通过分割符分割后得到的最多的字段数 |
| split.keys | string数组  |    必填    |  无   | 分割后字段对应的key |

!!! example
    === "base"
        ```yaml
        interceptors:
        - type: normalize
          processors:
          - split:
            separator: '|'
            keys: ["time", "order", "service", "price"]
        ```
        使用以上split配置可以将日志：
        ```
        2021-08-08|U12345|storeCenter|13.14
        ```
        转换成：
        ```
        "time": "2021-08-08"
        "order": "U12345"
        "service": "storeCenter"
        "price: 13.14
        ```

    
    === "max"
        ```yaml
        interceptors:
        - type: normalize
          processors:
          - split:
            separator: ' '
            max: 2
            keys: ["time", "content"]
        ```
        通过增加`max`参数，可以控制最多分割的字段。  
        比如以下日志:
        ```
        2021-08-08 U12345 storeCenter 13.14
        ```
        可以通过以上配置提取为:
        ```
        "time": "2021-08-08"
        "content": "U12345 storeCenter 13.14"
        ```



### drop
丢弃指定字段。

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| drop.targets | string数组  |    必填    |  无    | drop的字段 |

!!! example
    ```yaml
    interceptors:
    - type: normalize
      processors:
      - drop:
          targets: ["id", "body"]
    ```


### rename
重命名指定字段。

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| rename.convert | 数组  |    必填    |  无    |  |
| rename.convert[n].from | string  |    必填    |  无    | rename的目标 |
| rename.convert[n].to | string  |    必填    |  无    | rename后的名称 |

!!! example
    ```yaml
    interceptors:
    - type: normalize
      processors:
      - rename:
          convert:
          - from: "hello"
            to: "world"
    ```

### add
新增字段。  

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| add.fields | map  |    必填    |  无    | 新增的key:value值 |

!!! example
    ```yaml
    interceptors:
    - type: normalize
      processors:
      - add:
          fields:
            hello: world
    ```

### convert

字段类型转换。

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| convert.convert | 数组  |    必填    |  无    |  |
| convert.convert[n].from | string  |    必填    |  无    | 需要转换的字段名 |
| convert.convert[n].to | string  |    必填    |  无    | 转换后的类型，可为："bool", "integer", "float" |

!!! example
    ```yaml
    interceptors:
    - type: normalize
      processors:
      - convert:
          convert:
          - from: count
            to: float
    ```

### copy

字段复制。

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| copy.convert | 数组  |    必填    |  无    |  |
| copy.convert[n].from | string  |    必填    |  无    | 需要复制的字段名 |
| copy.convert[n].to | string  |    必填    |  无    | 复制后的字段名 |

!!! example
    ```yaml
    interceptors:
    - type: normalize
      processors:
      - copy:
          convert:
          - from: hello
            to: world
    ```

### underRoot

将字段中的所有`key:value`放到event最外层。

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| underRoot.keys | string数组  |    必填    |  无    | 需要underRoot的字段名 |

!!! example
    ```yaml
    interceptors:
    - type: normalize
      processors:
      - copy:
          convert:
          - from: hello
            to: world
    ```


### timestamp
转换时间格式。

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| timestamp.convert | 数组  |    必填    |  无    |  |
| timestamp.convert[n].from | string  |    必填    |  无    | 指定转换时间格式的字段 |
| timestamp.convert[n].fromLayout | string  |    必填    |  无    | 指定字段的时间格式(golang形式) |
| timestamp.convert[n].toLayout | string  |    必填    |  无    | 转换后的时间格式(golang形式)，另外可为`unix`和`unix_ms` |
| timestamp.convert[n].toType | string  |    非必填    |  无    | 转换后的时间字段类型 |

!!! example
    ```yaml
    interceptors:
    - type: normalize
      processors:
      - timestamp:
          convert:
          - from: logtime
            fromLayout: "2006-01-02T15:04:05Z07:00"
            toLayout: "unix"
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







