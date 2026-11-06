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
- 环境变量没进进程 → 打印 `process.env`（注意别打密钥到日志）

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
