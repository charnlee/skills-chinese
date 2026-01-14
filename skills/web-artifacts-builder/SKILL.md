---
name: web-artifacts-builder
description: 使用现代前端技术（React、Tailwind CSS、shadcn/ui）创建复杂、多组件 claude.ai HTML artifacts 的工具集。适用于需要 state management、routing 或 shadcn/ui 组件的复杂 artifact，非单纯单文件 HTML/JSX。
license: 完整条款见 LICENSE.txt
---

# Web Artifacts Builder

构建高阶前端 artifact 的流程：
1. 运行 `scripts/init-artifact.sh` 初始化前端仓库
2. 修改生成的代码以开发 artifact
3. 使用 `scripts/bundle-artifact.sh` 将所有代码打包成单一 HTML
4. 将 artifact 展示给用户
5. （可选）测试 artifact

**Stack**：React 18 + TypeScript + Vite + Parcel（打包）+ Tailwind CSS + shadcn/ui

## Design & Style Guidelines

**非常重要**：避免“AI slop”风格，不要滥用完全居中的布局、紫色渐变、统一圆角和 Inter 字体。

## Quick Start

### Step 1: Initialize Project

运行初始化脚本创建新的 React 项目：
```bash
bash scripts/init-artifact.sh <project-name>
cd <project-name>
```

脚本会生成：
- ✅ 基于 Vite 的 React + TypeScript
- ✅ Tailwind CSS 3.4.1，包含 shadcn/ui 主题系统
- ✅ 已配置 `@/` 路径别名
- ✅ 预装 40+ shadcn/ui 组件
- ✅ 包含所有 Radix UI 依赖
- ✅ 配置好的 `.parcelrc`（支持打包与路径别名）
- ✅ 兼容 Node 18+（自动侦测并固定 Vite 版本）

### Step 2: Develop Your Artifact

编辑生成的文件来构建 artifact。参考下方 **Common Development Tasks**（如需）。

### Step 3: Bundle to Single HTML File

将 React 应用打包成单个 HTML：
```bash
bash scripts/bundle-artifact.sh
```

脚本会生成 `bundle.html`，其中内联了全部 JavaScript、CSS 与依赖，可在 Claude 对话中直接作为 artifact 分享。

**前提**：项目根目录需存在 `index.html`。

**脚本执行内容：**
- 安装打包依赖（parcel、@parcel/config-default、parcel-resolver-tspaths、html-inline）
- 创建带路径别名支持的 `.parcelrc`
- 通过 Parcel 构建（无 source map）
- 使用 html-inline 将所有资源嵌入单一 HTML

### Step 4: Share Artifact with User

将打包好的 HTML 在对话中发送给用户以供查看。

### Step 5: Testing/Visualizing the Artifact (Optional)

此步骤完全可选，仅在必要或应用户请求时执行。

如需测试/预览，可使用现有工具（其他技能或 Playwright、Puppeteer 等内置工具）。一般不要提前测试，以免延迟用户看到成品。如用户反馈问题或明确要求，再进行测试。

## Reference

- **shadcn/ui components**: https://ui.shadcn.com/docs/components
