---
name: webapp-testing
description: 使用 Playwright 与本地 Web 应用交互和测试的工具包，可验证前端功能、调试 UI 行为、截取浏览器截图并查看日志。
license: 完整条款见 LICENSE.txt
---

# Web Application Testing

测试本地 Web 应用时，编写原生 Python Playwright 脚本。

**可用辅助脚本：**
- `scripts/with_server.py` —— 管理 server 生命周期（支持多 server）

**务必先以 `--help` 运行脚本** 了解参数。只有在运行后确实需要定制时才阅读源码，这些脚本可能很大，加载会污染上下文。它们设计成黑盒，可直接调用。

## Decision Tree: Choosing Your Approach

```
User task → Is it static HTML?
    ├─ Yes → 直接读取 HTML 文件寻找选择器
    │         ├─ 成功 → 使用这些选择器编写 Playwright 脚本
    │         └─ 失败/不完整 → 按动态应用处理
    │
    └─ No (dynamic webapp) → Server 是否已运行？
        ├─ No → 运行: python scripts/with_server.py --help
        │        然后结合 helper + 精简 Playwright 脚本
        │
        └─ Yes → 侦察后执行：
            1. 导航并等待 networkidle
            2. 截屏或查看 DOM
            3. 从渲染状态中识别选择器
            4. 用找到的选择器执行操作
```

## Example: Using with_server.py

启动 server 时先查看 `--help`，再调用 helper：

**单个 server：**
```bash
python scripts/with_server.py --server "npm run dev" --port 5173 -- python your_automation.py
```

**多个 server（如 backend + frontend）：**
```bash
python scripts/with_server.py \
  --server "cd backend && python server.py" --port 3000 \
  --server "cd frontend && npm run dev" --port 5173 \
  -- python your_automation.py
```

自动化脚本只需编写 Playwright 逻辑（server 已自动管理）：
```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)  # 始终以 headless 模式启动 chromium
    page = browser.new_page()
    page.goto('http://localhost:5173')  # server 已运行
    page.wait_for_load_state('networkidle')  # 关键：等待 JS 执行完
    # ... automation 逻辑
    browser.close()
```

## Reconnaissance-Then-Action Pattern

1. **检查渲染后的 DOM：**
   ```python
   page.screenshot(path='/tmp/inspect.png', full_page=True)
   content = page.content()
   page.locator('button').all()
   ```
2. **根据检查结果确定选择器**
3. **使用这些选择器执行操作**

## Common Pitfall

❌ **不要** 在动态应用中未等待 `networkidle` 就检查 DOM
✅ **要** 调用 `page.wait_for_load_state('networkidle')` 后再检查

## Best Practices

- **把 scripts 当作黑盒**：尽量利用 `scripts/` 中的脚本完成复杂流程。通过 `--help` 查看用法，然后直接调用。
- 使用 `sync_playwright()` 编写同步脚本
- 完成后务必关闭浏览器
- 使用语义清晰的选择器：`text=`、`role=`、CSS 选择器或 ID
- 合理等待：`page.wait_for_selector()` 或 `page.wait_for_timeout()`

## Reference Files

- **examples/** —— 常见模式示例：
  - `element_discovery.py` —— 查找按钮、链接、输入框
  - `static_html_automation.py` —— 针对 `file://` 本地 HTML
  - `console_logging.py` —— 自动化过程中捕获 console log
