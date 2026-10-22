# EP.12 异常过滤器

> 对应正文：[12. 异常过滤器](../docs/12-exception-filters/index.md) · 示例：[server/src/ch12-exception-filters](../server/src/ch12-exception-filters)

## 剧情回顾

- **第 1 格**：像素酱线上排查时发现：接口 500 直接把堆栈甩给了客户端。
- **第 2 格**：码叔搭起两层过滤器：HTTP 异常带业务码，其余一律兜底 500。
- **第 3 格**：标签君实测业务错误码：404 配 USER_NOT_FOUND，前端按码分支。
- **第 4 格**：码叔结案：业务码给前端，堆栈留服务端，永不混流。

## 知识点拆解

### 第 1 格 · 错误出口治理

默认异常过滤器对非 HttpException 一律 500 并可能带上内部信息。生产环境必须自定义：客户端只看安全文案，堆栈只进服务端日志。

### 第 2 格 · 分层捕获

```ts
@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {}

@Catch() // 全捕获，兜底
export class AllExceptionsFilter implements ExceptionFilter {}
```

最具体的过滤器优先匹配，两层过滤器同时注册互不冲突。

### 第 3 格 · 业务错误码体系

```json
{ "statusCode": 404, "code": "USER_NOT_FOUND",
  "message": "用户 999 不存在", "path": "/users/999" }
```

HTTP 状态码给网关与监控看，业务码（BizException 携带）给前端逻辑看：同一个 404 可能对应多种业务场景。

### 第 4 格 · 兜底纪律

兜底过滤器（@Catch()）必须 console.error 保留完整堆栈供排查，响应体里只给「服务器内部错误」加统一错误码。

## 坑点清单

1. 兜底过滤器不写日志，线上问题无从排查；
2. 把堆栈或原始数据库错误拼进响应体，属于信息泄露；
3. BizException 滥用默认 400——每个业务异常都应显式声明状态码。

## 自测 3 题（含答案）

1. @Catch(HttpException) 和 @Catch() 的区别？——**答案**：前者只捕获 HttpException 及其子类，后者捕获一切异常作为兜底。
2. 业务错误码存在的意义是什么？——**答案**：前端按稳定码值做分支处理，比解析报错文案可靠，且与 HTTP 状态码解耦。
3. 兜底过滤器里必须做什么？——**答案**：记录完整堆栈到服务端日志，同时只向客户端返回通用安全文案。
