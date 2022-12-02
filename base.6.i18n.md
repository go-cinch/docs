# 国际化


内置国际化[i18n](https://github.com/nicksnyder/go-i18n)支持, 若你有多语言需求, 这将是一个很棒的解决方案

!> proto中的validate暂时不支持, 默认是英文


## 映射关系


多语言的映射关系存放在`internal/server/middleware/locales`
- `en.yml` - 英文

    ```yaml
    too.many.requests: 'too many requests, please try again later'
    data.not.change: 'data has not changed'
    duplicate.field: 'duplicate field'
    record.not.found: 'record not found'
    no.permission: 'no permission to access this resource'
    idempotent.token.missing: 'idempotent token is missing'
    idempotent.token.invalid: 'idempotent token is invalid'
    ```

- `zh.yml` - 中文

    ```yaml
    too.many.requests: '请求频繁, 请稍候重试'
    data.not.change: '数据没有发生变化'
    duplicate.field: '字段重复'
    record.not.found: '记录不存在'
    no.permission: '无权访问'
    idempotent.token.missing: '缺少幂等性签名'
    idempotent.token.invalid: '幂等性签名无效'
    ```

?> 在[这里](http://www.iana.org/assignments/language-subtag-registry/language-subtag-registry)找你想要的语言

## 用法


`common/middleware/i18n`, 一行代码即可搞定

```go
import (
    "auth/api/reason" // auth是项目名
    "github.com/go-cinch/common/middleware/i18n"
)

func jwt(c *conf.Bootstrap, client redis.UniversalClient) func(handler middleware.Handler) middleware.Handler {
    return func(handler middleware.Handler) middleware.Handler {
        return func(ctx context.Context, req interface{}) (rp interface{}, err error) {
            ...
            err = reason.ErrorUnauthorized(i18n.FromContext(ctx).T(biz.JwtMissingToken))
            ...
        }
    }
}
```

?> 参见[auth.jwt](https://github.com/go-cinch/auth/blob/dev/internal/server/middleware/permission.go#L58), `internal/biz`和`internal/data`都有使用案例, 自己去发掘啦~

!> 必须使用`FromContext(ctx)`, 从上下文获取当前语言, 若不存在, 默认使用`configs/config.yaml`中`server.language`配置  


开启[auth服务](/started.0.init?id=auth%e6%9c%8d%e5%8a%a1)简单测试下

```shell
# 无参数, 使用系统默认值
curl http://127.0.0.1:6060/info
# {"code":401, "reason":"UNAUTHORIZED", "message":"token is missing", "metadata":{}}

# Accept-Language参数为中文
curl -H "Accept-Language: zh" http://127.0.0.1:6060/info
# {"code":401,"reason":"UNAUTHORIZED","message":"缺少jwt签名","metadata":{}}
# 或
curl -H "Accept-Language: zh-CN,zh;q=0.9" http://127.0.0.1:6060/info
# {"code":401,"reason":"UNAUTHORIZED","message":"缺少jwt签名","metadata":{}}
```
