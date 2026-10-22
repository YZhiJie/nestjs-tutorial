# EP.06 服务与提供者

> 对应正文：[06. 服务与提供者](../docs/06-providers/index.md) · 示例：[server/src/ch06-providers](../server/src/ch06-providers)

## 剧情回顾

- **第 1 格**：标签君说不清服务到底有几份实例，担心并发埋雷。
- **第 2 格**：码叔讲解四种注册方式：useClass、useValue、useFactory、useExisting。
- **第 3 格**：像素酱实测三种作用域：单例处处相同，瞬态每个消费者一份，请求级每请求重建。
- **第 4 格**：码叔结案：初始化放 onModuleInit，构造函数只做赋值。

## 知识点拆解

### 第 1 格 · 作用域决定生命周期

`@Injectable()` 默认单例（`Scope.DEFAULT`）。Nest 还提供 `Scope.TRANSIENT`（每个消费者类获得专属实例）与 `Scope.REQUEST`（每个 HTTP 请求新建，可注入 REQUEST 对象）。

### 第 2 格 · 四种注册方式

```ts
{ provide: CONFIG, useValue: { appName: 'demo' } }
{ provide: CONNECTION, useFactory: (cfg, log) => connect(cfg), inject: [CONFIG, Logger] }
{ provide: LEGACY, useExisting: Logger }
```

非类提供者必须用字符串或 Symbol 令牌。useFactory 适合需要运行时计算的实例；useExisting 起别名共享实例。

### 第 3 格 · 作用域对照实验

两个单例消费者各自注入同一个单例，得到相同实例；注入瞬态服务则拿到不同实例。注意：瞬态的真实语义是「每个消费者类一个实例」，同类内多个注入点会去重共享。

### 第 4 格 · 生命周期钩子

常用钩子三件套：`onModuleInit`（依赖就绪后）、`onApplicationBootstrap`（应用启动完成后）、`onModuleDestroy`（应用关闭前）。异步初始化必须放钩子里，构造函数里做异步没人 await 得到。

## 坑点清单

1. 构造函数里连数据库——异步初始化无法被等待，应放 onModuleInit；
2. REQUEST 作用域会级联传染宿主（控制器、服务链全部变请求级），性能敏感场景慎用；
3. useFactory 的 inject 数组顺序必须与工厂函数参数一一对应。

## 自测 3 题（含答案）

1. 默认作用域下两个消费者注入同一服务，拿到几份实例？——**答案**：一份，单例共享。
2. 瞬态作用域的准确语义是什么？——**答案**：每个消费者类一个专属实例，同类多注入点去重共享。
3. 字符串令牌注册的服务怎么注入？——**答案**：构造函数里 `@Inject(令牌)` 装饰参数。
