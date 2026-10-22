# EP.08 中间件

> 对应正文：[08. 中间件](../docs/08-middleware/index.md) · 示例：[server/src/ch08-middleware](../server/src/ch08-middleware)

## 剧情回顾

- **第 1 格**：标签君在 finish 回调里补写响应头，直接抛出 ERR_HTTP_HEADERS_SENT。
- **第 2 格**：码叔讲解三种挂载：全局函数、类中间件、consumer 绑定。
- **第 3 格**：像素酱实测 forRoutes 与 exclude：按控制器绑定，精准豁免。
- **第 4 格**：码叔结案：日志追踪归中间件，鉴权请交给守卫。

## 知识点拆解

### 第 1 格 · 头不可二次发送

响应一旦发出（res.end 被调用），头部就固化了。要在响应头里补耗时信息，必须抢在写出之前——示例的做法是包装 `res.end`，在真正写出前把耗时头写进去。

### 第 2 格 · 三种挂载方式

```ts
app.use(fn)                                  // 全局，对所有路由生效
class Logger implements NestMiddleware {}    // 类，可注入依赖
consumer.apply(Logger, Time).forRoutes(C)    // 按控制器绑定
```

类中间件实现 `NestMiddleware` 接口，能用构造函数注入服务；模块实现 `NestModule.configure()` 完成绑定。

### 第 3 格 · 绑定与豁免

`.forRoutes(UsersController)` 把中间件限定到指定控制器；`.exclude({ path: 'users/x', method: RequestMethod.GET })` 精准豁免某些路由。多个中间件按声明顺序串行执行。

### 第 4 格 · 职责定位

中间件是管线第一站：日志、请求 ID、耗时统计、CORS 都合适。它拿不到路由元数据——需要知道命中哪个控制器的场景（鉴权）用守卫。

## 坑点清单

1. 忘调 `next()` 请求永远挂起，除非中间件主动结束响应；
2. 中间件里读不到路由处理器元数据，别在中间件里做鉴权；
3. 包装 `res.end` 时忘记 `bind(this)` 或丢失原参数，直接把请求搞崩。

## 自测 3 题（含答案）

1. ERR_HTTP_HEADERS_SENT 是怎么触发的？——**答案**：响应发出后再修改响应头，例如在 finish 事件回调里 setHeader。
2. 类中间件相比函数中间件的优势是什么？——**答案**：可以通过构造函数注入其他提供者。
3. 中间件里能不能读取路由装饰器元数据？——**答案**：不能，此时请求尚未匹配到控制器，需要元数据的场景应使用守卫或拦截器。
