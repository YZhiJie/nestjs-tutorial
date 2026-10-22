# EP.02 最小可运行应用

> 对应正文：[02. 快速开始](../docs/02-quick-start/index.md) · 示例：[server/src/ch02-quick-start](../server/src/ch02-quick-start)

## 剧情回顾

- **第 1 格**：标签君想跑第一个接口，CLI 生成一大堆文件，反而看不清最小骨架。
- **第 2 格**：码叔给出启动四件套：入口、模块、控制器、服务。
- **第 3 格**：像素酱实测一次请求旅程：GET / 打到控制器，控制器转手调用服务。
- **第 4 格**：码叔结案：四个文件就是一个合法 Nest 应用。

## 知识点拆解

### 第 1 格 · 从 CLI 到最小骨架

`nest new` 生成的是完整工程（含 ESLint、Jest、Prettier），想看清骨架可以手动搭建——本教程示例项目就是这么做的。

### 第 2 格 · 启动四件套

入口文件用 `NestFactory.create(AppModule)` 创建应用并监听端口；根模块声明控制器与提供者；控制器用 `@Controller()` 声明路由；服务用 `@Injectable()` 标记为可注入。配套的 tsconfig 必须开启 `experimentalDecorators` 与 `emitDecoratorMetadata` 两个开关。

### 第 3 格 · 一次请求的旅程

```ts
@Get()
hello() {
  return this.appService.getHello(); // → 'Hello NestJS!'
}
```

控制器只做「取参、调用、返回」三段式中转，业务逻辑全部下沉到服务层。

### 第 4 格 · 运行与验证

示例项目执行 `npm run start:ch02` 后监听 3002 端口：`GET /` 返回文本 `Hello NestJS!`，`GET /health` 返回 `{"status":"ok"}`。

## 坑点清单

1. `import 'reflect-metadata'` 不在入口第一行，启动即报依赖解析错误；
2. 忘开 `emitDecoratorMetadata` 时构造函数注入拿到 undefined；
3. 端口被占用时用 `PORT` 环境变量切换，别改代码。

## 自测 3 题（含答案）

1. `NestFactory.create()` 的参数是什么？——**答案**：根模块（AppModule），它定义了应用的装配蓝图。
2. 最小 Nest 应用需要哪四个文件？——**答案**：main.ts 入口、app.module.ts 根模块、app.controller.ts 控制器、app.service.ts 服务。
3. 控制器返回字符串与返回对象有什么区别？——**答案**：返回对象自动 JSON 序列化；返回字符串按 text/html 发送。
