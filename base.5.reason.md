# 错误管理


## 错误码


我们通过[reason-proto]来统一管理比较常用的几个错误状态码

- `500 default` - 服务器内部错误, 如运行时异常
- `429 TOO_MANY_REQUESTS` - 请求过于频繁, 如超出ratelimit中间件限制
- `400 ILLEGAL_PARAMETER` - 参数不合法
- `400 NOT_FOUND` - 数据记录不存在, 这里仍用400, 我们认为本属于参数不合法范畴, 但有些情况下需要判断是否数据不存在
- `401 UNAUTHORIZED` - 认证失败, 如用户token不合法
- `403 FORBIDDEN` - 禁止访问, 如用户无权限访问


```protobuf
enum ErrorReason {
  // default reason
  option (errors.default_code) = 500;

  TOO_MANY_REQUESTS = 0 [(errors.code) = 429];
  ILLEGAL_PARAMETER = 1 [(errors.code) = 400];
  NOT_FOUND = 2 [(errors.code) = 400];
  UNAUTHORIZED = 3 [(errors.code) = 401];
  FORBIDDEN = 4 [(errors.code) = 403];
}
```

!> 你可以fork [reason-proto], 替换reason submodule, 按需增减, 但不建议增加太多错误码~


开启[auth服务](/started.0.init?id=auth%e6%9c%8d%e5%8a%a1)简单测试下

```bash
curl http://127.0.0.1:6060/info
# {"code":401, "reason":"UNAUTHORIZED", "message":"token is missing", "metadata":{}}
```

?> 其中  
`code=401`对应于`(errors.code) = 401`  
`reason=UNAUTHORIZED`对应于`UNAUTHORIZED = 3`  
`message=token is missing`是由应用内部自定义错误内容  


## 错误常量

为了便于管理, 默认将错误message定义在`internal/biz/reason.go`

```go
package biz

const (
	JwtMissingToken           = "jwt.token.missing"
	JwtTokenInvalid           = "jwt.token.invalid"
	JwtTokenExpired           = "jwt.token.expired"
	JwtTokenParseFail         = "jwt.token.parse.failed"
	JwtUnSupportSigningMethod = "jwt.wrong.signing.method"
	IdempotentMissingToken    = "idempotent.token.missing"
	IdempotentTokenExpired    = "idempotent.token.invalid"

	TooManyRequests    = "too.many.requests"
	DataNotChange      = "data.not.change"
	DuplicateField     = "duplicate.field"
	RecordNotFound     = "record.not.found"
	NoPermission       = "no.permission"
	IncorrectPassword  = "login.incorrect.password"
	SamePassword       = "login.same.password"
	InvalidCaptcha     = "login.invalid.captcha"
	LoginFailed        = "login.failed"
	UserLocked         = "login.user.locked"
	KeepLeastOntAction = "action.keep.least.one.action"
	DeleteYourself     = "user.delete.yourself"
)
```

?> 参见[auth reason](https://github.com/go-cinch/auth/blob/dev/internal/biz/reason.go#L1)

!> msg格式有一定规律, 因为做了国际化处理, 若不需要, 直接输入自定义内容


## 用法


`internal/biz`包下使用
```go
import "auth/api/reason" // auth是项目名

func (uc *UserUseCase) Info(ctx context.Context, code string) (rp *UserInfo, err error) {
	...
    err = reason.ErrorTooManyRequests(TooManyRequests)
	...
}
```

?> 参见[auth.Info](https://github.com/go-cinch/auth/blob/dev/internal/biz/user.go#L224)


`internal/data`包下使用
```go
import (
	"auth/api/reason" // auth是项目名
    "auth/internal/biz"
)

func (ro userRepo) GetByUsername(ctx context.Context, username string) (item *biz.User, err error) {
    ...
    err = reason.ErrorNotFound("%s User.username: %s", biz.RecordNotFound, username)
    ...
}
```

?> 参见[auth.GetByUsername](https://github.com/go-cinch/auth/blob/dev/internal/data/user.go#L56)


[reason-proto]: https://github.com/go-cinch/reason-proto

