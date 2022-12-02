# Make命令


项目根目录内置Makefile, 常用于配置、proto更新等  
本地开发阶段, 常用`sub`/`all`就好


## sub


更新git submodule

```shell
make sub
```


## api


更新`api/xxx-proto`文件, 生成到`api/xxx`目录, 并生成swagger json文档到`docs/xxx-proto`

```shell
make api
```


## config


更新`internal/conf/conf.proto`

```shell
make config
```


## generate


编译wire依赖注入

```shell
make generate
```


## all


合并执行api/config/generate命令

```shell
make all
```


## init


下载一些golang基础工具

```shell
make init
```


## build


编译应用

```shell
make build
```


## help


输出帮助信息

```shell
make help
```

