# DOCX Library Tutorial

使用 JavaScript/TypeScript 生成 .docx 文件的完整指南。

**务必在动手前通读此文。** 各章节覆盖关键格式规则与常见陷阱，跳读会导致文件损坏或渲染异常。

## Setup
假设全局已安装 `docx`，若未安装请运行 `npm install -g docx`。

```javascript
const { Document, Packer, Paragraph, TextRun, Table, TableRow, TableCell, ImageRun, Media, 
        Header, Footer, AlignmentType, PageOrientation, LevelFormat, ExternalHyperlink, 
        InternalHyperlink, TableOfContents, HeadingLevel, BorderStyle, WidthType, TabStopType, 
        TabStopPosition, UnderlineType, ShadingType, VerticalAlign, SymbolRun, PageNumber,
        FootnoteReferenceRun, Footnote, PageBreak } = require('docx');

// Create & Save
const doc = new Document({ sections: [{ children: [/* content */] }] });
Packer.toBuffer(doc).then(buffer => fs.writeFileSync("doc.docx", buffer)); // Node.js
Packer.toBlob(doc).then(blob => { /* download logic */ }); // Browser
```

## Text & Formatting
```javascript
// 关键：不要用 \n 换行，必须用独立 Paragraph
// ❌ new TextRun("Line 1\nLine 2")
// ✅ new Paragraph({ children: [new TextRun("Line 1")] }), new Paragraph({ children: [new TextRun("Line 2")] })

new Paragraph({
  alignment: AlignmentType.CENTER,
  spacing: { before: 200, after: 200 },
  indent: { left: 720, right: 720 },
  children: [
    new TextRun({ text: "Bold", bold: true }),
    new TextRun({ text: "Italic", italics: true }),
    new TextRun({ text: "Underlined", underline: { type: UnderlineType.DOUBLE, color: "FF0000" } }),
    new TextRun({ text: "Colored", color: "FF0000", size: 28, font: "Arial" }),
    new TextRun({ text: "Highlighted", highlight: "yellow" }),
    new TextRun({ text: "Strikethrough", strike: true }),
    new TextRun({ text: "x2", superScript: true }),
    new TextRun({ text: "H2O", subScript: true }),
    new TextRun({ text: "SMALL CAPS", smallCaps: true }),
    new SymbolRun({ char: "2022", font: "Symbol" }),
    new SymbolRun({ char: "00A9", font: "Arial" })
  ]
})
```

## Styles & Professional Formatting
```javascript
const doc = new Document({
  styles: {
    default: { document: { run: { font: "Arial", size: 24 } } },
    paragraphStyles: [
      { id: "Title", name: "Title", basedOn: "Normal",
        run: { size: 56, bold: true, color: "000000", font: "Arial" },
        paragraph: { spacing: { before: 240, after: 120 }, alignment: AlignmentType.CENTER } },
      { id: "Heading1", name: "Heading 1", basedOn: "Normal", next: "Normal", quickFormat: true,
        run: { size: 32, bold: true, color: "000000", font: "Arial" },
        paragraph: { spacing: { before: 240, after: 240 }, outlineLevel: 0 } },
      { id: "Heading2", name: "Heading 2", basedOn: "Normal", next: "Normal", quickFormat: true,
        run: { size: 28, bold: true, color: "000000", font: "Arial" },
        paragraph: { spacing: { before: 180, after: 180 }, outlineLevel: 1 } },
      { id: "myStyle", name: "My Style", basedOn: "Normal",
        run: { size: 28, bold: true, color: "000000" },
        paragraph: { spacing: { after: 120 }, alignment: AlignmentType.CENTER } }
    ],
    characterStyles: [{ id: "myCharStyle", name: "My Char Style",
      run: { color: "FF0000", bold: true, underline: { type: UnderlineType.SINGLE } } }]
  },
  sections: [{
    properties: { page: { margin: { top: 1440, right: 1440, bottom: 1440, left: 1440 } } },
    children: [
      new Paragraph({ heading: HeadingLevel.TITLE, children: [new TextRun("Document Title")] }),
      new Paragraph({ heading: HeadingLevel.HEADING_1, children: [new TextRun("Heading 1")] }),
      new Paragraph({ style: "myStyle", children: [new TextRun("Custom paragraph style")] }),
      new Paragraph({ children: [
        new TextRun("Normal with "),
        new TextRun({ text: "custom char style", style: "myCharStyle" })
      ]})
    ]
  }]
});
```

