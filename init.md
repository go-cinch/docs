# 环境准备


启动项目前, 我默认你已经准备好下列软件: 
- [go](https://golang.org/dl/)
- [protoc](https://github.com/protocolbuffers/protobuf)
- [protoc-gen-go](https://github.com/protocolbuffers/protobuf-go)
- [git](https://git-scm.com)
- [kratos cli工具](https://go-kratos.dev/docs/getting-started/usage)
    ```
    go install github.com/go-kratos/kratos/cmd/kratos/v2@latest
    ```


# Auth服务


权限认证服务无需再开发, 下载开箱即用

```
git clone https://github.com/go-cinch/auth
# 可以指定tag
# git clone -b v1.0.0 https://github.com/go-cinch/auth
```


# 创建Game服务


```
# 1.通过模板创建项目 -r 指定仓库 -b 指定分支
kratos new game -r https://github.com/go-cinch/layout.git -b dev

# 2. 进入项目
cd game
git init
# 一般的我们建议以dev作为开发分支
git checkout -b dev

# 3. 初始化submodule
rm -rf api/auth-proto
# -b指定分支 --name指定submodule名称
git submodule add -b master --name api/auth-proto https://github.com/go-cinch/auth-proto.git ./api/auth-proto

rm -rf api/reason-proto
# -b指定分支 --name指定submodule名称
git submodule add -b master --name api/reason-proto https://github.com/go-cinch/reason-proto.git ./api/reason-proto

# 这里用greeter作为示例
rm -rf api/greeter-proto
# -b指定分支 --name指定submodule名称
git submodule add -b master --name api/greeter-proto https://github.com/go-cinch/greeter-proto.git ./api/greeter-proto

# 4. 下载依赖
go mod download

# 5. 编译项目
make all
```


# 启动


## 配置文件


```
# 修改auth项目配置以及game项目配置
cd auth/configs
# 将mysql/redis的配置修改成你本地配置
vim conifg.yaml

# 修改game项目配置
cd game/configs
# 将mysql/redis的配置修改成你本地配置
vim conifg.yaml
# 将auth服务host和端口修改成你本地配置
vim client.yaml

# 启动auth
cd auth
kratos run

# 启动game
cd game
kratos run
```


## 环境变量


```
# 启动auth
cd auth
export AUTH_DATA_DATABASE_DSN=root:root@tcp(127.0.0.1:3306)/auth?parseTime=True
export AUTH_DATA_REDIS_DSN=redis://127.0.0.1:6379/0
kratos run

# 启动game
cd game
export CINCH_DATA_DATABASE_DSN=root:root@tcp(127.0.0.1:3306)/game?parseTime=True
export CINCH_DATA_REDIS_DSN=redis://127.0.0.1:6379/0
export CINCH_CLIENT_AUTH=127.0.0.1:6160
kratos run
```

?> Tips: 环境变量前缀可在cmd/xxx/main.go中修改


## 测试访问


auth服务: 
```
curl http://127.0.0.1:6060/idempotent
# 输出如下说明服务通了只是没有权限, 出现其他说明配置有误
# {"code":401, "reason":"UNAUTHORIZED", "message":"token is missing", "metadata":{}}
```

game服务: 
```
curl http://127.0.0.1:8080/greeter
# 输出如下说明服务通了只是没有权限, 出现其他说明配置有误
# {"code":401, "reason":"UNAUTHORIZED", "message":"token is missing", "metadata":{}}
```

?> 至此, 微服务已启动完毕, auth以及game, 接下来自定义你的game啦~
