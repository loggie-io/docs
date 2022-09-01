# 字段动态变量

在很多场景下，我们往往需要动态的获取event里的某个字段。
比如：

- kafka sink里的topic，或者elasticsearch sink里的index，根据event某个字段的值来动态的生成。我们可以使用`${a.b}`的方式来取值。
- 在transformer interceptor中，操作某个字段：copy(a.b, a.c)。

以下event为例:  
```json
{
    "fields": {
        "svc": "test",
    }
}
```

kafka sink topic配置为：`log-${fields.svc}`，则最终渲染生成的为`log-test`。


!!! caution

    一般可以使用`.`点号来表示嵌套的字段。但是，如果字段本身就包括`.`号，则需要使用`[]`包围起来，避免误认为是一个嵌套的字段。
    比如：
    ```json
    {
        "fields": {
            "a.b": "demo",
        }
    }
    ```
    需要使用`${fields.[a.b]}`来表示字段`a.b`。
