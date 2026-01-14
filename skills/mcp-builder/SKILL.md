---
name: mcp-builder
description: 指导如何创建高质量的 MCP（Model Context Protocol）server，使 LLM 能通过设计良好的工具访问外部服务。无论在 Python（FastMCP）还是 Node/TypeScript（MCP SDK）中构建集成外部 API 的 server，都应使用此技能。
license: 完整条款见 LICENSE.txt
---

# MCP Server Development Guide

## Overview

构建 MCP server，让 LLM 能通过设计完善的工具与外部服务交互。衡量 server 质量的标准是它能多大程度帮助 LLM 完成真实任务。

---

# Process

## 🚀 High-Level Workflow

高质量 MCP server 需经过四个阶段：

### Phase 1: Deep Research and Planning

#### 1.1 Understand Modern MCP Design

**API Coverage vs. Workflow Tools：**
在全面覆盖 API endpoints 与专项 workflow tools 之间取得平衡。workflow tools 在特定任务上更顺手，而全面覆盖能让 agent 自由组合操作。不同客户端表现不同：有的善于执行代码组合基础工具，有的更适合高层 workflow。若拿不准，优先确保 API 覆盖。

**Tool Naming and Discoverability：**
使用清晰、具描述性的名称帮助 agent 快速定位工具。保持一致前缀（如 `github_create_issue`、`github_list_repos`）并使用动词导向的命名。

**Context Management：**
提供简洁的工具描述，并支持过滤/分页。让工具返回聚焦且相关的数据。有的客户端支持代码执行，可在客户端侧进一步过滤与处理。

**Actionable Error Messages：**
错误信息要指向解决方案，提供具体建议与下一步。

#### 1.2 Study MCP Protocol Documentation

**浏览 MCP 规范：**

先查看 sitemap：`https://modelcontextprotocol.io/sitemap.xml`

然后为 markdown 版本获取带 `.md` 后缀的页面（例：`https://modelcontextprotocol.io/specification/draft.md`）。

重点阅读：
- 规范概览与架构
- 传输机制（streamable HTTP、stdio）
- Tool、resource、prompt 定义

#### 1.3 Study Framework Documentation

**推荐技术栈：**
- **Language**：TypeScript（SDK 成熟，兼容 MCPB 等运行环境，且 AI 擅长生成 TypeScript，得益于其广泛使用、静态类型与优秀 lint）
- **Transport**：远程 server 选 streamable HTTP + 无状态 JSON（易扩展、易维护），本地 server 用 stdio。

**加载框架文档：**

- **MCP Best Practices**：[📋 View Best Practices](./reference/mcp_best_practices.md) - 核心准则

**TypeScript（推荐）：**
- **TypeScript SDK**：用 WebFetch 加载 `https://raw.githubusercontent.com/modelcontextprotocol/typescript-sdk/main/README.md`
- [⚡ TypeScript Guide](./reference/node_mcp_server.md) - TypeScript 模式与示例

**Python：**
- **Python SDK**：用 WebFetch 加载 `https://raw.githubusercontent.com/modelcontextprotocol/python-sdk/main/README.md`
- [🐍 Python Guide](./reference/python_mcp_server.md) - Python 模式与示例

#### 1.4 Plan Your Implementation

**理解 API：**
阅读服务的 API 文档，确认关键 endpoints、认证方式、数据模型。必要时使用 web search 与 WebFetch。

**工具选择：**
优先确保 API 覆盖。列出要实现的 endpoints，从最常见的操作开始。

---

### Phase 2: Implementation

#### 2.1 Set Up Project Structure

根据语言选择参考指南：
- [⚡ TypeScript Guide](./reference/node_mcp_server.md) - 项目结构、package.json、tsconfig.json
- [🐍 Python Guide](./reference/python_mcp_server.md) - 模块组织、依赖

#### 2.2 Implement Core Infrastructure

创建共享工具：
- 带认证的 API client
- 错误处理辅助
- 响应格式化（JSON/Markdown）
- 分页支持

#### 2.3 Implement Tools

对每个工具：

**Input Schema：**
- TypeScript 用 Zod，Python 用 Pydantic
- 写明约束与清晰描述
- 在字段描述中加入示例

**Output Schema：**
- 在可行时定义 `outputSchema` 以输出结构化数据
- TypeScript SDK 中使用 `structuredContent`
- 帮助客户端理解与处理输出

