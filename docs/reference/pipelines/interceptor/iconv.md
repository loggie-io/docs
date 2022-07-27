# iconv

编码转换，用于将不同的编码转换为utf8，当下支持的编码转换格式.


!!! example

    ```yaml
    interceptors:
     - type: iconv
       charset: "gbk"
    ```

## charset

|    `字段`   |    `类型`    | `是否必填` | `默认值` |  `含义`  |
| ---------- | ----------- |--------|-------| -------- |
| charset | string  | 否      | utf-8 | 提取字段的匹配模型 |

当前支持的转换为utf-8的编码格式有
- `nop`
- `plain`
- `utf-8`
- `gbk`
- `big5`
- `euc-jp`
- `iso2022-jp`
- `shift-jis`
- `euc-kr`
- `iso8859-6e`
- `iso8859-6i`
- `iso8859-8e`
- `iso8859-8i`
- `iso8859-1`
- `iso8859-2`
- `iso8859-3`
- `iso8859-4`
- `iso8859-5`
- `iso8859-6`
- `iso8859-7`
- `iso8859-8`
- `iso8859-9`
- `iso8859-10`
- `iso8859-13`
- `iso8859-14`
- `iso8859-15`
- `iso8859-16`
- `cp437`
- `cp850`
- `cp852`
- `cp855`
- `cp858`
- `cp860`
- `cp862`
- `cp863`
- `cp865`
- `cp866`
- `ebcdic-037`
- `ebcdic-1040`
- `ebcdic-1047`
- `koi8r`
- `koi8u`
- `macintosh`
- `macintosh-cyrillic`
- `windows1250`
- `windows1251`
- `windows1252`
- `windows1253`
- `windows1254`
- `windows1255`
- `windows1256`
- `windows1257`
- `windows1258`
- `windows874`
- `utf-16be-bom`
- `utf-16le-bom`
