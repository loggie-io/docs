# db

持久化数据的配置。保存采集过程中的文件名称、文件inode、文件采集的offset等信息。用来在loggie reload或者重启后恢复上一次的采集进度。

v1.5版本及以后新引入了[badger持久化引擎](../../developer-guide/build.md)，可替换之前的sqlite，避免使用CGO。

!!! caution
    
    请注意，**不兼容改动**：在v1.5之后（包括v1.5），file source中的db移动到这里为全局配置。

    如果从低版本升级到v1.5及以后版本，请务必检查file source是否有配置过db。如果没有配置，可忽略，默认值会保持兼容。

!!! example
    
    === "sqlite"

        ```yml
        db:
          file: /opt/data/loggie.db
        ```

    === "badger"

        ```yml
        db:
          file: /opt/data/badger
        ```        

## file
|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| file | string  |    非必填    |   badger: `./data/badger`，sqlite: `./data/loggie.db`   | 持久化的目录文件 |

## flushTimeout
|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| flushTimeout | time.Duration  |    非必填    |   2s   |  |

## bufferSize
|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| bufferSize | int  |    非必填    |   2048   | 持久化写入使用的buffer |

## cleanInactiveTimeout
|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| cleanInactiveTimeout | time.Duration  |    非必填    |   504h   | 如果一条记录长时间没有被更新，则会被清理，默认为21d(504h) |

## cleanScanInterval
|    `字段`   |    `类型`    |  `是否必填`  |  `默认值`  |  `含义`  |
| ---------- | ----------- | ----------- | --------- | -------- |
| cleanInactiveTimeout | time.Duration  |    非必填    |   1h   | 清理逻辑执行的时间间隔 |
