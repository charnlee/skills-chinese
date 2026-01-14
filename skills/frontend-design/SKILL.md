---
name: frontend-design
description: 创建具有高设计水准、可投入生产的前端界面。用户要求构建 web components、pages、artifacts、posters 或 applications 时使用（包括 websites、landing pages、dashboards、React components、HTML/CSS layouts，以及任何 web UI 的美化）。输出富有创意、精致的代码与 UI 设计，避免通用 AI 风格。
license: 完整条款见 LICENSE.txt
---

此技能用于指导打造具有鲜明个性、可真实运行的前端界面，杜绝套路化的 “AI slop”。实现能够运行的代码，同时在美学细节与创意选择上做到极致。

用户会提供要构建的前端需求：组件、页面、应用或界面，并可能附带目的、受众或技术约束。

## 设计思考

在动手写代码前，先理解上下文，并选定一个大胆的美学方向：
- **Purpose**：界面解决什么问题？谁来使用？
- **Tone**：选择极致风格：brutally minimal、maximalist chaos、retro-futuristic、organic/natural、luxury/refined、playful/toy-like、editorial/magazine、brutalist/raw、art deco/geometric、soft/pastel、industrial/utilitarian 等。它们只是灵感，关键是设计一个忠于所选风格的成品。
- **Constraints**：技术需求（框架、性能、accessibility）。
- **Differentiation**：如何做到令人难忘？别人会记住什么核心亮点？

**关键**：确定一个清晰的概念方向，并精准执行。无论是张扬的 maximalism 还是克制的 minimalism，重点是意图明确。

随后实现可运行的代码（HTML/CSS/JS、React、Vue 等），必须：
- 达到生产级、功能完整
- 视觉上引人注目且独特
- 保持统一且有明确审美观点
- 每一处细节都经过精雕细琢

## Frontend Aesthetics Guidelines

重点关注：
- **Typography**：选择独特、有个性的字体。避免 Arial、Inter 等常见字体，使用能提升整体气质的组合；可以将独特 display font 与精致 body font 搭配。
- **Color & Theme**：坚持统一主题，用 CSS variables 保持一致性。鲜明的主色与锐利的点缀优于平均分配的配色。
- **Motion**：利用动画与微交互。HTML 尽量使用纯 CSS，React 可用 Motion library。聚焦关键时刻：一次协调的入场动画加上 `animation-delay` 的阶梯，比零散的微交互更令人惊喜；尝试 scroll-trigger、hover 态等意想不到的反馈。
- **Spatial Composition**：尝试非传统布局。非对称、重叠、对角线动势、打破网格、充足留白或者高度密集都可以。
- **Backgrounds & Visual Details**：营造氛围而不是纯色背景。添加与整体风格匹配的质感，例如 gradient meshes、noise textures、geometric patterns、图层透明、戏剧化阴影、装饰性边框、custom cursor、grain overlay 等。

不要使用通用 AI 设计：避开滥用的字体（Inter、Roboto、Arial、system fonts）、俗套的配色（尤其是白底紫渐变）、可预期的布局与组件模式，以及缺乏上下文个性的模板化设计。

保持创造力，做出意想不到的选择，让设计与场景真正匹配。不同需求要呈现不同主题：明暗配色、字体、整体风格都要变化，绝不能总是收敛到常见方案（例如 Space Grotesk）。

**重要**：实现复杂度需与审美愿景匹配。Maximalist 需要复杂的代码、丰富动画与特效；Minimalist/Refined 需要克制、精准，以及在留白、字距、细节上的细腻处理。优雅来自对愿景的高质量执行。

记住：Claude 拥有卓越的创意能力。放开手脚，展示当我们跳出框架、坚定追求独特愿景时，前端设计可以达到的高度。
