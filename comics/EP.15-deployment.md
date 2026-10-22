# EP.15 生产部署

> 对应正文：[15. 部署](../docs/15-deployment/index.md) · 示例：[server/src/ch15-deployment](../server/src/ch15-deployment)

## 剧情回顾

- **第 1 格**：标签君把端口环境变量写成 8o15，启动不报错，流量高峰才炸。
- **第 2 格**：码叔上启动期校验：ConfigModule validate 非法配置直接拒启。
- **第 3 格**：像素酱实测优雅关闭：SIGTERM 后先停接新、处理存量，再退出。
- **第 4 格**：码叔结案：校验前置、优雅关闭、瘦镜像，上线三件套。

## 知识点拆解

### 第 1 格 · fail fast

配置错误拖到运行期暴露，代价是半夜的事故。让非法配置在启动第一秒就把进程拦下来。

### 第 2 格 · 启动期校验

```ts
ConfigModule.forRoot({
  validate: (env) => parseConfig(env),
})
```

validate 函数把环境变量解析成带类型的配置对象，NODE_ENV 非法、端口非数字直接抛错拒绝启动。

### 第 3 格 · 优雅关闭

`app.enableShutdownHooks()` 让框架监听 SIGTERM 与 SIGINT：停止接收新连接，等待在途请求完成并执行 `onModuleDestroy` 释放连接池，最后退出进程。K8s 滚动发布不断流全靠它。

### 第 4 格 · 多阶段 Docker 构建

第一阶段安装全部依赖并编译 TS；第二阶段只拷贝 dist 与生产依赖（`npm ci --omit=dev`）。镜像更小、攻击面更窄，配套 .dockerignore 排除 node_modules 与 .git。

## 坑点清单

1. 生产镜像带着 devDependencies，体积与漏洞面双倍膨胀；
2. `kill -9` 绕过所有信号处理，无法优雅关闭——编排系统默认发 SIGTERM 才有救；
3. 健康检查里查数据库，DB 抖动会误杀所有实例，探针只报进程状态。

## 自测 3 题（含答案）

1. enableShutdownHooks 的作用是什么？——**答案**：监听 SIGTERM 与 SIGINT，执行优雅关闭：停接新请求、等待存量、执行 onModuleDestroy 后退出。
2. 为什么配置校验要在启动期做？——**答案**：fail fast——非法配置第一秒暴露，而不是运行到流量高峰才炸。
3. 多阶段构建省掉了什么？——**答案**：运行时镜像不需要 TypeScript 编译器与开发依赖，只保留 dist 产物与生产依赖。
