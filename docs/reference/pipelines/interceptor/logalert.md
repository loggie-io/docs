# logAlert

用于日志报警检测。  
属于source interceptor。
使用示例请参考[日志报警](../../../user-guide/monitor/service-log-alarm.md)。

!!! example

    ```yaml
    interceptors:
    - type: logAlert
      matcher:
        contains: ["error", "err"]
        regexp: ['.*example.*']
      ignore: ['.*INFO.*']
      sendOnlyMatched: true
      additions:
        module: "loggie"
        alertname: "alert-test"
        cluster: "local-cluster"
        namespace: "default"
      advanced:
        enabled: true
        mode: [ "noData","regexp" ]
        duration: 20
        matchType: "any"
        rules:
          - regexp: '(?<date>.*?) (?<time>[\S|\\.]+)  (<status>[\S|\\.]+) (?<u>.*?) --- (?<thread>\[*?\]) (?<pkg>.*) : (?<message>(.|\n|\t)*)'
            matchType: "any"
            groups:
              - key: status
                operator: "eq"
                value: WARN
              - key: thread
                operator: "eq"
                value: 200
          - regexp: '(?<date>.*?) (?<time>[\S|\\.]+) (?<status>[\S|\\.]+) (?<u>.*?) --- (?<thread>\[.*?\]) (?<pkg>.*) : (?<message>(.|\n|\t)*)'
            matchType: "any"
            groups:
              - key: status
                operator: "eq"
                value: ERROR
    ```

## matcher

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| matcher.contains | string数组  |    非必填    |    | 日志数据包含字符串检测 |
| matcher.regexp | string数组  |    非必填    |    | 日志数据正则检测 |
| matcher.target | string  |    非必填    |  body  | 根据日志数据的该字段进行检测，如果进行日志切分或者drop body字段，请填写所需字段 |


## ignore

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| ignore | string数组  |    非必填    |    | 正则表达式，若匹配，则忽略这条日志，向下传递，可用于告警时排除某些日志 |

## additions

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| additions | map |    非必填    |    | 发送alert时，额外添加的字段，会放在`_additions`字段中，可用作渲染。 |


## sendOnlyMatched

|    `字段`   | `类型` |  `是否必填`  | `默认值` | `含义`               |
| ---------- |------| ----------- |-------|--------------------|
| sendOnlyMatched | bool |    非必填    | false | 是否仅将匹配成功的数据发送至sink |


## advanced

| `字段`      | `类型`          |  `是否必填`  |  `默认值`  | `含义`                                   |
|-----------|---------------| ----------- | --------- |----------------------------------------|
| enabled   | bool          |    非必填    |  false  | 是否开启高级匹配模式                             |
| mode      | string列表      |    非必填    |    | 匹配模式 支持`regexp`和`noData`两种，可同时生效。      |
| duration  | time.Duration |    非必填    |    | noData模式必填，在一定时间内，没有日志会发出告警。           |
| matchType | string        |    非必填    |    | regexp模式必填，可选any或者all，表示匹配任意或者全部规则rule |
| rules     | Rule列表        |    非必填    |    | regexp模式必填，匹配规则列表                      |

### advanced.rule

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| regexp | string |    必填    |    | 正则分组表达式 |
| matchType | string |    必填    |    | 可选any或者all，表示匹配任意或者全部匹配组group |
| groups | group列表 |    必填    |    | 匹配组列表|

#### advanced.rule.group

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| key | string |    必填    |    | 分组匹配之后的键值 |
| operator | string |    必填    |    | 操作符，目前支持eq，gt，lt |
| value | string |    必填    |    | 目标值 |

