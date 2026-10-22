# EP.07 依赖注入进阶

> 对应正文：[07. 依赖注入](../docs/07-dependency-injection/index.md) · 示例：[server/src/ch07-di](../server/src/ch07-di)

## 剧情回顾

- **第 1 格**：标签君的 AlphaService 与 BetaService 互相引用，启动直接报循环依赖。
- **第 2 格**：码叔用 forwardRef 破环：惰性求值绕开先后问题，两侧成对添加。
- **第 3 格**：像素酱实测 @Optional 与 ModuleRef：可选依赖优雅降级，运行时按需解析。
- **第 4 格**：码叔结案：循环依赖是设计信号，重构优于 forwardRef。

## 知识点拆解

### 第 1 格 · 循环依赖怎么来的

A 模块导入 B、B 又导入 A（或两个服务互相注入），容器无法决定先实例化谁，启动期抛出 circular dependencies 错误。

### 第 2 格 · forwardRef 破环

```ts
@Inject(forwardRef(() => BetaService))
private readonly beta: BetaService
```

把引用包成惰性求值的函数，推迟到真正使用时解析。**模块级与提供者级必须两侧成对出现**，只加一侧仍然报错。

### 第 3 格 · 可选依赖与运行时解析

`@Optional()` 标记的依赖未注册时注入 undefined 而不是启动失败，适合插件式可选能力。`ModuleRef`（从 `@nestjs/core` 导入）是容器的编程式入口：`moduleRef.get(Token)` 在运行时按需解析提供者。

### 第 4 格 · 使用边界

ModuleRef.get 默认 `strict: true`，只能取当前模块内注册的提供者；跨模块要传 `{ strict: false }`。它是服务定位器模式，优先级永远低于构造函数注入。

## 坑点清单

1. forwardRef 只加一侧仍然报错，模块级、提供者级都要成对；
2. ModuleRef 从 `@nestjs/common` 导入是常见笔误，正确来源是 `@nestjs/core`；
3. 循环依赖多数时候该重构：把双方都依赖的部分抽成第三个模块。

## 自测 3 题（含答案）

1. forwardRef 的原理是什么？——**答案**：把直接引用包成惰性求值函数，推迟到运行时真正使用时才解析，绕开实例化先后问题。
2. `@Optional()` 的行为是什么？——**答案**：依赖未注册时注入 undefined，应用照常启动。
3. ModuleRef.get 默认的解析范围是什么？——**答案**：当前模块（strict 模式），跨模块需显式传 strict: false。
