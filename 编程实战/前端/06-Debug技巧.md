# 前端 Debug 技巧

## React DevTools

- **Components**：看 props/state/hooks；改值试探。  
- **Profiler**：谁在反复渲染、耗时多少。  
- 勾选 Highlight updates，观察闪动范围。

症状 → 动作：

| 现象 | 先查 |
|---|---|
| 点了没反应 | 事件有没有绑上；被遮罩；pointer-events |
| 一改全刷 | 状态是否抬太高；Context value 是否新对象 |
| 列表乱跳 | `key` 是否稳定唯一 |
| Hook 报错 | 是否条件调用；组件是否变成普通函数调用 |

## 浏览器 Network / Console

1. Network：状态码、请求头（Authorization）、payload、响应体。  
2. 过滤 XHR/Fetch；看瀑布图谁拖慢。  
3. Console：红错点开栈；sourcemap 是否生效。  
4. Application：Cookie / LocalStorage / 是否脏数据。

跨域：看 OPTIONS 与 CORS 头；别用关安全的临时插件当长期方案。

## Node.js inspect

```bash
node --inspect dist/server.js
# 或
node --inspect-brk index.js
```

Chrome 打开 `chrome://inspect` → 进 Debugger。  
VS Code：`launch.json` 类型 `node`，断点直接打。

常见：

- 未处理 Promise rejection → 进程退出策略  
- 端口占用 `EADDRINUSE` → `lsof` / 改 PORT  
- 环境变量没进进程 → 只记录白名单非敏感项（如 PORT、NODE_ENV）或配置是否存在，禁止整包打印 process.env

## 常见报错速查

| 报错 | 含义 / 处理 |
|---|---|
| `Objects are not valid as a React child` | 别直接渲染对象；渲染字段或 `JSON.stringify` 调试 |
| `Too many re-renders` | `setState` 写在 render 路径；改到事件/effect |
| `Cannot read property of undefined` | 可选链；接口数据守卫 |
| Hydration mismatch（Next） | 服务端/客户端首屏 HTML 不一致；检查 `Date`/随机数/`window` |
| `module not found` | 别名、大小写、没装依赖、导出方式 CJS/ESM |
| CORS error | 后端放行源；或开发代理 |

## 套路（定位错误）

1. **复现**：最小步骤；隐身窗口排除扩展。  
2. **分层**：UI → 状态 → 请求 → 后端。  
3. **二分**：注释最近改动；git bisect。  
4. **证据**：截图 Network + 报错栈一起留存。

## Storybook 里调

组件在页面里被污染时，丢进 Storybook 只传 props，快速判断是组件 bug 还是容器/路由问题。

## 加厚：按证据链定位，而不是“每层都改一下”

| 层次 | 问题 | 证据 |
|---|---|---|
| UI/事件 | 用户操作是否触发处理函数？ | 事件断点、覆盖层、禁用状态、可访问名称 |
| 状态 | 输入是否正确？是否被旧闭包/旧响应覆盖？ | 组件 props/state、更新时序、Profiler |
| 请求 | 发出去什么？是否到达正确环境？ | URL、方法、状态码、时序、脱敏 request ID |
| 服务端 | 为什么拒绝/变慢？ | 同 request ID 的服务日志和 trace |
| 构建/发布 | 用户是否拿到旧代码/错误配置？ | 版本号、资源 hash、CDN/SW 缓存、source map |

不要只截 Console 的最后一条错：记录最早的原因、复现步骤、实际/预期、版本与环境；HAR、截图、heap snapshot 可能含 Cookie、Authorization、个人信息，分享前必须脱敏。

## 加厚：四类常见故障的排查顺序

### 1. 页面“偶尔回到旧数据”

用 Network 人工限速并快速切换筛选，确认响应返回顺序；检查 queryKey 是否完整、Effect 是否清理、是否有旧闭包。仅改为 debounce 不能保证在途请求顺序。

### 2. CORS / Cookie 失败

先分浏览器拦截还是服务端真实 401/403。看预检 OPTIONS、允许 Origin/方法/请求头；带凭据时不能随意用通配 Origin。再核对 Cookie 的 Domain/Path/SameSite/Secure 与跨站场景。开发代理只绕开发环境跨源问题，不会改变生产安全要求，`mode:'no-cors'` 也不是通用修复。

### 3. Hydration mismatch

对照服务端 HTML 和客户端第一次 render：随机数、当前时间、浏览器特有 API、时区/locale、无效 HTML 嵌套和外部 DOM 修改都可能造成不同。优先让首屏输入确定一致；必要的客户端差异在正确边界处理，不把 suppressHydrationWarning 当一键修复。

### 4. 卡顿与内存泄漏

先分网络等待、JS 长任务、React render 和布局/绘制。React Profiler 看组件提交，浏览器 Performance 看长任务和强制布局；两者不是同一个指标。反复进入/退出页面后对象持续增加，再查订阅、定时器、未释放监听器与无上限缓存；“内存暂时涨了”不直接证明泄漏。

## 常见坑

- 给所有 undefined 加可选链只隐藏数据契约错误；先确认字段缺失是否允许。
- source map 有文件但版本不匹配，栈会指错源码；部署版本、映射文件与监控 release 必须对应。
- 生产公开 --inspect 端口，给远程调试能力暴露入口；只在受控网络/本地授权排查，不随意监听公网。
- 为了查配置打印整个 process.env 或请求头；只记录白名单非敏感项、是否存在等信息。
- 误把 Error Boundary 当异步 try/catch；事件和请求失败仍要单独处理。

## 回归清单

- [ ] 最小复现失败前存在，修复后同一用例通过。
- [ ] 快速切换、慢网、断网、空数据、权限拒绝都能说明结果。
- [ ] 不靠清空所有缓存/重启所有服务作为唯一“修复”。
- [ ] 补一条 story、单测或 E2E，记录根因与影响边界，确认日志不泄密。

## 面试口述

> 我先固定版本和复现步骤，再沿事件、状态、请求、服务端逐层找第一个偏离预期的证据。卡顿区分网络、JS、组件渲染和布局，偶发旧数据重点看竞态与缓存键，水合问题对照两端首次输入。修复后用原复现和边界场景做回归，并补自动化用例。调试产物和日志也要脱敏，不能为排障引入新的安全问题。

## 延伸阅读

- 开源入口：[React](../开源精选/frontend/react-README.md)、[Next.js](../开源精选/frontend/nextjs-README.md)、[Storybook](../开源精选/frontend/storybook-README.md)。
- 官方机制：[React Effect 的清理与竞态](https://react.dev/reference/react/useEffect)。
