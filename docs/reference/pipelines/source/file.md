# file

file source用于日志采集。  

!!! example

    ```yaml
    sources:
    - type: file
      name: accesslog

    ```

## paths
|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| paths | string数组  |    必填      |    无  | 采集的path路径，使用glob表达式来匹配 |


## excludeFiles

## ignoreOlder

## ignoreSymlink

## workerCount

## readChanSize

## readBufferSize

## maxContinueRead

## maxContinueReadTimeout

## inactiveTimeout

## multi
