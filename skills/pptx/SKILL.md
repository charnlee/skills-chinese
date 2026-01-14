---
name: pptx
description: “演示文稿（.pptx）创建、编辑与分析。用于：①新建演示文稿；②修改或编辑内容；③处理 layouts；④添加 comments 或 speaker notes；以及其他 PPTX 任务。”
license: 专有，完整条款见 LICENSE.txt
---

# PPTX creation, editing, and analysis

## Overview

用户可能要求创建、编辑或分析 .pptx。PPTX 本质是 ZIP，包含 XML 与媒体。不同任务对应不同工具与工作流。

## Reading and analyzing content

### Text extraction
若只需查看文本，可用 markitdown 转为 markdown：
```bash
python -m markitdown path-to-file.pptx
```

### Raw XML access
处理 comments、speaker notes、slide layouts、animations、design elements、复杂格式等时需读取原始 XML。

#### Unpacking a file
`python ooxml/scripts/unpack.py <office_file> <output_dir>`
（如路径不同，可 `find . -name "unpack.py"`）

#### Key file structures
* `ppt/presentation.xml` —— 元数据与 slide 引用
* `ppt/slides/slide{N}.xml` —— 各 slide 内容
* `ppt/notesSlides/notesSlide{N}.xml` —— speaker notes
* `ppt/comments/modernComment_*.xml` —— comments
* `ppt/slideLayouts/` —— 布局模板
* `ppt/slideMasters/` —— 母版
* `ppt/theme/` —— 主题信息
* `ppt/media/` —— 媒体文件

#### Typography / Color
仿制示例设计时：
1. 查 `ppt/theme/theme1.xml` 中 `<a:clrScheme>`、`<a:fontScheme>`
2. 查 `ppt/slides/slide1.xml` 中 `<a:rPr>` 获取实际字体/颜色
3. grep `<a:solidFill>`、`<a:srgbClr>` 等找颜色引用

## Creating a new PowerPoint presentation (no template)

从零创建演示时使用 **html2pptx**：将 HTML 精确转成 PPT。

### Design Principles

**在写代码前先说明设计策略**：
1. 内容主题、语气、行业
2. 品牌要求
3. 选色理由

**要求**：
- 先描述设计再写代码
- 仅用 web-safe fonts（Arial、Helvetica、Times New Roman、Georgia、Courier New、Verdana、Tahoma、Trebuchet MS、Impact）
- 通过字号/粗细/颜色建立层次
- 确保对比、字号、对齐
- 保持一致性

#### Color Palette
- 不要默认色，结合内容挑选
- 兼顾行业/受众/能量
- 组建 3-5 色调色板，含主色/辅助/强调
- 确保可读对比
- 可参考文中的多套示例色板（Classic Blue、Teal & Coral、Bold Red 等）

#### Visual Details Options
- 几何：斜向分割、非对称列宽、旋转标题、几何框、三角点缀、叠层
- 边框：单侧粗线、双线、角括、L 形、加粗下划
- 字体：极致字号对比、全大写加间距、编号大字、Courier New 表达技术数据、Arial Narrow 处理密集信息、描边强调

### html2pptx Workflow
1. 使用提供的 HTML/React 模板编写幻灯片内容
2. 运行 html2pptx 渲染脚本（依赖 pptxgenjs、playwright、react-icons、sharp 等）生成 PPT
3. 保持 HTML 与 PPT 的元素位置一致
4. 输出最终 .pptx 并说明设计

## Template-Driven Workflow（推荐处理品牌模板）

当用户提供模板或需要统一品牌时，遵循以下步骤：

### Step 1: Template Recon
- 运行 `python scripts/thumbnail.py template.pptx [prefix]` 生成缩略图（默认 5 列、每图最多 30 页，可用 `--cols 3-6` 调整）
- 快速了解布局、占位符数量、视觉节奏

