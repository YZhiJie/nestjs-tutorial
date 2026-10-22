# EP.11 管道与校验

> 对应正文：[11. 管道](../docs/11-pipes/index.md) · 示例：[server/src/ch11-pipes](../server/src/ch11-pipes)

## 剧情回顾

- **第 1 格**：标签君收到安全告警：前端多传的 hacker 字段被原样写进了数据库。
- **第 2 格**：码叔讲解管道两件事：转换与校验，不过关直接 400。
- **第 3 格**：像素酱实测 ValidationPipe：DTO 装饰器即规则，多余字段直接拒绝。
- **第 4 格**：码叔结案：校验在边界完成，错误信息逐字段返回。

## 知识点拆解

### 第 1 格 · 信任边界

客户端传来的任何数据都不可信。没有校验层，多余字段、非法类型、越界数值全部直通数据库。校验必须发生在业务逻辑之前。

### 第 2 格 · 管道的两种职责

```ts
@Query('page', new DefaultValuePipe(1), ParseIntPipe)
page: number
```

管道可串联：先补默认值再转数字。自定义管道实现 `PipeTransform` 接口，transform 抛出异常即校验失败。

### 第 3 格 · ValidationPipe 全家桶

- `whitelist: true`：剥离 DTO 未声明的属性；
- `forbidNonWhitelisted: true`：出现未声明字段直接 400（比静默剥离更严格）；
- `transform: true`：按 DTO 声明自动转型。

错误信息按字段聚合成数组返回，前端可直接逐字段展示。

### 第 4 格 · DTO 设计

规则用 class-validator 装饰器声明在类上：`@IsString()`、`@MinLength(2)`、`@Max(150)`。可选项必须加 `@IsOptional()`。

## 坑点清单

1. `@IsOptional()` 放错位置，undefined 也被后续规则校验；
2. class-validator 默认英文报错，面向用户务必覆写 message；
3. 全局 ValidationPipe 与参数级管道混用时，先想清楚各自职责。

## 自测 3 题（含答案）

1. forbidNonWhitelisted 与 whitelist 的区别？——**答案**：whitelist 静默剥离多余字段，forbidNonWhitelisted 遇到多余字段直接 400。
2. 管道串联的执行规则是什么？——**答案**：按声明顺序执行，前一个管道的输出是后一个的输入。
3. 自定义管道实现哪个接口？——**答案**：PipeTransform，transform 方法返回值替换原参数，抛异常则校验失败。
