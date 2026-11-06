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

```tsx
// Button.stories.tsx
import type { Meta, StoryObj } from "@storybook/react";
import { Button } from "./Button";

const meta = {
  title: "UI/Button",
  component: Button,
  tags: ["autodocs"],
  argTypes: { onClick: { action: "clicked" } },
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

1. 在 `preview.ts` 里包 `ConfigProvider`（antd）或 `ThemeProvider`（MUI）：  

```ts
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

- **Essentials**：Controls、Actions、Viewport  
- **a11y**：无障碍检查  
- **docs**：从 CSF 生成文档  

## 干活建议

- 基础 UI 与业务复合组件都写故事；至少覆盖主态 + 边界态。  
- CI 可跑 `test-runner` / Chromatic 做视觉回归（按团队需要）。  
- 别把 Storybook 当唯一文档：复杂流程仍要产品说明。
