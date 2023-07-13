# addHostMeta

用于主机部署情况下，新增主机的一些元信息参数。

!!! example

    ```yaml
    interceptors:
      - type: addHostMeta
        addFields:
          hostname: "${hostname}"
          ip: "${ip}"
          os: "${os}"
          platform: "${platform}"
          kernelVersion: "${kernelVersion}"
          kernelArch: "${kernelArch}"
    ```

## addFields

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| addFields | map  |    必填    |      | 需要添加的元信息 |

目前支持添加的元信息字段有：

- `${hostname}`：节点名称
- `${ip}`: 节点的IPv4地址数组
- `${os}`：操作系统
- `${kernelVersion}`: 内核版本
- `${kernelArch}`
- `${platform}`
- `${platformFamily}`
- `${platformVersion}`


加上以上的元信息，展示出的日志示例：
!!! example

    ```json
     {
        "@timestamp": "2023-07-13T07:13:50.394Z",
        "host": {
            "kernelVersion": "22.2.0",
            "os": "darwin",
            "platform": "darwin",
            "platformFamily": "Standalone Workstation",
            "platformVersion": "13.1",
            "hostname": "xxxMacBook-Pro.local",
            "ip": [
                "10.xxx.xxx.221",
                "192.xxx.xxx.1"
            ],
            "kernelArch": "arm64"
        },
        "body": "xxx"
    }
    ```

## fieldsName
|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| fieldsName | string  |    非必填    |   host   | 添加的元信息字段名称 |

