# 第一个接口


## 增加接口

为greeter增加一个接口: 创建一个游戏
```
cd api/greeter-proto
vim greeter.proto
```

在service Greeter中增加以下内容(这里省略CreateGameRequest内容, 参考CreateGreeterRequest)
```protobuf
  rpc CreateGame (CreateGameRequest) returns (google.protobuf.Empty) {
    option (google.api.http) = {
      post: "/game"
      body: "*"
    };
  }
```

## 编译api

```
# 进入项目根目录
cd game
make api
# api/greeter目录下的文件会发生变化
```

## 实现业务逻辑

```
cd internal/service
vim greeter.go
```

增加以下内容
```go
func (s *GreeterService) CreateGame(ctx context.Context, req *greeter.CreateGameRequest) (rp *emptypb.Empty, err error) {
	tr := otel.Tracer("api")
	ctx, span := tr.Start(ctx, "CreateGame")
	defer span.End()
	log.WithContext(ctx).Info("create game success")
	return
}
```

## 关闭鉴权

为了测试方便, 暂时关闭鉴权
```
cd internal/server/middleware
vim whitelist.go
```

增加以下内容
```go
	whitelist[greeter.OperationGreeterCreateGame] = struct{}{}
```

?> Tips: 上述代码表示将create game这个接口加入白名单, 跳过鉴权


## 重启game服务

重启步骤参见[启动](/init?id=启动)


## 测试访问

```
curl -H "Content-Type: application/json" -X POST http://127.0.0.1:8080/game
# 输出如下说明服务通了只是没有权限, 出现其他说明配置有误
# {}
# 且日志会打印出create game success
```

?> 至此, 一个简单的接口就创建完成啦~
