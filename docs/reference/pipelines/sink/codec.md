# codec

codec主要为sink发送时根据需求进行格式的转换。  

!!! example
    ```yaml
    codec:
      type: json
      codec:
        type: json
        beatsFormat: true
        transformer:
        - rename:
            target:
            - from: systemState
              to: meta
        - drop:
            target: ["systemPipelineName", "systemSourceName", "meta.collectTime"]
    ```


## type: json

### pretty

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| codec.pretty | bool  |    否    |  false    | 是否将json格式日志展示成便于阅读的形式 |


### beatsFormat

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| codec.beatsFormat | bool  |    否    |  false    | 是否将json格式日志展示成便于阅读的形式 |


### prune
|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| codec.prune | bool  |    否    |  false    | 是否为精简模式，精简模式下将不发送system开头的系统保留字段 |

!!! example 
    ```yaml
    sink:
      type: dev
      printEvents: true
      codec:
        type: json
        pretty: false
        beatsFormat: true
        prune: true
    ```

### transformer

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| codec.transformer | 数组 |    否    |   无   | 日志格式转换transformer |

transformer会按配置顺序执行。  
字段名称支持通过`.`来引用多层级。比如使用`fields.hostname`来引用在`fields`结构里的`hostname`字段。

!!! example
    ```yaml
    sink:
      type: dev
      printEvents: true
      codec:
        type: json
        transformer:
        - add:
            target:
              agent: loggie
          drop:
            target: ["unUsedKey"]
          rename:
            target:
            - from: fields.host_name
              to: fields.hostname
    ```


#### transformer.add
|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| codec.transformer.add.target | map |    否    |   无   | 增加字段 |

#### transformer.drop
|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| codec.transformer.add.target | string数组 |    否    |   无   | 删除不用的字段 |

#### transformer.move
|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| codec.transformer.move.target | 数组 |    否    |   无   | 移动字段 |
| codec.transformer.move.target[n].from | string |    必填    |   无   | 移动字段的来源 |
| codec.transformer.move.target[n].to | string |    必填    |   无   | 移动字段的去向 |

#### transformer.copy
|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| codec.transformer.copy | 数组 |    否    |   无   | 复制字段 |
| codec.transformer.copy.target[n].from | string |    必填    |   无   | 复制字段的来源 |
| codec.transformer.copy.target[n].to | string |    必填    |   无   | 复制字段的去向 |

#### transformer.rename
|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| codec.transformer.rename | 数组 |    否    |   无   | 重命名字段 |
| codec.transformer.rename.target[n].from | string |    必填    |   无   | 重命名字段的来源 |
| codec.transformer.rename.target[n].to | string |    必填    |   无   | 重命名字段的去向 |

#### transformer.timestamp
|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| codec.transformer.timestamp | 数组 |    否    |   无   | 格式化转换时间 |
| codec.transformer.timestamp[n].from | string |    必填    |   无   | 格式化转换时间的字段 |
| codec.transformer.timestamp[n].fromLayout | string |    必填    |   无   | 格式化转换前时间字段格式(golang layout形式) |
| codec.transformer.timestamp[n].toLayout | string |    必填    |   无   | 格式化转换时间后时间字段格式(golang layout形式) |

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


