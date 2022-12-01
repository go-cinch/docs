# 配置


## 所在目录


```
├── configs              // 配置文件目录
│   ├── config.yaml      // 主配置文件
│   ├── client.yaml      // 配置grpc服务client, 如auth
│   └── ...              // 其他自定义配置文件以yaml结尾
├── internal
│   ├── conf             // 内部使用的config的结构定义, 使用proto格式生成
│   │   └── conf.proto
```


## configs


当前目录下可包含1或多个yaml文件, 并且支持热更新, 参考[Kratos Config](https://go-kratos.dev/docs/component/config)


### config.yaml


在kratos官方配置文件增加的额外字段用法:

- `server.machineId`  
  机器编号或应用编号, 系统内置[sonyflake](http://github.com/sony/sonyflake)雪花id生成算法, 若是多个节点部署最好设置不一样的id, 大于0即可
- `server.language`  
  默认语言, 若请求Accept-Language未指定, 自动使用该默认语言, 内置i18n多语言管理, 目前支持中文和英文(zh/en), 其他语言自行扩展
- `data.redis.dsn`  
  redis服务地址, 系统中很多组件依赖redis, 属于强依赖, 不配置会导致某些功能无法使用, 配置规则参见[asynq.ParseRedisURI](https://github.com/hibiken/asynq/blob/master/asynq.go#L436)
- `tracer.enable`  
  开启或关闭链路追踪, 系统内置[OpenTelemetry](https://github.com/open-telemetry/opentelemetry-go)链路追踪, 若关闭, log中的`trace.id`/`span.id`字段将不会显示
- `tracer.otlp.endpoint`  
  链路追踪otlp服务地址, `tracer.enable`=`true`才会生效, 可以试试[SigNoz](https://github.com/SigNoz/signoz/tree/develop/deploy#using-docker-compose)部署一个本地服务, 若不方便部署, 设置为空字符串`''`, 会打印到控制台
- `tasks[i].category`  
  定时(异步)常驻任务分组名称, 方便查看同一组数据
- `tasks[i].uuid`  
  定时(异步)常驻任务唯一编号, 若出现相同的编号, 只会有1个任务注册成功
- `tasks[i].expr`  
  定时(异步)常驻任务执行表达式, 最小单位1分钟


### client.yaml


主要用于定义外部微服务连接地址, 如auth/order/...服务
```yml
client:
  auth: '${CLIENT_AUTH:auth:6160}'
  order: '${CLIENT_AUTH:order:7160}'
  ...: '...'
```

?> 你也可以定义到`config.yaml`中, 但微服务过多会显得比较臃肿


## 变更


增加或删除配置

### conf.proto


```shell
vim internal/conf/conf.proto
```


### 编译


```shell
make config
```

执行后, `internal/conf/conf.pb.go`会发生变化


### configs


修改configs下的配置文件字段与`internal/conf/conf.proto`一一对应即可


### conf.Bootstrap


包含`conf.Bootstrap`变量都能使用配置文件中的字段, 一般只用于初始化
- `internal/data/data.go` - 用于初始化MySQL/Redis等
- `internal/biz/greeter.go` - 用于初始化UseCase, 当然不仅限于greeter


## 项目名


项目名称默认值`cinch-layout`

```shell
vim cmd/game/main.go
# 修改为你想要的即可, 不建议为空
# Name = "game"
```


## 环境变量


### 开启环境变量


在configs/xxx.yaml对应字段写入`${环境变量名:默认值}`
即可使用`环境变量前缀+环境变量名`来覆盖配置文件

!> bool变量不支持该方式, 若需要, 请将bool类型改为string

### 环境变量前缀


前缀默认值`CINCH_`

```shell
vim cmd/game/main.go
# 修改为你想要的即可, 建议全部大写以下划线结尾`_`, 不建议为空
# EnvPrefix = "GAME_"
```

修改后使用环境变量覆盖MySQL的dsn
```shell
cd game
export GAME_DATA_DATABASE_DSN=root:root@tcp(127.0.0.1:3306)/game?parseTime=True
kratos run
```
