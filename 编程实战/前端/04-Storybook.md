# Storybook

> 开源：`../开源精选/frontend/storybook-README.md`（MIT） · 文档：[storybook.js.org](https://storybook.js.org/)

## 干什么

在**隔离环境**里开发和展示 UI 组件：不依赖完整页面、路由、后端。用途：

- 组件文档 + 可视化状态（loading / 空 / 错 / 超长文案）  
- 设计师 / 产品对一下交互  
- 视觉回归、无障碍 addon、交互测试  

一句话：**组件的工作台和说明书**。

## 快速开始

```bash
pnpm dlx storybook@latest init
pnpm storybook
```

常生成 `.storybook/`（`main.ts`、`preview.ts`）和示例 stories。

## CSF 基本写法（Component Story Format）

以下以现代 React + Vite 适配包为例，Button 假设支持 children、variant、loading、onClick；其他框架或旧版请换成项目匹配的类型/测试导入路径。

```tsx
// Button.stories.tsx
import type { Meta, StoryObj } from "@storybook/react-vite";
import { fn } from "storybook/test";
import { Button } from "./Button";

const meta = {
  title: "UI/Button",
  component: Button,
  tags: ["autodocs"],
  args: { onClick: fn() },
} satisfies Meta<typeof Button>;

export default meta;
type Story = StoryObj<typeof meta>;

export const Primary: Story = {
  args: { children: "提交", variant: "primary" },
};

export const Loading: Story = {
  args: { children: "提交", loading: true },
};
```

要点：`meta` 描述组件；每个 `export` 是一个故事（一种状态）；`args` 可在 UI 里拧。

## 和组件库联调

1. 在 `preview.tsx` 里包 `ConfigProvider`（antd）或 `ThemeProvider`（MUI）：  

```tsx
import { ConfigProvider } from "antd";
import zhCN from "antd/locale/zh_CN";

export const decorators = [
  (Story) => (
    <ConfigProvider locale={zhCN}>
      <Story />
    </ConfigProvider>
  ),
];
```

2. 全局 CSS / Tailwind 在 `preview` 引入，保证 Story 和真实 App 视觉一致。  
3. 业务组件依赖 API：用 MSW addon mock，或 stories 里传假数据 props。  
4. shadcn 组件：故事直接引 `components/ui/*`，改源码后热更新可见。

## 常用 addon

- Controls、Actions、Viewport：现代版本可能内置，旧版本常由 Essentials 提供，按主版本核对  
- **a11y**：无障碍检查  
- **docs**：从 CSF 生成文档  

## 干活建议

- 基础 UI 与业务复合组件都写故事；至少覆盖主态 + 边界态。  
- CI 可跑 `test-runner` / Chromatic 做视觉回归（按团队需要）。  
- 别把 Storybook 当唯一文档：复杂流程仍要产品说明。

## 加厚：故事不是“截图页”，而是状态契约

| 测试层 | 回答的问题 | 不能替代 |
|---|---|---|
| args / 静态 story | 某组输入下组件长什么样 | 交互后状态是否正确 |
| play / 交互测试 | 点击、输入、校验、回调是否符合预期 | 全站路由、真实鉴权和后端链路 |
| 视觉回归 | 样式是否发生可见变化 | 业务逻辑正确性 |
| a11y 检查 | 是否存在可自动检测的无障碍问题 | 人工键盘与读屏体验 |
| E2E | 真实页面/系统链路能否完成 | 大量低成本组件边界覆盖 |

## 交互测试：操作与断言都要等待

下面承接前面的 meta，假设 Button 有 onClick 属性，现代版本支持从 `storybook/test` 导入；具体框架适配包以项目锁定版本为准。

```tsx
import { expect } from "storybook/test";

export const Clickable: Story = {
  args: { children: "保存" },
  play: async ({ canvas, userEvent, args }) => {
    await userEvent.click(canvas.getByRole("button", { name: "保存" }));
    await expect(args.onClick).toHaveBeenCalled();
  },
};
```

优先通过 role 和可访问名称找元素，少依赖易变 class。异步状态用合适的等待查询/断言，不靠固定 sleep。Portal 弹窗可能在 canvas 外，需要按当前测试 API 查询实际容器，不能因此放弃无障碍名称。

## Mock 与隔离

- API 用 MSW 等在网络层模拟成功、空数据、4xx/5xx、超时；每条 story 覆盖不同处理器，避免真实写入接口。
- 路由、主题、国际化、数据请求 Provider 用 decorator 配齐，但不要全量启动业务 App 掩盖依赖。
- 给每个故事独立缓存，或在测试之间重置，防止前一个 story 的 Query 缓存让后一个假通过。
- 时间、随机数和动画保持可控，减轻视觉测试抖动；不要只提高差异阈值掩盖真实布局问题。
- 不在 args、静态构建包或 mock 数据里放真实 token、客户数据；公开部署 Storybook 前也要审查内容。

## 常见坑

- Story 能渲染不代表真实页面可用：SSR、路由、权限、CSS 顺序仍需集成验证。
- 只有默认故事，没有 loading/错误/长文本，升级时最容易漏边界回归。
- 使用 action 自动推断代替可断言的 spy，交互测试无法可靠验证回调；优先显式 fn()。
- preview 里有 JSX 却使用 .ts；应改 .tsx 并与构建工具配置一致。
- 不同 Storybook 主版本直接照抄插件名、测试导入路径；现代版本可用 Vitest addon 等，旧项目可能使用 test-runner，按对应文档选用，不混装整套旧 Essentials。

## 面试口述

> Storybook 把组件的一组输入和边界状态变成可复现的工作台，不只是给设计师看截图。我用 decorator 补主题/路由，用 mock 隔离接口，再用 play 测试关键交互。视觉、交互、无障碍和 E2E 关注不同问题，需要组合使用。CI 要固定数据和运行环境，升级库时跑回归，不能拿一个能打开的故事当生产验收。

## 延伸阅读

- 已读开源：[Storybook README](../开源精选/frontend/storybook-README.md)。
- 官方机制核验：[Interaction tests](https://storybook.js.org/docs/writing-tests/interaction-testing)。
