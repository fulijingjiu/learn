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
- **事件循环**分多个阶段处理计时器、IO 等；实际时序受阶段/上下文/版本影响，别阻塞主线程（同步大文件、重计算）。  
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

## 加厚：事件循环不是“异步就并行”

Node.js 通常在一个 JS 主线程上执行回调；网络 IO 可由操作系统事件机制驱动，部分文件、DNS、加密操作使用 libuv 工作线程池。工作线程池不是任意 JS 代码的自动加速器：在 async 函数里执行五秒循环，仍可能阻塞主线程。

| 工作类型 | 常见处理 | 风险 |
|---|---|---|
| HTTP / 数据库等待 | 异步 IO + 超时 | 并发无限会压垮连接池/下游 |
| 大量 JSON 解析、复杂排序 | 限制输入、拆分或 worker_threads | 外层加 await 也不自动卸载 CPU |
| 大文件转发 | stream + backpressure | 一次读入内存可能 OOM |
| 高成本密码运算 | 使用安全异步 API，评估线程池与限流 | 大量请求挤满线程池影响其他操作 |

Promise 回调等微任务会在相应边界被处理，`process.nextTick` 也有自己的调度语义；无限递归安排这些工作可让 IO 饿死。`setTimeout(0)` 是到期后有机会执行，不是立刻执行；它和 `setImmediate` 的先后受调用场景、事件循环阶段和版本影响，不背无条件顺序。

## 加厚：并发、超时、重试要一起设计

1. `Promise.all` 快速拒绝不等于已经启动的其他请求自动取消。
2. `allSettled` 能收集结果，不等于无限并发安全；先限制并发，再按业务聚合失败。
3. 入站请求要有上限，下游调用要有超时与取消传递；重试只针对可重试错误。
4. 支付、发消息等写动作需幂等键与状态查询，超时可能是“执行成功但响应丢了”。
5. 对慢客户端用流和背压，不把所有响应 chunk 无限制塞数组。背压是让生产速度服从消费能力，不是单纯加大缓冲区。

## 常见坑

- 在高并发请求路径使用同步 fs、巨大 JSON.parse、灾难性正则，造成全服务延迟上升。
- 把 `forEach(async () => ...)` 当成能等待整个循环；顺序执行用 for...of + await，有限并发用受控任务池。
- 忘记检查 fetch 的 `response.ok`：HTTP 4xx/5xx 通常不会自动 reject。
- 环境变量都是字符串；PORT、布尔开关要显式解析/校验，不能 `Boolean('false')`。
- 无限制注册事件监听器、不清理定时器、缓存无淘汰，导致长期内存增长。
- 忽略 SIGTERM：应停止接新请求、有限等待在途任务、释放数据库池，超时再退出；不能让进程永远卡在优雅关闭。

## 排障检查清单

- 延迟高：先分 event loop lag、CPU、堆/GC、下游耗时，结合请求 trace，不先拍脑袋加进程。
- 内存涨：看持续趋势、活跃句柄/缓存规模；受控环境做 heap snapshot，快照可能含敏感数据。
- 模块导入错：看 package.json type、导出条件、扩展名、工具链目标版本，不只改 import 字样。
- 依赖冲突：确认 Node 与包管理器版本、锁文件，干净安装验证，不同时提交多套互相竞争的锁文件。

## 面试口述

> Node 通过事件循环处理大量 IO 等待，但 JS 回调仍会占主线程，async 不会把 CPU 运算自动放到后台。我会对下游调用做有限并发、超时和必要的幂等重试，对大数据流使用背压。CPU 重任务考虑 worker_threads 或独立服务，先量 event loop lag 和 CPU 再决定扩容。生产还要管理连接池、敏感日志和优雅退出。

## 延伸阅读

- 开源应用背景：[Next.js README](../开源精选/frontend/nextjs-README.md)；该 README 不覆盖事件循环全部机制。
- 官方：[Don't Block the Event Loop](https://nodejs.org/en/learn/asynchronous-work/dont-block-the-event-loop)。
