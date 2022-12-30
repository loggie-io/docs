# zinc

zinc sink用于发送数据至[zinc](https://github.com/zinclabs/zinc)存储。


!!! example

    ```yaml
    sink:
      type: zinc
      host: "http://127.0.0.1:4080"
      username: admin
      password: Complexpass#123
      index: "demo"    
    ```

## host

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| host | string  |    非必填    |   http://127.0.0.1:4080   | zinc的url地址 |

## username

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| username | string  |    非必填    |      | 发送至zinc的用户名 |

## password

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| password | string  |    非必填    |      | 发送至zinc的密码 |

## index

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| index | string  |    非必填    |  default    | 发送至zinc的index |

## SkipSSLVerify

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| SkipSSLVerify | bool  |    非必填    |  true    | 是否忽略SSL校验 |
