# 第一个接口


## 增加接口

为greeter增加一个接口: 创建一个游戏
```
vim api/greeter-proto/greeter.proto
```

在service Greeter中增加以下内容
接口名、请求参数名、响应参数名、是否开启http接口(参考CreateGreeter): 
```protobuf
  rpc CreateGame (CreateGameRequest) returns (google.protobuf.Empty) {
    option (google.api.http) = {
      post: "/game"
      body: "*"
    };
  }
```

新增message CreateGameRequest
请求参数字段, 这里简单演示创建游戏名(create接口不需要响应参数字段)
```protobuf
message CreateGameRequest {
  string name = 1;
}
```

!> 这里为了演示方便, 直接使用greeter-proto, 正常情况下你应该新建一个game-proto管理game服务的对外接口.

## 编译api

```
# 进入项目根目录
cd game
make api
# api/greeter目录下的文件会发生变化
```

## 实现业务逻辑

```
vim internal/service/greeter.go
```

增加以下内容
```go
import "github.com/go-cinch/common/log"

func (s *GreeterService) CreateGame(ctx context.Context, req *greeter.CreateGameRequest) (rp *emptypb.Empty, err error) {
	tr := otel.Tracer("api")
	ctx, span := tr.Start(ctx, "CreateGame")
	defer span.End()
	log.WithContext(ctx).Info("create game: %s success", req.Name)
	return
}
```

!> common/log基于kratos logger封装一层, 内部适配链路追踪以及gorm SQL语句打印, 若使用其他log可能导致显示不正常

## 关闭鉴权

为了测试方便, 暂时关闭鉴权
```
vim internal/server/middleware/whitelist.go
```

增加以下内容
```go
	whitelist[greeter.OperationGreeterCreateGame] = struct{}{}
```

?> Tips: 上述代码表示将create game这个接口加入白名单, 跳过鉴权


## 重启game服务

重启步骤参见[启动](/started.0.init?id=启动)


## 测试访问

```
curl -H "Content-Type: application/json" -X POST http://127.0.0.1:8080/game
# 输出如下说明服务通了, 出现其他说明配置有误
# {}
# 且日志会打印出create game success
```

?> 至此, 一个简单的接口就创建完成啦~
