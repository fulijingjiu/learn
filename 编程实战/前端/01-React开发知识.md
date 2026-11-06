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
| `useState` | 本地状态 | 本次渲染为快照；依赖前值用函数式更新 `set(x => x+1)` |
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
- **Strict Mode**：开发环境可能额外执行 Effect 的 setup/cleanup 检查，帮助发现清理缺失。

## 干活检查清单

- [ ] TypeScript 类型住 props / API  
- [ ] 错误边界（Error Boundary）罩关键业务区  
- [ ] 空态 / 加载 / 错误三态齐全  
- [ ] 无障碍：按钮可聚焦、表单有 label

## 加厚：从 render、快照到 Effect

### 状态为什么不能当普通变量立即读？

一次渲染中的 state 是该次渲染的快照。调用 setter 是请求下一次更新，不会修改当前闭包已经拿到的值；React 还可能批处理更新。因此连续 `setCount(count+1)` 可能都使用同一个旧 count，而依赖前值时应写 `setCount(c => c+1)`。这比笼统说“setState 永远异步”更准确。

| 场景 | 推荐 | 原因 |
|---|---|---|
| 从 props/state 算筛选结果 | render 内直接算，贵时再评估 useMemo | 不制造两份需要同步的状态 |
| 用户点击提交 | 事件处理函数 | 提交由明确交互触发，不用 Effect 猜测 |
| 订阅 WebSocket、外部 DOM API | Effect + cleanup | 与组件外部系统同步 |
| 表单修改未提交数据 | 本地 state | 不立即污染服务端缓存 |

### Effect 的完整生命周期

组件提交后执行 setup；依赖变化时，先清理上一次 setup，再执行新的 setup；卸载时清理。依赖以 `Object.is` 比较，组件内新建对象/函数可能每次身份不同。别为了“只执行一次”禁用依赖检查，应该重新组织逻辑。

StrictMode 开发环境可能额外执行 setup→cleanup→setup 来检查清理是否对称，这不代表生产所有请求必发两次。也不应使用一个全局布尔值掩盖漏清理。

### 请求竞态：后发不一定后到

搜索词从 A 变 B，A 的慢响应可能晚于 B 返回，覆盖新结果。请求层支持取消时传 AbortSignal，并防止过期结果写回：

```tsx
useEffect(() => {
  const controller = new AbortController();
  async function load() {
    try {
      const response = await fetch(url, { signal: controller.signal });
      if (!response.ok) throw new Error(`HTTP ${response.status}`);
      const result = await response.json();
      if (!controller.signal.aborted) setData(result);
    } catch (error) {
      if (!controller.signal.aborted) setError(error);
    }
  }
  void load();
  return () => controller.abort();
}, [url]);
```

以上假设组件已声明 url、setData、setError；生产请求还需重置状态、校验返回结构，复杂缓存优先用数据请求库。客户端取消不保证服务器端已经执行的写动作撤销。

## 常见坑（深入版）

- 直接修改数组/对象再 set 同一引用，可能不触发期望更新；创建新对象并保持业务身份。
- 用随机 key 导致每次重挂载；可重排列表用稳定业务 ID，索引仅适合身份确实不变化的场景。
- 把 useMemo 当正确性保证；缓存可能被丢弃，核心计算应纯且可重复。
- 将路由 Guard 当权限安全边界；真正数据权限在后端验证。
- Error Boundary 主要隔离子树渲染错误，不能包办任意事件回调、定时器与异步请求错误；这些要单独 catch/状态化处理。

## 面试口述（完整 4 句）

> React 用组件和状态声明 UI，render 应保持纯，state 是一次渲染的快照，更新会进入队列。普通派生数据在 render 计算，用户行为放事件处理，只有与外部系统同步才使用 Effect。Effect 要写对依赖与清理，处理订阅泄漏和请求竞态，再用 Profiler 定位性能问题。服务端数据与本地交互状态分开管理，不靠无限加 memo 或关掉 StrictMode 掩盖问题。

## 延伸阅读

- 已读开源入口：[React README](../开源精选/frontend/react-README.md)。
- 官方机制核验：[useEffect](https://react.dev/reference/react/useEffect)。
