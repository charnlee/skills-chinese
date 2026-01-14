---
name: docx
description: “全面的文档创建、编辑与分析能力，支持 tracked changes、comments、格式保留与文本抽取。用于处理专业 .docx 文件：①新建文档；②修改或编辑内容；③处理 tracked changes；④添加 comments；以及其他文档任务。”
license: 专有，完整条款见 LICENSE.txt
---

# DOCX creation, editing, and analysis

## Overview

用户可能要求你创建、编辑或分析 .docx 文件。.docx 实际上是包含 XML 与资源的 ZIP。不同任务对应不同工具与流程。

## Workflow Decision Tree

### Reading/Analyzing Content
使用下方 “Text extraction” 或 “Raw XML access”。

### Creating New Document
遵循 “Creating a new Word document” 流程。

### Editing Existing Document
- **自己创建的文档 + 简单修改** → 使用 “Basic OOXML editing”
- **他人文档** → 使用 **“Redlining workflow”**（推荐默认）
- **法律/学术/商业/政府文档** → 必须使用 **“Redlining workflow”**

## Reading and analyzing content

### Text extraction
若只需阅读文本，用 pandoc 转为 markdown，能很好保留结构并显示 tracked changes：

```bash
# 转 markdown，保留 tracked changes
pandoc --track-changes=all path-to-file.docx -o output.md
# 可选：--track-changes=accept/reject/all
```

### Raw XML access
处理 comments、复杂格式、结构、嵌入媒体或 metadata 时，需要直接读取 XML。

#### Unpacking a file
`python ooxml/scripts/unpack.py <office_file> <output_directory>`

#### Key file structures
* `word/document.xml` —— 主体内容
* `word/comments.xml` —— comment 引用
* `word/media/` —— 图像与媒体
* Tracked changes 通过 `<w:ins>` / `<w:del>`

## Creating a new Word document

从零创建 Word 文档时使用 **docx-js**（JavaScript/TypeScript）。

### Workflow
1. **必须完整阅读** [`docx-js.md`](docx-js.md)（约 500 行）。**不要限制范围**，需获取语法、格式规则与最佳实践。
2. 使用 Document、Paragraph、TextRun 组件编写 JS/TS 文件（依赖如未安装，见文末）
3. 用 `Packer.toBuffer()` 导出 .docx

## Editing an existing Word document

编辑现有文档时使用 **Document library**（Python OOXML 库）。它提供常见操作的高层方法，并可直接访问 DOM。

### Workflow
1. **必须完整阅读** [`ooxml.md`](ooxml.md)（约 600 行），获取 Document library API 与 XML 模式。
2. 解包：`python ooxml/scripts/unpack.py <office_file> <output_directory>`
3. 使用 Document library 编写并运行 Python 脚本（参考 ooxml.md 中 “Document Library”）
4. 打包：`python ooxml/scripts/pack.py <input_directory> <office_file>`

## Redlining workflow for document review

该流程允许先在 markdown 中规划 tracked changes，再落地到 OOXML。**关键**：必须系统性地实现所有变更。

**批处理策略**：把相关修改分成 3-10 条一组，便于调试且效率高，每组完成后再继续。

**原则：精确且最小化**
仅标记真正变化的部分。重复未变化文本不仅难以审查，也显得不专业。将替换拆成：[未变部分] + [删除] + [插入] + [未变部分]。未变文本沿用原 `<w:r>` 的 RSID。

示例：把 “30 days” 改为 “60 days”：
```python
# BAD - 整句替换
'<w:del><w:r><w:delText>The term is 30 days.</w:delText></w:r></w:del><w:ins><w:r><w:t>The term is 60 days.</w:t></w:r></w:ins>'

# GOOD - 仅标记改动，保留原 <w:r>
'<w:r w:rsidR="00AB12CD"><w:t>The term is </w:t></w:r><w:del><w:r><w:delText>30</w:delText></w:r></w:del><w:ins><w:r><w:t>60</w:t></w:r></w:ins><w:r w:rsidR="00AB12CD"><w:t> days.</w:t></w:r>'
```

### Tracked changes workflow

1. **获取 markdown 表示**：
   ```bash
   pandoc --track-changes=all path-to-file.docx -o current.md
   ```

2. **识别并分组修改**：
   - 通过章节号、段落编号、grep 唯一文本、结构描述等定位
   - **不要** 用 markdown 行号（与 XML 不对应）
   - 分批策略：按章节、按类型、按复杂度或按页范围（每批 3-10 条）

3. **阅读文档并解包**：
   - **必须完整阅读** [`ooxml.md`](ooxml.md)，重点关注 “Document Library” 与 “Tracked Change Patterns”
   - `python ooxml/scripts/unpack.py <file.docx> <dir>`
   - 记录脚本输出的建议 RSID，供后续使用

4. **按批实现修改**：
   - 便于调试、支持增量推进、保持效率
   - 可按章节、类型或位置划分

   对每批操作：
   a. **映射文本到 XML**：在 `word/document.xml` 中 grep，确认 `<w:r>` 分段方式
   b. **编写并运行脚本**：使用 `get_node` 查找节点，实现修改后 `doc.save()`，模式参考 ooxml.md
   c. 每次写脚本前都重新 grep `word/document.xml`，因为行号会变化

5. **重新打包**：
   ```bash
   python ooxml/scripts/pack.py unpacked reviewed-document.docx
   ```

6. **最终核对**：
   - 转 markdown：
     ```bash
     pandoc --track-changes=all reviewed-document.docx -o verification.md
     ```
   - 校验：
     ```bash
     grep "original phrase" verification.md      # 应该找不到
     grep "replacement phrase" verification.md   # 应该找到
     ```
   - 确认没有意外改动

## Converting Documents to Images

需要视觉检查可将文档转为图片：

1. **DOCX → PDF**：
   ```bash
   soffice --headless --convert-to pdf document.docx
   ```
2. **PDF → JPEG**：
   ```bash
   pdftoppm -jpeg -r 150 document.pdf page
   ```
   生成 `page-1.jpg` 等。

选项：`-r` 分辨率、`-f/-l` 起止页、最后的 `page` 为前缀。

示例（仅 2-5 页）：
```bash
pdftoppm -jpeg -r 150 -f 2 -l 5 document.pdf page
```

## Code Style Guidelines
**重要**：生成 DOCX 代码时保持精炼，不必使用冗长变量或多余打印。

## Dependencies

所需依赖（如缺失需安装）：
- **pandoc**：`sudo apt-get install pandoc`
- **docx**：`npm install -g docx`
- **LibreOffice**：`sudo apt-get install libreoffice`
- **Poppler**：`sudo apt-get install poppler-utils`
- **defusedxml**：`pip install defusedxml`
