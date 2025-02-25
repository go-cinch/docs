# 更新日志

下面以版本号逆序方式排列更新日志

## TODO

- [ ] 增加E2E测试用例

?> 如果有好的Idea, 请提PR或群内讨论, 合适的话可以考虑加入TODO列表

!> 不一定下个版本就会完成TODO, 结合实际需要和空闲程度而定, 客官不要着急~

## v1.2.0

- 包升级, go v1.23, kratos v2.8.3, go-redis v9.7.0
- 使用bsm/redislock替换原来的分布式锁
- 更强大的工具库samber/lo
- 幂等性token从原来server端生成替换为mobile端生成
- 日志接入logrus替换kratos默认log
- 修复worker的一些问题
- 修复已知bug, 同步最新layout
- 完善docs文档

## v1.1.0

- 包升级, go v1.22, kratos v2.7.0, go-redis v9.2.1
- 缓存预热, 部分接口直接读redis, 提高并发访问性能
- 修复已知bug, 同步最新layout
- 集成Minio对象存储
- 增加Grafana监控系统

## v1.0.3

- 包升级, go v1.20, kratos v2.6.2, go-redis v9.0.5, gorm v1.25.2
- cinch增加gen命令, 包括gen gorm, gen migrate, 结合cinch更容易写CRUD
- 增加数据库层面的多tenant, 深度集成gorm
- 修复在nignx auth_request模式下权限校验的一些问题
- 增加白名单, 动态查询jwt/idempotent/permission, 不写死在代码
- 移除一些不当的用法, 如布隆过滤器

## v1.0.0 ~ v1.0.2

项目初始化以及kratos的学习