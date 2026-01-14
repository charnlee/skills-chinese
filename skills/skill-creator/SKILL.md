---
name: skill-creator
description: 编写高效技能的指南。当用户希望创建新技能或更新现有技能，以专用知识、工作流或工具集成扩展 Claude 能力时，使用此技能。
license: 完整条款见 LICENSE.txt
---

# Skill Creator

本技能提供创建高质量技能的完整方法论。

## About Skills

Skill 是扩展 Claude 能力的模块化自包含包，提供特定领域的知识、工作流与工具。可以把它视为针对某项任务的 onboarding guide，把 Claude 从通用助手变成掌握专门流程的专家。

### What Skills Provide

1. Specialized workflows —— 针对特定领域的多步骤流程
2. Tool integrations —— 针对特定文件格式或 API 的使用说明
3. Domain expertise —— 公司特有的知识、schema、业务逻辑
4. Bundled resources —— 为复杂/重复任务准备的脚本、参考资料与素材

## Core Principles

### Concise is Key

上下文窗口是公共资源。Skills 需要与 system prompt、对话历史、其他技能元数据以及用户请求共享空间。

**默认假设：Claude 已经足够聪明。** 仅提供其尚不了解的内容。质疑每条信息：“Claude 是否真的需要？”、“这段文字值得它占用 tokens 吗？”

优先给出简洁示例，不要冗长堆砌。

### Set Appropriate Degrees of Freedom

根据任务的脆弱性和可变性选择细化程度：

- **High freedom（文本级指令）**：存在多种可行解、决策依赖上下文、以启发式为主。
- **Medium freedom（伪代码或带参数脚本）**：有推荐模式，可接受一定变化，配置会影响行为。
- **Low freedom（具体脚本/少量参数）**：操作脆弱易错，或必须严格按固定步骤执行。

想像 Claude 在探索一条道路：悬崖边的窄桥需要护栏（低自由度），开阔草地则可以自行选择路径（高自由度）。

### Anatomy of a Skill

每个 skill 必须包含 SKILL.md，其他资源按需附带：

```
skill-name/
├── SKILL.md (required)
│   ├── YAML frontmatter metadata (name, description)
│   └── Markdown instructions
└── Bundled Resources (optional)
    ├── scripts/
    ├── references/
    └── assets/
```

#### SKILL.md (required)

- **Frontmatter（YAML）**：仅含 `name` 与 `description`。Claude 通过这两项判断何时触发技能，因此要清楚地描述技能作用与触发场景。
- **Body（Markdown）**：说明如何使用技能，仅在技能触发后加载。

#### Bundled Resources (optional)

##### Scripts (`scripts/`)

用于需要确定性可靠或频繁重复编写的代码（Python/Bash 等）。
- **何时添加**：重复编写的代码，或需要高度可靠的任务
- **示例**：`scripts/rotate_pdf.py` 处理 PDF 旋转
- **优势**：节省 tokens，可直接执行；如需因环境差异调整，可再阅读

##### References (`references/`)

在需要时加载的文档或资料，用于指导 Claude 的思考。
- **何时添加**：执行任务时需要查阅的文档
- **示例**：`references/finance.md`（财务 schema）、`references/mnda.md`（公司 NDA）、`references/api_docs.md`（API 文档）
- **用途**：数据库 schema、API 说明、领域知识、公司制度、详细流程
- **优势**：让 SKILL.md 保持轻量，仅在需要时加载
- **最佳实践**：若文件 >10k 字，最好在 SKILL.md 中列出 grep 查询提示
- **避免重复**：信息要么放在 SKILL.md，要么放在 references。除非是核心流程，否则详细资料、schema、示例都应放在 references，以免撑爆上下文。

##### Assets (`assets/`)

不需要加载进上下文，而是在输出中直接使用的文件。
- **何时添加**：技能需要在成果中引用这些文件
- **示例**：`assets/logo.png` 品牌资产、`assets/slides.pptx` 模板、`assets/frontend-template/` HTML/React boilerplate、`assets/font.ttf` 字体
- **用途**：模板、图像、图标、样板代码、字体、示例文件
- **优势**：将输出资源与文档分离，Claude 在产出时直接使用

