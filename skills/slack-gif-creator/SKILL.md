---
name: slack-gif-creator
description: 一套面向 Slack 的动图创作知识与工具，包含约束条件、验证工具与动画概念。当用户请求为 Slack 制作动图（如 “make me a GIF of X doing Y for Slack”）时使用。
license: 完整条款见 LICENSE.txt
---

# Slack GIF Creator

一个帮助创作符合 Slack 要求的动图工具包。

## Slack 要求

**尺寸：**
- Emoji GIF：建议 128x128
- Message GIF：480x480

**参数：**
- FPS：10-30（越低文件越小）
- Colors：48-128（越少文件越小）
- Duration：Emoji GIF 应控制在 3 秒以内

## 核心流程

```python
from core.gif_builder import GIFBuilder
from PIL import Image, ImageDraw

# 1. 创建 builder
builder = GIFBuilder(width=128, height=128, fps=10)

# 2. 生成帧
for i in range(12):
    frame = Image.new('RGB', (128, 128), (240, 248, 255))
    draw = ImageDraw.Draw(frame)

    # 使用 PIL 图元绘制动画
    # （圆形、多边形、直线等）

    builder.add_frame(frame)

# 3. 以优化设置保存
builder.save('output.gif', num_colors=48, optimize_for_emoji=True)
```

## 绘制图形

### 处理用户上传的图像
若用户上传了图像，需要判断其是想：
- **直接使用**（如 “animate this” 或 “split this into frames”）
- **作为灵感**（如 “make something like this”）

使用 PIL 载入：
```python
from PIL import Image

uploaded = Image.open('file.png')
# 可直接使用，或仅作为风格/配色参考
```

### 从零绘制
从头绘制时，使用 PIL ImageDraw 提供的图元：

```python
from PIL import ImageDraw

draw = ImageDraw.Draw(frame)

# 圆形/椭圆
draw.ellipse([x1, y1, x2, y2], fill=(r, g, b), outline=(r, g, b), width=3)

# 星形、三角形等多边形
points = [(x1, y1), (x2, y2), (x3, y3), ...]
draw.polygon(points, fill=(r, g, b), outline=(r, g, b), width=3)

# 直线
draw.line([(x1, y1), (x2, y2)], fill=(r, g, b), width=5)

# 矩形
draw.rectangle([x1, y1, x2, y2], fill=(r, g, b), outline=(r, g, b), width=3)
```

**不要使用：** Emoji 字体（跨平台不可靠），也不要假设本技能自带现成图像素材。

### 让图形更出彩

图形必须精致、有创意而非基础占位。技巧包括：

**使用更粗的线条** —— 轮廓与线条 `width` 至少为 2，过细会显得粗糙。

**增加视觉深度**：
- 使用 `create_gradient_background` 绘制渐变背景
- 叠加多层形状（如大星星内嵌小星星）营造复杂度

**让形状更有趣**：
- 不要只画纯色圆，可加高光、环状装饰或图案
- 星形可以添加发光效果（在后面绘制更大的半透明形状）
- 组合多个形状（星星+闪光、圆形+环形）

**注意配色**：
- 使用鲜明且互补的颜色
- 通过深浅对比突出轮廓
- 关注整体构图

**复杂形状**（心形、雪花等）：
- 结合多边形与椭圆
- 精准计算坐标保持对称
- 添加细节（心形的高光弧线、雪花的枝杈）

务必发挥创意与细节。一张优秀的 Slack GIF 应该看起来像精心创作，而非临时素材。

## 可用工具

### GIFBuilder（`core.gif_builder`）
汇集帧并进行 Slack 优化：
```python
builder = GIFBuilder(width=128, height=128, fps=10)
builder.add_frame(frame)          # 加入单帧 PIL Image
builder.add_frames(frames)        # 加入帧列表
builder.save('out.gif', num_colors=48, optimize_for_emoji=True, remove_duplicates=True)
```

### Validators（`core.validators`）
检查 GIF 是否满足 Slack 要求：
```python
from core.validators import validate_gif, is_slack_ready

# 详细校验
passes, info = validate_gif('my.gif', is_emoji=True, verbose=True)

# 快速检查
if is_slack_ready('my.gif'):
    print("Ready!")
```