**推荐字体组合**
- **Arial / Arial**：通用且干净
- **Times New Roman / Arial**：经典衬线 + 现代无衬线
- **Georgia / Verdana**：屏幕阅读友好

**核心准则**
- 用内建 ID（Heading1/2/3 等）覆写 Word 样式
- `HeadingLevel.HEADING_1` 等常量会调用对应 ID
- 设置 `outlineLevel` 以确保 TOC 正常
- 尽量用样式代替内联格式，并通过 `styles.default` 指定默认字体
- 用行距/段前后间距建立层次，颜色少量点缀
- 标准页边距：1440 = 1 inch

## Lists（必须使用真正的列表配置）
```javascript
const doc = new Document({
  numbering: {
    config: [
      { reference: "bullet-list",
        levels: [{ level: 0, format: LevelFormat.BULLET, text: "•", alignment: AlignmentType.LEFT,
          style: { paragraph: { indent: { left: 720, hanging: 360 } } } }] },
      { reference: "first-numbered-list",
        levels: [{ level: 0, format: LevelFormat.DECIMAL, text: "%1.", alignment: AlignmentType.LEFT,
          style: { paragraph: { indent: { left: 720, hanging: 360 } } } }] },
      { reference: "second-numbered-list",
        levels: [{ level: 0, format: LevelFormat.DECIMAL, text: "%1.", alignment: AlignmentType.LEFT,
          style: { paragraph: { indent: { left: 720, hanging: 360 } } } }] }
    ]
  },
  sections: [{
    children: [
      new Paragraph({ numbering: { reference: "bullet-list", level: 0 },
        children: [new TextRun("First bullet point")] }),
      new Paragraph({ numbering: { reference: "bullet-list", level: 0 },
        children: [new TextRun("Second bullet point")] }),
      new Paragraph({ numbering: { reference: "first-numbered-list", level: 0 },
        children: [new TextRun("First numbered item")] }),
      new Paragraph({ numbering: { reference: "first-numbered-list", level: 0 },
        children: [new TextRun("Second numbered item")] }),
      new Paragraph({ numbering: { reference: "second-numbered-list", level: 0 },
        children: [new TextRun("Starts at 1 again (because different reference)")] })
    ]
  }]
});

// 同一 reference = 连续编号；不同 reference = 重新编号
// Unicode 符号造假的列表全部禁止
```

## Tables
```javascript
const tableBorder = { style: BorderStyle.SINGLE, size: 1, color: "CCCCCC" };
const cellBorders = { top: tableBorder, bottom: tableBorder, left: tableBorder, right: tableBorder };

new Table({
  columnWidths: [4680, 4680],
  margins: { top: 100, bottom: 100, left: 180, right: 180 },
  rows: [
    new TableRow({
      tableHeader: true,
      children: [
        new TableCell({
          borders: cellBorders,
          width: { size: 4680, type: WidthType.DXA },
          shading: { fill: "D5E8F0", type: ShadingType.CLEAR },
          verticalAlign: VerticalAlign.CENTER,
          children: [new Paragraph({ alignment: AlignmentType.CENTER,
            children: [new TextRun({ text: "Header", bold: true, size: 22 })] })]
        }),
        new TableCell({
          borders: cellBorders,
          width: { size: 4680, type: WidthType.DXA },
          shading: { fill: "D5E8F0", type: ShadingType.CLEAR },
          children: [new Paragraph({ alignment: AlignmentType.CENTER,
            children: [new TextRun({ text: "Bullet Points", bold: true, size: 22 })] })]
        })
      ]
    }),
    new TableRow({
      children: [
        new TableCell({
          borders: cellBorders,
          width: { size: 4680, type: WidthType.DXA },
          children: [new Paragraph({ children: [new TextRun("Regular data")] })]
        }),
        new TableCell({
          borders: cellBorders,
          width: { size: 4680, type: WidthType.DXA },
          children: [
            new Paragraph({ numbering: { reference: "bullet-list", level: 0 },
              children: [new TextRun("First bullet point")] }),
            new Paragraph({ numbering: { reference: "bullet-list", level: 0 },
              children: [new TextRun("Second bullet point")] })
          ]
        })
      ]
    })
  ]
})
```

