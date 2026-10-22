# EP.04 模块系统

> 对应正文：[04. 模块系统](../docs/04-modules/index.md) · 示例：[server/src/ch04-modules](../server/src/ch04-modules)

## 剧情回顾

- **第 1 格**：标签君的根模块塞了两百多个提供者，编译一次十几秒。
- **第 2 格**：码叔讲解模块四属性：imports、controllers、providers、exports。
- **第 3 格**：像素酱实测跨模块注入：OrdersModule 导入 UsersModule 后注入 UsersService 成功。
- **第 4 格**：码叔结案：特性模块分域，动态模块传参，全局留给基础设施。

## 知识点拆解

### 第 1 格 · 按业务域拆分

所有提供者塞进根模块是典型反模式。正确做法是按业务域拆特性模块：用户域一个 `UsersModule`，订单域一个 `OrdersModule`，每域一个目录。

### 第 2 格 · 模块四属性

```ts
@Module({
  imports: [UsersModule],      // 引入别人的导出
  controllers: [OrdersController],
  providers: [OrdersService],  // 模块内注册
  exports: [OrdersService],    // 对其他模块可见
})
```

模块默认封装：`providers` 里的服务只能本模块注入，想让别的模块用必须 `exports`。

### 第 3 格 · 跨模块注入

`exports` 与 `imports` 成对出现：UsersModule 导出 UsersService，OrdersModule 导入 UsersModule，订单服务构造函数里就能注入 UsersService。示例中订单接口返回的用户数据正来自用户域。

### 第 4 格 · 动态模块与全局模块

- 动态模块：类上定义静态方法 `forRoot(options)` 返回 `DynamicModule`，把配置传进模块（如 `CacheModule.forRoot({ ttl: 60 })`）；
- 全局模块：`@Global()` 标记后只需在根模块导入一次，导出内容全站可见，仅限配置、日志这类基础设施。

## 坑点清单

1. 模块循环 import 会在启动期报错，先考虑抽公共模块；
2. `@Global()` 滥用会让依赖关系隐形化，依赖看构造函数就知情的优势没了；
3. 动态模块的配置令牌要记得 `exports`，否则消费方注入不到。

## 自测 3 题（含答案）

1. 模块默认是封装的还是共享的？——**答案**：封装的，providers 仅本模块可注入，必须 exports 才对其他模块可见。
2. 动态模块的应用场景是什么？——**答案**：需要接收配置参数的模块，如 ConfigModule.forRoot、TypeOrmModule.forRoot。
3. 全局模块应该怎么用？——**答案**：只给配置、日志、数据库连接等真正的基础设施用，在根模块导入一次即可。
