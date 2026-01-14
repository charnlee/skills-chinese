> **说明：** 本仓库是官方 Anthropic Skills 的中文化 Fork，旨在让中文使用者更容易理解与学习这些技能。关于 Agent Skills 标准，请访问 [agentskills.io](http://agentskills.io)。

# Skills
Skills 是由说明、脚本与资源组成的文件夹，Claude 会按需加载，用于提升在特定任务上的表现。无论是按照公司品牌规范创建文档、遵循组织特定流程分析数据，还是自动化个人任务，Skills 都能教会 Claude 以可重复方式完成这些工作。

了解更多：
- [什么是 Skills？](https://support.claude.com/en/articles/12512176-what-are-skills)
- [如何在 Claude 中使用 Skills](https://support.claude.com/en/articles/12512180-using-skills-in-claude)
- [如何创建自定义 Skills](https://support.claude.com/en/articles/12512198-creating-custom-skills)
- [Agent Skills 如何让智能体适配真实世界](https://anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)

# 关于本仓库

此仓库在官方 Skills 的基础上进行完整中文化翻译，帮助中文开发者学习 Claude Skills 系统的能力，涵盖创意应用（艺术、音乐、设计）、技术任务（Web 应用测试、MCP server 生成）以及企业工作流（沟通、品牌等）。

每个 Skill 都在独立文件夹中，包含描述与指令的 `SKILL.md`。可浏览这些示例以获取灵感或了解不同模式与方法。

仓库中多数技能开源（Apache 2.0）。此外，我们也开放了为 [Claude 文档功能](https://www.anthropic.com/news/create-files) 提供支持的文档创建/编辑技能，位于 [`skills/docx`](./skills/docx)、[`skills/pdf`](./skills/pdf)、[`skills/pptx`](./skills/pptx)、[`skills/xlsx`](./skills/xlsx) 子目录。这些技能为 source-available（非开源），但我们希望让开发者参考这些在生产环境中使用的复杂技能。

## 免责声明

**这些技能仅用于演示与学习目的。** Claude 提供的实际功能可能与此处展示不同。这些技能旨在展示模式与可能性。若要在关键任务中依赖它们，务必在自己的环境中充分测试。

# Skill 分类
- [./skills](./skills)：涵盖创意设计、开发技术、企业沟通、文档等示例技能
- [./spec](./spec)：Agent Skills 规范
- [./template](./template)：技能模板

# 在 Claude Code、Claude.ai 与 API 中体验

## Claude Code
可在 Claude Code 中运行以下命令，将该仓库注册为 Claude Code Plugin 市场：
```
/plugin marketplace add anthropics/skills
```

然后安装技能包：
1. 选择 `Browse and install plugins`
2. 选择 `anthropic-agent-skills`
3. 选择 `document-skills` 或 `example-skills`
4. 点击 `Install now`

或者直接安装：
```
/plugin install document-skills@anthropic-agent-skills
/plugin install example-skills@anthropic-agent-skills
```

安装后，直接在对话中提及技能即可。例如安装 `document-skills` 后，可让 Claude Code 执行：“使用 PDF 技能提取 `path/to/some-file.pdf` 的表单字段”。

## Claude.ai

付费计划的 Claude.ai 已预装这些示例技能。上传自定义技能或使用本仓库技能的方法见 [Using skills in Claude](https://support.claude.com/en/articles/12512180-using-skills-in-claude#h_a4222fa77b)。

## Claude API

可通过 Claude API 使用 Anthropic 预构建的技能，也可以上传自定义技能。参考 [Skills API Quickstart](https://docs.claude.com/en/api/skills-guide#creating-a-skill)。

# 创建基础 Skill

Skill 的结构非常简单：一个包含 YAML frontmatter 与说明的 `SKILL.md` 文件的文件夹。可从仓库中的 **template-skill** 开始：

```markdown
---
name: my-skill-name
description: 对技能作用与触发时机的清晰描述
---

# My Skill Name

[在此编写技能被激活时 Claude 要遵循的指令]

## Examples
- Example usage 1
- Example usage 2

## Guidelines
- Guideline 1
- Guideline 2
```

frontmatter 仅需两个字段：
- `name`：技能唯一标识（使用小写，用连字符代替空格）
- `description`：完整描述技能功能与使用场景

后续的 Markdown 内容包含指令、示例与指南。更多细节见 [How to create custom skills](https://support.claude.com/en/articles/12512198-creating-custom-skills)。

# 合作伙伴技能

Skills 也是让 Claude 学会使用特定软件的绝佳方式。若合作伙伴提供了优秀示例，我们会在此展示：

- **Notion** - [Notion Skills for Claude](https://www.notion.so/notiondevs/Notion-Skills-for-Claude-28da4445d27180c7af1df7d8615723d0)