### Step 2: Slide Inventory
- 使用 `scripts/inventory_template.py` 生成 `template-slides.md`（或读取 `reference/template-inventory.md`）
- 文件需列出每张幻灯片、类别、布局代码、用途，并记录其中的 placeholders（标题/正文/图片等）
- 这是选择模板的依据，必不可少

### Step 3: Outline + Template Mapping
- 审视可用模板，匹配实际内容
- **严格匹配内容数量与 placeholder**：
  - 单列布局用于单一叙事
  - 双列适用于两项内容
  - 三列适用于三项内容
  - 图文布局仅在确有图像时使用
  - 引用布局只用于真实引用（含来源）
  - 不要硬塞内容或留下空 placeholder
- 统计内容后再选布局，确保每个 placeholder 有意义
- 在 `outline.md` 中记录内容与模板索引（0-based），如：
  ```
  template_mapping = [0, 34, 34, 50, 54]
  ```

### Step 4: Rearrange Slides
- 使用 `python scripts/rearrange.py template.pptx working.pptx 0,34,34,50,52`
- 脚本会复制需要的 slide、删除多余、按索引重排。索引从 0 开始，可重复表示复制

### Step 5: Extract Text Inventory
- `python scripts/inventory.py working.pptx text-inventory.json`
- 必须完整阅读 `text-inventory.json`，了解每张幻灯片的 shapes：
  - 命名如 `slide-0/shape-1`
  - 包含 placeholder_type（TITLE、BODY 等）、位置尺寸、默认字体大小、段落文本及属性（bullet、level、alignment、spacing、font、color）
  - SLIDE_NUMBER 会被过滤

### Step 6: Prepare Replacement JSON
- 仅引用 inventory 中存在的 shapes；脚本会校验
- 所有 shapes 若未提供 `paragraphs` 将在替换时被清空
- 对需要内容的 shape 添加 `paragraphs` 数组，并复制必要属性：
  - 标题通常 `bold: true`
  - 列表需 `"bullet": true, "level": 0`（level 必填）
  - 保留 alignment、font、color、line spacing 等关键属性
  - bullet 文本不要加入符号（脚本会自动添加）
- 根据 shape 尺寸控制文本长度
- 将结果保存为 `replacement-text.json`

示例：
```json
"paragraphs": [
  {"text": "New presentation title", "alignment": "CENTER", "bold": true},
  {"text": "Section header", "bold": true},
  {"text": "First bullet", "bullet": true, "level": 0},
  {"text": "Red text", "color": "FF0000"},
  {"text": "Theme text", "theme_color": "DARK_1"}
]
```

### Step 7: Apply Replacement
- `python scripts/replace.py working.pptx replacement-text.json output.pptx`
- 脚本将：
  - 重新提取 inventory
  - 校验 JSON 中的 slide/shape
  - 清空所有文本
  - 仅对提供 paragraphs 的 shape 写入新内容并保留格式
  - 处理 bullets、alignment、font、color
  - 如引用不存在的 shape 或文本溢出会报错

## Creating Thumbnail Grids

`python scripts/thumbnail.py deck.pptx [output_prefix] --cols 4` 可快速生成缩略图，便于审阅设计、确定布局与导航。

## Converting Slides to Images

1. `soffice --headless --convert-to pdf deck.pptx`
2. `pdftoppm -jpeg -r 150 deck.pdf slide`
   - `-r` 控制 DPI，`-f/-l` 指定范围，`slide` 为输出前缀

## Code Style Guidelines

编写 PPTX 相关脚本时保持简洁：
- 精炼变量
- 避免冗余操作与 print

## Dependencies

- `markitdown[pptx]`：`pip install "markitdown[pptx]"`
- `pptxgenjs`：`npm install -g pptxgenjs`
- `playwright`：`npm install -g playwright`
- `react-icons react react-dom`：`npm install -g react-icons react react-dom`
- `sharp`：`npm install -g sharp`
- **LibreOffice**：`sudo apt-get install libreoffice`
- **Poppler**：`sudo apt-get install poppler-utils`
- **defusedxml**：`pip install defusedxml`
