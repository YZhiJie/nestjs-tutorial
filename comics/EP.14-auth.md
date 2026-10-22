# EP.14 JWT 认证授权

> 对应正文：[14. 认证与授权](../docs/14-auth/index.md) · 示例：[server/src/ch14-auth](../server/src/ch14-auth)

## 剧情回顾

- **第 1 格**：像素酱在演示库里发现明文密码列，拖库即全裸。
- **第 2 格**：码叔讲解 scrypt 加盐散列与恒定时间比较。
- **第 3 格**：标签君走完 JWT 往返：登录发令牌，后续请求带 Bearer Token。
- **第 4 格**：码叔结案：短期令牌加角色守卫，密钥只走环境变量。

## 知识点拆解

### 第 1 格 · 口令安全底线

口令绝不明文存储。存储格式为 `salt:hash`：随机盐 + scrypt 散列。校验时用 `timingSafeEqual` 恒定时间比较，防时序侧信道攻击。

### 第 2 格 · 散列实现

```ts
const salt = randomBytes(16).toString('hex');
const hash = scryptSync(plain, salt, 64).toString('hex');
return `${salt}:${hash}`;
```

全程使用 Node 内置 crypto，无需原生依赖。

### 第 3 格 · JWT 往返

登录成功返回 `accessToken`（含 sub、username、role）。后续请求带 `Authorization: Bearer <token>`，JwtAuthGuard 调 `verifyAsync` 校验签名与过期时间，payload 挂到 `request.user`。

### 第 4 格 · 守卫链装配

JwtModule 注册全局 JwtService；全局守卫链先 JWT 认证再角色授权，与第 9 章的守卫模型完全一致，只是令牌从静态表升级为签名 JWT。

## 坑点清单

1. JWT 密钥写死进代码等于公开签名权，必须走 JWT_SECRET 环境变量；
2. payload 可被任何人 base64 解码（签名不加密），绝不放敏感信息；
3. 登录失败统一报「用户名或密码错误」，防止账号枚举。

## 自测 3 题（含答案）

1. 为什么用 timingSafeEqual 比较散列？——**答案**：普通比较在首个不匹配字节即返回，耗时差异可被用于逐字节猜测口令（时序攻击）。
2. JWT 的 payload 能放敏感信息吗？——**答案**：不能，JWT 只是签名未加密，任何人可解码 payload。
3. 令牌有效期为什么要短？——**答案**：令牌无法主动作废，有效期越短泄露后的风险窗口越小，配合 refresh token 续期。
