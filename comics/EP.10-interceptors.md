# EP.10 拦截器与 AOP

> 对应正文：[10. 拦截器](../docs/10-interceptors/index.md) · 示例：[server/src/ch10-interceptors](../server/src/ch10-interceptors)

## 剧情回顾

- **第 1 格**：标签君面对三种响应格式并存：裸数组、code 壳、msg 壳，前端解析写满 if。
- **第 2 格**：码叔揭示 RxJS 本质：handle() 返回流，map 一下全站响应变样。
- **第 3 格**：像素酱实测超时拦截器：慢接口 4 秒才回，拦截器 3 秒就掐，客户端拿到 408。
- **第 4 格**：码叔结案：日志、包装、超时、缓存都是拦截器的主场。

## 知识点拆解

### 第 1 格 · 响应格式统一

响应格式不统一的根源是每个接口各自包装。统一结构应该在出口处一次性解决——这正是拦截器的 AOP 价值：业务代码零感知。

### 第 2 格 · RxJS 流式处理

```ts
intercept(ctx, next) {
  return next.handle().pipe(
    map(data => ({ code: 0, data, timestamp: Date.now() })),
  );
}
```

`next.handle()` 返回 Observable，处理器的返回值以流的形式经过操作符。响应尚未发出，此刻包装是安全的。

### 第 3 格 · 超时控制

`timeout(3000)` 在 3 秒后让流转错误通道，`catchError` 把它转成 RequestTimeoutException。客户端 3 秒收到 408，不再傻等处理器完成。阈值应做成可注入参数（useFactory 传入）。

### 第 4 格 · 全局与局部

`APP_INTERCEPTOR` 注册全局拦截器，注册顺序即链上顺序；`@UseInterceptors()` 在控制器或路由级追加。后置逻辑按洋葱模型逆序返回。

## 坑点清单

1. 处理器抛出的异常不经过 map，直接进异常过滤器——错误处理用 catchError；
2. 多个拦截器的前置按注册顺序、后置逆序执行，包装顺序要想清楚；
3. 超时阈值写死在代码里，生产环境没法调整。

## 自测 3 题（含答案）

1. 拦截器为什么能修改响应？——**答案**：next.handle() 返回的 Observable 流经 map 操作符时可以替换响应值，此时响应尚未发送。
2. 超时拦截器由哪两个操作符组成？——**答案**：timeout 控制时限，catchError 把超时错误转成 RequestTimeoutException（408）。
3. 全局拦截器与路由级拦截器的执行关系？——**答案**：全局在前路由级在后，前置按此顺序进入，后置逆序返回（洋葱模型）。
