# EP.13 TypeORM 数据库

> 对应正文：[13. 数据库集成](../docs/13-database/index.md) · 示例：[server/src/ch13-database](../server/src/ch13-database)

## 剧情回顾

- **第 1 格**：标签君只想跑个数据库示例，却被「先装 MySQL、再建账号」劝退。
- **第 2 格**：码叔讲解实体即表：装饰器就是建表语句，关系也用装饰器描述。
- **第 3 格**：像素酱实测 Repository：注入仓库直接 CRUD，业务代码不写 SQL。
- **第 4 格**：码叔结案：forRoot 换连接参数即可切 MySQL，业务零改动。

## 知识点拆解

### 第 1 格 · 零依赖起步

示例项目用 sql.js（WebAssembly 版 SQLite，纯 JS 内存库）演示，无需安装任何数据库。业务代码与生产一致，切库只改连接参数。

### 第 2 格 · 实体即表

```ts
@Entity('users')
export class User {
  @PrimaryGeneratedColumn() id: number;
  @Column({ unique: true }) email: string;
  @OneToMany(() => Post, (p) => p.author)
  posts: Post[];
}
```

`@Entity` 定表名，`@Column({ unique: true })` 加唯一约束，`@OneToMany` 与 `@ManyToOne` 描述一对多双向关系。

### 第 3 格 · Repository 模式

`@InjectRepository(User)` 注入实体仓库，`repo.find()`、`repo.save()`、`repo.delete()` 覆盖常见操作；`relations: ['posts']` 一个参数把关联数据 JOIN 带回。`create()` 只在内存构造实体，`save()` 才落库。

### 第 4 格 · 切换生产数据库

安装 `mysql2` 或 `pg` 驱动，修改 `TypeOrmModule.forRoot` 连接参数（type、host、port、username、password、database），业务代码零改动。

## 坑点清单

1. `synchronize: true` 会按实体自动改表结构，仅限开发，生产必须关闭并改用 migrations；
2. 唯一约束冲突会裸抛 QueryFailedError，应捕获后转成 409 ConflictException；
3. 数据库错误码直接透出客户端属于信息泄露，服务层要做翻译。

## 自测 3 题（含答案）

1. create 和 save 的区别是什么？——**答案**：create 在内存构造实体实例，save 执行 INSERT 或 UPDATE 落库。
2. 为什么生产必须关闭 synchronize？——**答案**：它按实体直接修改表结构，可能造成数据丢失，生产应使用 migrations 管理变更。
3. 唯一约束冲突的正确处理方式？——**答案**：在服务层捕获 QueryFailedError，翻译成 ConflictException（409）返回统一错误结构。
