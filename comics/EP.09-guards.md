# EP.09 守卫与鉴权

> 对应正文：[09. 守卫](../docs/09-guards/index.md) · 示例：[server/src/ch09-guards](../server/src/ch09-guards)

## 剧情回顾

- **第 1 格**：像素酱忘了挂守卫，内部 admin 接口全网可访问，被安全扫描点名。
- **第 2 格**：码叔讲解 CanActivate：返回 false 或抛异常，请求就进不了控制器。
- **第 3 格**：标签君搭出全局守卫链：APP_GUARD 按注册顺序先认证后授权。
- **第 4 格**：码叔结案：认证看你是谁，授权看你能干啥。

## 知识点拆解

### 第 1 格 · 默认全保护

逐个接口加 `@UseGuards` 总有漏网之鱼。正确姿势：APP_GUARD 全局注册认证守卫，公开接口用 `@Public()` 显式放行。

### 第 2 格 · 守卫的核心接口

```ts
canActivate(context: ExecutionContext): boolean {
  const request = context.switchToHttp().getRequest();
  if (valid) { request.user = parsed; return true; }
  throw new UnauthorizedException();
}
```

守卫能通过 ExecutionContext 知道路由类与方法，配合 `Reflector.getAllAndOverride()` 读取 `@Public()`、`@Roles()` 等元数据。

### 第 3 格 · 全局守卫链

```ts
{ provide: APP_GUARD, useClass: AuthGuard }  // 先执行
{ provide: APP_GUARD, useClass: RolesGuard } // 后执行
```

注册顺序即执行顺序。守卫之间通过 `request.user` 传递数据：认证守卫解析用户挂载上去，角色守卫只读不解析。

### 第 4 格 · 元数据驱动

`@Public()` 与 `@Roles('admin')` 本质都是 SetMetadata。角色守卫读取路由所需角色，与用户角色比对，不满足抛 403。

## 坑点清单

1. `getAllAndOverride` 路由级优先、类级兜底；与 `get` 语义不同别混用；
2. 给 `request.user` 挂数据时记得扩展 Express.Request 类型声明；
3. 守卫只管「放行与否」，参数校验是管道的活，别越权。

## 自测 3 题（含答案）

1. 守卫返回 false 和抛 UnauthorizedException 的区别是什么？——**答案**：前者默认 403，后者按异常携带的状态码（401）响应，语义更准确。
2. 全局守卫的注册顺序意味着什么？——**答案**：APP_GUARD 的注册顺序即执行顺序，应先认证后授权。
3. @Public() 是怎么生效的？——**答案**：SetMetadata 写入路由元数据，认证守卫用 Reflector 读取并跳过校验。
