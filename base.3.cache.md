# 缓存


biz层设置缓存分3个步骤即可完成

```shell
vim internal/biz/greeter.go
```


## get


读取数据库中的数据, 写入缓存
```go
func (uc *GreeterUseCase) get(ctx context.Context, action string, id uint64) (res string, ok bool) {
	// read data from db and write to cache
	rp := &Greeter{}
	item, err := uc.repo.Get(ctx, id)
	notFound := errors.Is(err, reason.ErrorNotFound(i18n.FromContext(ctx).T(RecordNotFound)))
	if err != nil && !notFound {
		return
	}
	copierx.Copy(&rp, item)
	res = utils.Struct2Json(rp)
	uc.cache.Set(ctx, action, res, notFound)
	ok = true
	return
}
```

?> Tips: 存入缓存的数据一般是json格式, 这里用到了utils.Struct2Json


## Get

定义当前action, 一般具有唯一性: 由方法名+参数组成  
调用cache.Get方法, 并设置回调函数

```go
func (uc *GreeterUseCase) Get(ctx context.Context, id uint64) (rp *Greeter, err error) {
	rp = &Greeter{}
	action := fmt.Sprintf("get_%d", id)
	str, ok := uc.cache.Get(ctx, action, func(ctx context.Context) (string, bool) {
		return uc.get(ctx, action, id)
	})
	if ok {
		utils.Json2Struct(&rp, str)
		if rp.Id == constant.UI0 {
			err = reason.ErrorNotFound("%s Greeter.id: %d", i18n.FromContext(ctx).T(RecordNotFound), id)
		}
		return
	}
	err = reason.ErrorTooManyRequests(i18n.FromContext(ctx).T(TooManyRequests))
	return
}
```

?> Tips: ok表示从缓存中读到数据, 反之可能因高并发请求导致没有获取到redis锁, 避免死锁, 设置了最大重试次数


## Flush


若数据发生变动, 直接使用Flush即可, 一般配合Tx一起使用

```go
func (uc *GreeterUseCase) Update(ctx context.Context, item *UpdateGreeter) error {
	return uc.tx.Tx(ctx, func(ctx context.Context) error {
		return uc.cache.Flush(ctx, func(ctx context.Context) (err error) {
			err = uc.repo.Update(ctx, item)
			return
		})
	})
}

func (uc *GreeterUseCase) Delete(ctx context.Context, ids ...uint64) error {
	return uc.tx.Tx(ctx, func(ctx context.Context) error {
		return uc.cache.Flush(ctx, func(ctx context.Context) (err error) {
			err = uc.repo.Delete(ctx, ids...)
			return
		})
	})
}
```

!> Flush会自动清除关联action下所有缓存, 若只想单个删除, 可以调用Del方法, 案例参见[auth.LastLogin](https://github.com/go-cinch/auth/blob/dev/internal/biz/user.go#L275)

?> 至此, 你的接口已自动防止缓存击穿、缓存穿透、缓存雪崩~  
不信的话自己用[locust](https://locust.io)测试~