# React 开发知识（面试口述 + 干活要点）

> 官方入口：[react.dev](https://react.dev/) · 开源：`../开源精选/frontend/react-README.md`（MIT）

## 一句话

React 是**声明式 UI 库**：你描述「数据长什么样」，框架帮你高效更新 DOM。核心是**组件 + 状态 + Hooks**。

## 组件

- **函数组件**为主（类组件面试知道即可）。
- Props 只读；要改界面 → 改 state / 上游数据。
- 组合优于继承：用 children、render props、组合小组件。

```tsx
function Hello({ name }: { name: string }) {
  return <div>Hello {name}</div>;
}
```

**口述**：组件是 UI 的复用单元；props 向下传，事件向上抛（回调）。

## Hooks（必会）

| Hook | 干什么 | 坑 |
|---|---|---|
| `useState` | 本地状态 | 异步更新；函数式更新 `set(x => x+1)` |
| `useEffect` | 副作用（请求、订阅） | 依赖数组；清理函数防泄漏 |
| `useMemo` / `useCallback` | 缓存值/函数 | 别滥用，先量再优化 |
| `useRef` | DOM / 可变箱子不触发渲染 | 改 `.current` 不重渲染 |
| `useContext` | 跨层传值 | 值变则消费者重渲染，粒度要控 |

**规则**：只在组件顶层调 Hook；不要在循环/条件里调。

## 状态放哪

1. **只用这里** → `useState`  
2. **兄弟多处共享** → 上提 / Context / Zustand  
3. **服务端数据** → TanStack Query / SWR（缓存、重试、失效）  
4. **全局复杂业务** → Zustand / Redux Toolkit（别一上来就 Redux）

## 路由

- SPA：`react-router`（`BrowserRouter`、`Routes`、`Outlet`、loader）。  
- 全栈 / SSR：Next.js App Router（文件路由、`layout.tsx`、Server Components）。

**干活**：路由懒加载 `React.lazy` + `Suspense`；权限路由包一层 Guard。

## 性能要点

1. 先定位：React DevTools Profiler / `why-did-you-render`。  
2. 列表：稳定 `key`（别用 index 当唯一身份）。  
3. 大列表：虚拟列表。  
4. 减少无意义重渲染：拆组件、memo、状态下沉。  
5. 代码分割：路由级 + 重型组件级。  
6. 图片/字体：懒加载、合适尺寸。

## 面试高频口述

- **虚拟 DOM**：内存中的树；diff 后批量刷真实 DOM（现代 React 还有并发特性）。  
- **受控 vs 非受控**：value+onChange vs defaultValue+ref。  
- **key 的作用**：帮助 reconciler 识别节点身份。  
- **Strict Mode**：开发环境故意双调 effect，帮你找不纯副作用。

## 干活检查清单

- [ ] TypeScript 类型住 props / API  
- [ ] 错误边界（Error Boundary）罩关键业务区  
- [ ] 空态 / 加载 / 错误三态齐全  
- [ ] 无障碍：按钮可聚焦、表单有 label
