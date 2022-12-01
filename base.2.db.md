# 数据库


## 事务


data层已经做好事务的封装, 在biz业务层直接使用即可

```shell
vim internal/biz/greeter.go
```

```go
func (uc *GreeterUseCase) Create(ctx context.Context, item *Greeter) error {
	return uc.tx.Tx(ctx, func(ctx context.Context) error {
		return uc.repo.Create(ctx, item)
	})
}
```

?> 若uc.repo.Create抛出异常, 事务自动回滚, 否则自动提交


## DB


获取gorm.DB实例

```shell
vim internal/data/greeter.go
```

```go
func (ro greeterRepo) Create(ctx context.Context, item *biz.Greeter) (err error) {
	...
	db := ro.data.DB(ctx)
	return
}
```

!> 若biz层使用Tx, db将包含事务, 否则无事务


## 雪花Id


系统内置[sonyflake](http://github.com/sony/sonyflake)雪花id生成算法

```shell
vim internal/data/greeter.go
```

在Create(创建数据)前使用ro.data.Id生成
```go
func (ro greeterRepo) Create(ctx context.Context, item *biz.Greeter) (err error) {
	...
	db := ro.data.DB(ctx)
	m.Id = ro.data.Id(ctx)
	err = db.Create(&m).Error
	return
}
```