要点：列宽需同时在 `columnWidths` 与 `TableCell.width` 设置；DXA 换算 1440=1 inch；边框施加在单元格上；Letter 纸面宽 9360 DXA（1" 边距）。

## Links & Navigation
```javascript
new TableOfContents("Table of Contents", { hyperlink: true, headingStyleRange: "1-3" });

new Paragraph({
  children: [new ExternalHyperlink({
    children: [new TextRun({ text: "Google", style: "Hyperlink" })],
    link: "https://www.google.com"
  })]
});

new Paragraph({
  children: [new InternalHyperlink({
    children: [new TextRun({ text: "Go to Section", style: "Hyperlink" })],
    anchor: "section1"
  })]
});
new Paragraph({ children: [new TextRun("Section Content")], bookmark: { id: "section1", name: "section1" } });
```

## Images & Media
```javascript
new Paragraph({
  alignment: AlignmentType.CENTER,
  children: [new ImageRun({
    type: "png",
    data: fs.readFileSync("image.png"),
    transformation: { width: 200, height: 150, rotation: 0 },
    altText: { title: "Logo", description: "Company logo", name: "Name" }
  })]
})
```

## Page Breaks
```javascript
new Paragraph({ children: [new PageBreak()] });

new Paragraph({ pageBreakBefore: true, children: [new TextRun("This starts on a new page")] });
```
PageBreak 只能放在 Paragraph 中，单独使用会生成无效 XML。

## Headers/Footers & Page Setup
```javascript
const doc = new Document({
  sections: [{
    properties: {
      page: {
        margin: { top: 1440, right: 1440, bottom: 1440, left: 1440 },
        size: { orientation: PageOrientation.LANDSCAPE },
        pageNumbers: { start: 1, formatType: "decimal" }
      }
    },
    headers: {
      default: new Header({ children: [new Paragraph({ alignment: AlignmentType.RIGHT,
        children: [new TextRun("Header Text")] })] })
    },
    footers: {
      default: new Footer({ children: [new Paragraph({ alignment: AlignmentType.CENTER,
        children: [new TextRun("Page "), new TextRun({ children: [PageNumber.CURRENT] }),
          new TextRun(" of "), new TextRun({ children: [PageNumber.TOTAL_PAGES] })] })] })
    },
    children: []
  }]
});
```

## Tabs
```javascript
new Paragraph({
  tabStops: [
    { type: TabStopType.LEFT, position: TabStopPosition.MAX / 4 },
    { type: TabStopType.CENTER, position: TabStopPosition.MAX / 2 },
    { type: TabStopType.RIGHT, position: TabStopPosition.MAX * 3 / 4 }
  ],
  children: [new TextRun("Left\tCenter\tRight")]
});
```

## 常用常量
- 下划线：`SINGLE`、`DOUBLE`、`WAVY`、`DASH`
- 边框：`SINGLE`、`DOUBLE`、`DASHED`、`DOTTED`
- 编号：`DECIMAL`、`UPPER_ROMAN`、`LOWER_LETTER`
- 制表位：`LEFT`、`CENTER`、`RIGHT`、`DECIMAL`
- 常用符号：`"2022"`(•)、`"00A9"`(©)、`"00AE"`(®)、`"2122"`(™)、`"00B0"`(°)、`"F070"`(✓)、`"F0FC"`(✗)

## 常见错误总结
- PageBreak 必须置于 Paragraph 中
- 表格底色必须 `ShadingType.CLEAR`
- 度量单位为 DXA；每个单元格至少包含一个 Paragraph
- 默认字体建议设为 Arial，保持层级
- 表格需设置列宽数组 + 单元格宽度
- 禁用 Unicode 项符，统一使用 numbering 配置
- 禁用 `\n` 换行
- Paragraph 只能包含 TextRun/ImageRun 等子元素，勿直接写字符串
- ImageRun 必填 `type`
- Bullet 需 `LevelFormat.BULLET` 常量，并配置 `text: "•"`
- 不同 numbered list reference 互相独立；需要重新编号时换 reference
- TOC 使用 HeadingLevel 段落，勿叠加自定义 style
- Table margin 统一在表格级别设定以保持一致
