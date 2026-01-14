# Workflow Patterns

## Sequential Workflows

针对复杂任务，可在 SKILL.md 开头给 Claude 一个整体流程，将操作拆成明确顺序步骤：

```markdown
填写 PDF 表单包含以下步骤：

1. 分析表单（运行 analyze_form.py）
2. 创建字段映射（编辑 fields.json）
3. 验证映射（运行 validate_fields.py）
4. 填写表单（运行 fill_form.py）
5. 核对输出（运行 verify_output.py）
```

## Conditional Workflows

若任务存在分支逻辑，需要引导 Claude 按决策点选择流程：

```markdown
1. 判断修改类型：
   **创建全新内容？** → 按下方“Creation workflow”
   **编辑现有内容？** → 按下方“Editing workflow”

2. Creation workflow：[…步骤…]
3. Editing workflow：[…步骤…]
```
