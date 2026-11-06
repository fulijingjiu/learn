# Node.js 开发知识

> 运行时：[nodejs.org](https://nodejs.org/) · 常和 React 同仓（BFF / API）

## 一句话

Node = **Chrome 的 V8 + 事件循环**，用 JS/TS 写服务端；适合 I/O 密集（API、网关、脚本），CPU 重活要 worker / 别的语言。

## 模块

- **CommonJS**：`require` / `module.exports`（老项目、很多工具）。  
- **ESM**：`import` / `export`（`"type": "module"` 或 `.mjs`）。  
- 混用小心：CJS 里动态 `import()` 可以加载 ESM。

路径：`path`、`fileURLToPath`；读文件：`fs/promises`。

## 异步模型

- 回调 → Promise → `async/await`（现在默认用后者）。  
- **事件循环**：timers → I/O → check → close；别阻塞事件循环（同步大文件、重计算）。  
- 并发：`Promise.all` / `allSettled`；限流用 p-limit 一类。

```js
async function main() {
  const data = await fetch(url).then(r => r.json());
  console.log(data);
}
main().catch(console.error);
```

## Web 框架（选一个说清）

| 框架 | 特点 | 何时用 |
|---|---|---|
| **Express** | 生态大、中间件多 | 简单 API、遗留项目 |
| **Fastify** | 更快、schema 校验 | 新 API、要性能 |
| **NestJS** | Angular 风格、DI、模块 | 中大型企业后端 TS |

一句话面试：Express 灵活；Fastify 性能+校验；Nest 工程化强。

## 包管理

- **npm**（自带）、**pnpm**（磁盘省、严格）、**yarn**。  
- 锁文件必须提交（`package-lock.json` / `pnpm-lock.yaml`）。  
- 脚本：`package.json` → `scripts`；本地 bin 用 `npx` / `pnpm exec`。  
- 版本：semver；生产依赖 vs `devDependencies`。

## 环境变量

- 开发：`.env` + `dotenv`（或框架内置）。  
- **密钥不要进 Git**；用部署平台密钥 / Vault。  
- 区分：`NODE_ENV=development|production|test`。  
- 校验：zod 校验 `process.env`，启动失败比跑一半崩好。

```ts
import { z } from "zod";
const env = z.object({
  PORT: z.coerce.number().default(3000),
  DATABASE_URL: z.string().url(),
}).parse(process.env);
```

## 干活要点

- HTTP：状态码语义、超时、重试幂等。  
- 日志：结构化 JSON；请求带 `requestId`。  
- 进程：`SIGTERM` 优雅退出；PM2 / systemd / K8s。  
- 安全：依赖审计 `npm audit`；防原型污染；校验输入。
