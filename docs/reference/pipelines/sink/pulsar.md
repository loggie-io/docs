# pulsar

pulsar sink用于发送数据至[pulsar](https://github.com/apache/pulsar)存储。  
该sink为beta试用状态，请谨慎使用于生产环境。


!!! example

    ```yaml
    sink:
      type: pulsar
      url: pulsar://localhost:6650
      topic: my-topic
    ```


## url

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| url| string  |    必填    |   无  | 日志发送端pulsar连接地址 |

## topic

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| topic | string  |    必填    |   无  | 发送日志至pulsar的topic |


## producerName

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| producerName | string  |    非必填    |   无  | specifies a name for the producer |

## properties

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| producerName | map  |    非必填    |   无  | Properties specifies a set of application defined properties for the producer |


## operationTimeoutSeconds

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| operationTimeoutSeconds| time.Duration|    非必填    |   30s  | Producer-create, subscribe and unsubscribe operations will be retried until this interval, after which the operation will be marked as failed |


## connectionTimeout

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| connectionTimeout| time.Duration|    非必填    |   5s | Timeout for the establishment of a TCP connection |


## sendTimeout

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| sendTimeout| time.Duration|    非必填    |   30s | SendTimeout set the timeout for a message that is not acknowledged by the server 30s |


## maxPendingMessages

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| sendTimeout| time.Duration|    非必填    |  无 | MaxPendingMessages specifies the max size of the queue holding the messages pending to receive an acknowledgment from the broker |



## hashingSchema

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| hashingSchema| int|    非必填    |  0 |HashingScheme is used to define the partition on where to publish a particular message. 0:JavaStringHash，1:Murmur3_32Hash |


## compressionType

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| compressionType | int|    非必填    |   0 | 0:NoCompression, 1:LZ4, 2:ZLIB, 3:ZSTD |

## compressionLevel

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| compressionLevel | int|    非必填    |   0 | 0:Default, 1:Faster, 2:Better |


## logLevel

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| logLevel| string  |    非必填    |   0 | 日志级别: "info","debug", "error" |



## batchingMaxSize

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| batchingMaxSize| int  |    非必填    |   2048(KB)  | BatchingMaxSize specifies the maximum number of bytes permitted in a batch |



## batchingMaxMessages

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| batchingMaxMessages| int  |    非必填    |   1000  |BatchingMaxMessages specifies the maximum number of messages permitted in a batch |



## batchingMaxPublishDelay

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| batchingMaxPublishDelay| time.Duration  |    非必填    |   10ms  | BatchingMaxPublishDelay specifies the time period within which the messages sent will be batched |

## useTLS

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| useTLS| bool  |    非必填    |   false  | 是否使用TLS认证 |


## tlsTrustCertsFilePath

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| tlsTrustCertsFilePath| string  |    非必填    |   无  | the path to the trusted TLS certificate file |



## tlsAllowInsecureConnection

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| tlsAllowInsecureConnection| bool|    非必填    |   false  | Configure whether the Pulsar client accept untrusted TLS certificate from broker |



## certificatePath

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| certificatePath| string |    非必填    |   无 | TLS证书路径 |



## privateKeyPath

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| privateKeyPath| string |    非必填    |   无 | TLS privateKey路径 |



## token

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| token | string|    非必填    |  无  |  如果使用token认证鉴权pulsar，请填写此项|



## tokenFilePath

|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| tokenFilePath| string|    非必填    |  无  |  auth token from a file|