### Easing Functions（`core.easing`）
使用平滑的运动曲线：
```python
from core.easing import interpolate

# 进度 0.0 → 1.0
t = i / (num_frames - 1)

# 应用 easing
y = interpolate(start=0, end=400, t=t, easing='ease_out')

# 可选：linear, ease_in, ease_out, ease_in_out,
#       bounce_out, elastic_out, back_out
```

### Frame Helpers（`core.frame_composer`）
常用辅助函数：
```python
from core.frame_composer import (
    create_blank_frame,          # 纯色背景
    create_gradient_background,  # 垂直渐变
    draw_circle,                 # 圆形辅助
    draw_text,                   # 基本文字渲染
    draw_star                    # 五角星
)
```

## 动画概念

### Shake/Vibrate
用振荡偏移位置：
- 搭配帧序号使用 `math.sin()` 或 `math.cos()`
- 加入小幅随机扰动提升自然感
- 可作用于 x 与/或 y

### Pulse/Heartbeat
节奏性缩放：
- 使用 `math.sin(t * frequency * 2 * math.pi)` 获得平滑脉冲
- 心跳：两个快速脉冲后停顿（调整正弦波）
- 通常在 0.8~1.2 倍之间缩放

### Bounce
物体下落并弹跳：
- 着陆阶段使用 `interpolate()` 并设 `easing='bounce_out'`
- 下落阶段使用 `easing='ease_in'`
- 每帧增加 y 方向速度模拟重力

### Spin/Rotate
围绕中心旋转：
- PIL：`image.rotate(angle, resample=Image.BICUBIC)`
- 若想摇摆，用正弦波控制角度

### Fade In/Out
渐显或渐隐：
- 创建 RGBA 图像并调整 alpha 通道
- 或用 `Image.blend(image1, image2, alpha)`
- Fade in：alpha 从 0 到 1
- Fade out：alpha 从 1 到 0

### Slide
把对象从画面外滑入：
- 起始位置在画布外
- 结束位置为目标坐标
- 使用 `interpolate()` 并设 `easing='ease_out'` 使停止更平滑
- 要有回弹可用 `easing='back_out'`

### Zoom
缩放并重新定位实现推进：
- Zoom in：从 0.1 放大到 2.0，同时裁剪中心
- Zoom out：从 2.0 缩回 1.0
- 可加 Motion Blur（PIL 滤镜）增强戏剧感

### Explode/Particle Burst
创建向外扩散的粒子：
- 为粒子生成随机角度与速度
- 每帧更新：`x += vx`，`y += vy`
- 加入重力：`vy += gravity_constant`
- 随时间降低透明度实现淡出

## 优化策略

只有在用户明确要求减小文件大小时，才采用下述方法：

1. **减少帧数** —— 降低 FPS（例如 20→10）或缩短时长
2. **减少颜色** —— 使用 `num_colors=48` 代替 128
3. **缩小尺寸** —— 由 480x480 改为 128x128
4. **移除重复帧** —— 保存时设 `remove_duplicates=True`
5. **Emoji 模式** —— `optimize_for_emoji=True` 自动优化

```python
# Emoji 场景的极限优化
builder.save(
    'emoji.gif',
    num_colors=48,
    optimize_for_emoji=True,
    remove_duplicates=True
)
```

## 指导思想

本技能提供：
- **知识**：Slack 约束与动画概念
- **工具**：GIFBuilder、validators、easing 函数
- **灵活度**：通过 PIL 图元自定义动画逻辑

但不提供：
- 刻板模板或预制动画函数
- Emoji 字体渲染（跨平台不稳定）
- 预打包的素材库

**关于用户上传**：若用户提供图像，可用 PIL 读取，并根据请求判断是要直接使用还是仅作为灵感。

发挥创意！组合多种概念（如 bounce + rotate、pulse + slide），充分利用 PIL 能力。

## 依赖

```bash
pip install pillow imageio numpy
```
