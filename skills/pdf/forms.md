**重要：必须严格按顺序完成以下步骤，先执行分析再写代码。**

当需要填写 PDF 表单时，先确认文件是否包含可填充字段。在本目录运行：
`python scripts/check_fillable_fields <file.pdf>`
根据输出选择“Fillable fields”或“Non-fillable fields”部分继续。

# Fillable fields
若 PDF 具有可填充字段：
- 在本目录运行：`python scripts/extract_form_field_info.py <input.pdf> <field_info.json>`。该脚本会生成字段列表 JSON，格式如下：
```
[
  {
    "field_id": …,          // 字段唯一 ID
    "page": …,              // 页面编号（从 1 开始）
    "rect": [left, bottom, right, top], // PDF 坐标系，y=0 为页面底部
    "type": "text" | "checkbox" | "radio_group" | "choice"
  },
  // checkbox 额外包含 checked/unchecked 值
  {
    "type": "checkbox",
    "checked_value": …,
    "unchecked_value": …
  },
  // radio_group 列出所有选项
  {
    "type": "radio_group",
    "radio_options": [
      {
        "value": …,
        "rect": …
      }
    ]
  },
  // 下拉/多选
  {
    "type": "choice",
    "choice_options": [
      { "value": …, "text": … }
    ]
  }
]
```
- 将 PDF 转成逐页 PNG：
`python scripts/convert_pdf_to_images.py <file.pdf> <output_directory>`
结合图像与 bounding box（注意将 PDF 坐标转换为图像坐标），判断每个字段含义。
- 按下列格式创建 `field_values.json`，写入要填的值：
```
[
  {
    "field_id": "last_name", // 必须与 field_info.json 对应
    "description": "用户姓氏",
    "page": 1,
    "value": "Simpson"
  },
  {
    "field_id": "Checkbox12",
    "description": "若用户年满 18 岁则勾选",
    "page": 1,
    "value": "/On" // checkbox 使用 checked_value；radio 取 radio_options 的 value
  }
]
```
- 运行 `fill_fillable_fields.py` 生成填写后的 PDF：
`python scripts/fill_fillable_fields.py <input pdf> <field_values.json> <output pdf>`
脚本会校验字段 ID 与取值，若报错请修正后重试。

# Non-fillable fields
若 PDF 不支持表单，需要手动确定文本位置并创建注释。必须严格执行以下步骤：
1. 将 PDF 转成 PNG 并确定 bounding box。
2. 创建字段信息 JSON 与验证图。
3. 验证 bounding box。
4. 根据 bounding box 写入内容。

## Step 1: Visual Analysis（必做）
- 运行：`python scripts/convert_pdf_to_images.py <file.pdf> <output_directory>`
- 仔细检查每页 PNG，找出所有需填写的区域。对于每个字段，分别确定 label 区域与输入区域，两者不得重叠。输入框需完整覆盖将要写入的文本。
- 常见结构示例：
  - 标签在框内：输入区域通常在标签右侧直至边框。
  - 标签后跟下划线：输入区域应覆盖整条线的宽度并位于其上方。
  - 标签在下方：输入区域位于线条之上（常见于签名/日期）。
  - 标签在上方：输入区域从标签下缘到线条上缘。
  - 复选框：识别正方形方框，输入 bounding box 只覆盖方框本身，不包含文字。

## Step 2: 创建 fields.json 与验证图（必做）
- `fields.json` 格式：
```
{
  "pages": [
    {"page_number": 1, "image_width": …, "image_height": …},
    {"page_number": 2, …}
  ],
  "form_fields": [
    {
      "page_number": 1,
      "description": "用户姓氏",
      "field_label": "Last name",
      "label_bounding_box": [30,125,95,142],
      "entry_bounding_box": [100,125,280,142],
      "entry_text": {
        "text": "Johnson",
        "font_size": 14,
        "font_color": "000000"
      }
    },
    {
      "page_number": 2,
      "description": "需勾选的复选框",
      "field_label": "Yes",
      "label_bounding_box": [100,525,132,540],
      "entry_bounding_box": [140,525,155,540],
      "entry_text": { "text": "X" }
    }
  ]
}
```
- 生成验证图（红框=输入区域、蓝框=标签）：
`python scripts/create_validation_image.py <page_number> <path_to_fields.json> <input_image> <output_image>`

## Step 3: 验证 Bounding Boxes（必做）
### 自动检查
- 运行：`python scripts/check_bounding_boxes.py <fields.json>`
确保无交集且输入框足够容纳文本。

### 手动检查
- 必须人工检查验证图：红框只能覆盖输入区，且不得包含文字；蓝框需覆盖标签文字。
- 复选框：红框需居中覆盖方框，蓝框覆盖标签文字。
- 若有误，修改 fields.json、重新生成验证图并复检，直至完全准确。

## Step 4: 将注释放入 PDF
- 在本目录运行：
`python scripts/fill_pdf_form_with_annotations.py <input_pdf> <path_to_fields.json> <output_pdf>`
该脚本会依据 fields.json 在对应位置生成填写结果。