#### What to Not Include in a Skill

技能只包含直接支持功能的文件，不要创建额外文档，例如：README、INSTALLATION_GUIDE、QUICK_REFERENCE、CHANGELOG 等。避免无关的流程说明、搭建/测试步骤或面向用户的文档，以免增加噪音。

### Progressive Disclosure Design Principle

技能通过三级加载机制控制上下文：

1. **Metadata（name + description）** —— 始终存在（约 100 词）
2. **SKILL.md body** —— 触发技能时加载（<5k 词）
3. **Bundled resources** —— Claude 需要时再加载（脚本可直接执行，不占上下文）

#### Progressive Disclosure Patterns

保持 SKILL.md 内容必要即可，总行数尽量 <500。如接近上限，把细节拆到独立文件，并在 SKILL.md 中明确指向这些文件以及何时需要阅读。

**核心原则**：当技能支持多种变体/框架/选项时，SKILL.md 仅保留核心流程与选择指引；将变体细节（模式、示例、配置）移至 reference。

**Pattern 1：高层指南 + 参考文件**

```markdown
# PDF Processing

## Quick start

Extract text with pdfplumber:
[code example]

## Advanced features

- **Form filling**: See [FORMS.md](FORMS.md)
- **API reference**: See [REFERENCE.md](REFERENCE.md)
- **Examples**: See [EXAMPLES.md](EXAMPLES.md)
```

只有在需要时才加载 FORMS/REFERENCE/EXAMPLES。

**Pattern 2：按领域组织**

```
bigquery-skill/
├── SKILL.md
└── reference/
    ├── finance.md
    ├── sales.md
    ├── product.md
    └── marketing.md
```

当用户询问 sales 指标时，只需读取 sales.md。

支持多框架的技能也可按变体组织：

```
cloud-deploy/
├── SKILL.md
└── references/
    ├── aws.md
    ├── gcp.md
    └── azure.md
```

用户选择 AWS 时，仅读取 aws.md。

**Pattern 3：按需展开细节**

```markdown
# DOCX Processing

## Creating documents

Use docx-js...

## Editing documents

For simple edits, modify XML.

**For tracked changes**: See [REDLINING.md](REDLINING.md)
**For OOXML details**: See [OOXML.md](OOXML.md)
```

Claude 只有在需要处理 tracked changes 时才读取 REDLINING.md。

**重要指南：**
- **避免多层嵌套**：reference 与 SKILL.md 的距离最多一层，全部在 SKILL.md 中直接链接。
- **结构化长文档**：>100 行的 reference 顶部应包含目录，方便 Claude 快速了解范围。

## Skill Creation Process

创建技能需遵循以下步骤：

1. 通过具体示例理解技能
2. 规划可复用内容（scripts、references、assets）
3. 初始化技能目录（运行 `init_skill.py`）
4. 编辑技能（实现资源并编写 SKILL.md）
5. 打包技能（运行 `package_skill.py`）
6. 根据实战反馈迭代

无特殊原因不要跳步，必须按顺序执行。

### Step 1: Understanding the Skill with Concrete Examples

只有在技能用法已绝对清晰时才可跳过，即使是迭代既有技能也应执行。

要构建有效技能，必须搞清楚技能在真实场景中的使用方式。可通过用户提供的示例，或自己生成后再与用户确认。

示例：构建 image-editor 技能时需要弄清楚：

- “该技能应支持哪些操作？编辑、旋转、还有吗？”
- “能给一些具体的请求示例吗？”
- “我猜用户会说 ‘Remove the red-eye from this image’ 或 ‘Rotate this image’，还有其他场景吗？”
- “什么样的用户输入应该触发此技能？”

不要一次问太多问题。先问最关键的，必要时再跟进。

当你清楚技能要支持的功能时，结束此步骤。

### Step 2: Planning the Reusable Skill Contents

基于上述示例，分析：

1. 从零完成该示例需要哪些步骤
2. 若要重复执行，应准备哪些 scripts、references、assets

