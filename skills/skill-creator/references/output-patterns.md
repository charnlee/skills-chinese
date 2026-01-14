# Output Patterns

当技能需要输出一致且高质量的结果时，可使用以下模式。

## Template Pattern

提供输出模板，并根据需求决定严格程度。

**严格要求（如 API 响应或结构化数据）示例：**

```markdown
## Report structure

务必遵循以下模板：

# [Analysis Title]

## Executive summary
[一段式概述关键发现]

## Key findings
- 发现 1 + 支持数据
- 发现 2 + 支持数据
- 发现 3 + 支持数据

## Recommendations
1. 可执行建议
2. 可执行建议
```

**弹性指导（允许按情境调整）示例：**

```markdown
## Report structure

以下为默认结构，可视情况调整：

# [Analysis Title]

## Executive summary
[概览]

## Key findings
[依据发现自定义段落]

## Recommendations
[结合上下文提出建议]

需要时可增减章节。
```

## Examples Pattern

当输出质量依赖示例时，提供输入/输出对：

```markdown
## Commit message format

生成提交信息时遵循下列示例：

**Example 1:**
Input: Added user authentication with JWT tokens
Output:
```
feat(auth): implement JWT-based authentication

Add login endpoint and token validation middleware
```

**Example 2:**
Input: Fixed bug where dates displayed incorrectly in reports
Output:
```
fix(reports): correct date formatting in timezone conversion

Use UTC timestamps consistently across report generation
```

格式为 type(scope): 简述，随后为详细说明。
```

示例能比纯文字描述更清晰地传达风格与细节层级。
