# channel

channel queue,是基于go chan实现的内存缓冲queue。

!!! example

    ```yaml
    queue:
      type: channel

    ```

## batchSize

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| batchSize | int  |    不必填      |    2048  | 一个批次包含的event数量 |

## batchBufferFactor

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| batchBufferFactor | int  |    不必填      |    2  | queue缓冲区的大小(channel的容量)=batchSize*batchBufferFactor |

## batchBytes

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| batchBytes | int64  |    不必填      |    33554432(32MB)  | 一个批次包含的数据最大字节数 |

## batchAggTimeout

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| batchAggTimeout | time.Duration  |    不必填      |  1s   | 组装聚合多个event成一个batch等待的超时时间 |