示例：构建 `pdf-editor`，用户常说 “Help me rotate this PDF”：

1. 每次旋转都要重复写代码
2. 因此需要 `scripts/rotate_pdf.py`

示例：`frontend-webapp-builder` 处理 “Build me a todo app” 等请求：

1. 每次都要搭建相似的 HTML/React boilerplate
2. 需要在 `assets/hello-world/` 保存模板工程

示例：`big-query` 处理 “How many users have logged in today?”：

1. 每次都要重新摸清表结构
2. 需要 `references/schema.md` 记录 schema

通过逐个示例分析，列出技能需包含的可复用资源。

### Step 3: Initializing the Skill

此时开始真正创建技能。

只有在技能已存在、仅需迭代或打包时才能跳过此步骤。

创建全新技能时，务必运行 `init_skill.py`。该脚本会生成包含所有必要结构的模板目录，让创建过程更高效、更可靠。

用法：

```bash
scripts/init_skill.py <skill-name> --path <output-directory>
```

脚本会：
- 在指定路径创建技能目录
- 生成带 frontmatter 与 TODO 占位的 SKILL.md 模板
- 创建 `scripts/`、`references/`、`assets/` 示例目录
- 在各目录放入示例文件，可供定制或删除

初始化后，根据需要定制或移除示例内容。

### Step 4: Edit the Skill

编辑新建或既有技能时，记住它最终是给另一个 Claude 使用的。写入对 Claude 有帮助但不显而易见的信息，思考哪些流程知识、领域细节或可复用资产能帮助其更好完成任务。

#### Learn Proven Design Patterns

根据需求查阅以下指南：

- **Multi-step processes**：见 `references/workflows.md`，了解顺序流程与条件逻辑
- **Specific output formats/quality standards**：见 `references/output-patterns.md`，包含模板与示例

这些文件总结了成熟的技能设计实践。

#### Start with Reusable Skill Contents

实现时先处理前面识别出的 `scripts/`、`references/`、`assets/`。可能需要用户提供素材，例如 `brand-guidelines` 技能需要品牌资产或文档。

新增脚本必须实际运行验证，确保无 bug 且输出正确。如果脚本很多，可抽样测试以在保证可靠性的同时控制耗时。

不需要的示例文件/目录应删除。初始化脚本只是在展示结构，大多数技能并不需要全部示例。

#### Update SKILL.md

**写作准则：** 使用祈使/不定式语气。

##### Frontmatter

编写 `name` 与 `description`：
- `name`：技能名称
- `description`：触发条件，需要说明技能能做什么以及何时使用
  - 把所有 “when to use” 信息写在这里，而不是 body（因为 body 只会在触发后加载）
  - 示例（`docx`）："Comprehensive document creation, editing, and analysis with support for tracked changes, comments, formatting preservation, and text extraction. Use when Claude needs to work with professional documents (.docx files) for: (1) Creating new documents, (2) Modifying or editing content, (3) Working with tracked changes, (4) Adding comments, or any other document tasks"

YAML frontmatter 不要添加其它字段。

##### Body

撰写如何使用技能及其资源的说明。

### Step 5: Packaging a Skill

完成开发后，需要打包成可分发的 `.skill` 文件。打包脚本会先自动校验：

```bash
scripts/package_skill.py <path/to/skill-folder>
```

可指定输出目录：

```bash
scripts/package_skill.py <path/to/skill-folder> ./dist
```

脚本将：

1. **Validate**（校验）
   - YAML frontmatter 格式与必填字段
   - 目录/命名是否合规
   - description 是否完整高质
   - 文件组织与资源引用是否正确

2. **Package**（打包）
   - 校验通过后生成 `my-skill.skill`（本质是 zip）。包含全部文件并保持目录结构

若校验失败，脚本会报告错误并终止。修复后重新运行。

### Step 6: Iterate

在真实任务中使用技能后，用户往往会提出改进意见，通常发生在使用完技能的第一时间。

**迭代流程：**

1. 在真实任务中使用技能
2. 观察困难或低效之处
3. 判断需要更新 SKILL.md 或资源的部分
4. 实施修改并再次测试