**Tool Description：**
- 凝练总结功能
- 参数说明
- 返回类型 schema

**Implementation：**
- I/O 操作使用 async/await
- 规范处理错误并给出可执行提示
- 需要时支持分页
- 使用现代 SDK 时同时返回文本与结构化数据

**Annotations：**
- `readOnlyHint`: true/false
- `destructiveHint`: true/false
- `idempotentHint`: true/false
- `openWorldHint`: true/false

---

### Phase 3: Review and Test

#### 3.1 Code Quality

检查：
- 避免重复代码（DRY）
- 错误处理一致
- 类型覆盖完整
- 工具描述清晰

#### 3.2 Build and Test

**TypeScript：**
- 运行 `npm run build` 确认可编译
- 使用 MCP Inspector 测试：`npx @modelcontextprotocol/inspector`

**Python：**
- 通过 `python -m py_compile your_server.py` 验证语法
- 使用 MCP Inspector 测试

更多测试细节与质量清单见各语言指南。

---

### Phase 4: Create Evaluations

实现 server 后，需要创建完善的评估来验证效果。

**加载 [✅ Evaluation Guide](./reference/evaluation.md) 获取完整评估指南。**

#### 4.1 Understand Evaluation Purpose

评估用于测试 LLM 是否能有效使用你的 MCP server 回答真实且复杂的问题。

#### 4.2 Create 10 Evaluation Questions

遵循 evaluation guide 流程创建评估：

1. **Tool Inspection**：列出可用工具并理解功能
2. **Content Exploration**：使用 READ-ONLY 操作探索数据
3. **Question Generation**：设计 10 个复杂且真实的问题
4. **Answer Verification**：亲自解答并校验答案

#### 4.3 Evaluation Requirements

确保每个问题：
- **Independent**：彼此独立
- **Read-only**：只需非破坏性操作
- **Complex**：需要多次调用工具、深入探索
- **Realistic**：源自真实用户关心的场景
- **Verifiable**：有唯一、可字符串比对的答案
- **Stable**：答案不会随时间变化

#### 4.4 Output Format

按以下结构生成 XML：

```xml
<evaluation>
  <qa_pair>
    <question>Find discussions about AI model launches with animal codenames. One model needed a specific safety designation that uses the format ASL-X. What number X was being determined for the model named after a spotted wild cat?</question>
    <answer>3</answer>
  </qa_pair>
<!-- More qa_pairs... -->
</evaluation>
```

---

# Reference Files

## 📚 Documentation Library

在开发过程中按需加载这些资源：

### Core MCP Documentation (Load First)
- **MCP Protocol**：先访问 `https://modelcontextprotocol.io/sitemap.xml`，再获取带 `.md` 的具体页面
- [📋 MCP Best Practices](./reference/mcp_best_practices.md) - 通用指南，涵盖：
  - server 与工具命名规范
  - 响应格式（JSON vs Markdown）
  - 分页最佳实践
  - 传输方案选择（streamable HTTP vs stdio）
  - 安全与错误处理标准

### SDK Documentation (Load During Phase 1/2)
- **Python SDK**：`https://raw.githubusercontent.com/modelcontextprotocol/python-sdk/main/README.md`
- **TypeScript SDK**：`https://raw.githubusercontent.com/modelcontextprotocol/typescript-sdk/main/README.md`

### Language-Specific Implementation Guides (Load During Phase 2)
- [🐍 Python Implementation Guide](./reference/python_mcp_server.md) - 完整的 Python/FastMCP 指南，包含：
  - server 初始化模式
  - Pydantic 模型示例
  - 使用 `@mcp.tool` 注册工具
  - 完整示例代码
  - 质量检查清单

- [⚡ TypeScript Implementation Guide](./reference/node_mcp_server.md) - 完整的 TypeScript 指南，包含：
  - 项目结构
  - Zod schema 模式
  - 使用 `server.registerTool` 注册工具
  - 完整示例代码
  - 质量检查清单

### Evaluation Guide (Load During Phase 4)
- [✅ Evaluation Guide](./reference/evaluation.md) - 评估创建指南，包含：
  - 题目编写原则
  - 答案校验策略
  - XML 格式规范
  - 示例问答
  - 使用提供脚本运行评估
