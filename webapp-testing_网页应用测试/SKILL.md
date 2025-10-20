---
name: webapp-testing
description: 使用 Playwright 与本地 Web 应用交互并进行测试的工具包，可验证前端功能、调试 UI 行为、捕获浏览器截图、查看日志。
license: 完整条款见 LICENSE.txt
---

# Web 应用测试

测试本地 Web 应用时，请编写原生 Python Playwright 脚本。

**可用辅助脚本：**
- `scripts/with_server.py` —— 管理服务器生命周期（支持多服务器）

**每次运行脚本前务必先执行 `--help`**。不要在未尝试运行脚本前阅读源码，除非确实需要定制功能。这些脚本可能非常庞大，会污染上下文窗口；它们的设计目标是作为黑盒脚本直接调用，而不是被阅读后加载进上下文。

## 决策树：选择正确策略

```
用户任务 → 是否为静态 HTML？
    ├─ 是 → 直接读取 HTML 文件查找选择器
    │         ├─ 成功 → 使用选择器编写 Playwright 脚本
    │         └─ 失败 / 不完整 → 按动态应用处理（见下）
    │
    └─ 否（动态 Web 应用） → 服务器是否已运行？
        ├─ 否 → 先运行：python scripts/with_server.py --help
        │        使用辅助脚本，并编写简化的 Playwright 脚本
        │
        └─ 是 → 先侦察，后操作：
            1. 导航页面并等待 networkidle
            2. 截图或查看 DOM
            3. 从渲染结果中确定选择器
            4. 使用发现的选择器执行操作
```

## 示例：使用 with_server.py

启动服务器时，先运行 `--help`，再调用辅助脚本：

**单个服务器：**
```bash
python scripts/with_server.py --server "npm run dev" --port 5173 -- python your_automation.py
```

**多个服务器（如后端 + 前端）：**
```bash
python scripts/with_server.py \
  --server "cd backend && python server.py" --port 3000 \
  --server "cd frontend && npm run dev" --port 5173 \
  -- python your_automation.py
```

自动化脚本中只保留 Playwright 逻辑（服务器由辅助脚本管理）：
```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)  # 始终以无头模式启动 chromium
    page = browser.new_page()
    page.goto('http://localhost:5173')  # 服务器已由辅助脚本启动
    page.wait_for_load_state('networkidle')  # 关键：等待 JS 执行完成
    # ... 编写自动化逻辑 ...
    browser.close()
```

## “先侦察，再行动”模式

1. **检查渲染后的 DOM：**
   ```python
   page.screenshot(path='/tmp/inspect.png', full_page=True)
   content = page.content()
   page.locator('button').all()
   ```

2. **根据检查结果确定选择器**

3. **使用选择器执行操作**

## 常见陷阱

❌ **不要** 在动态应用中等待 `networkidle` 之前就检查 DOM  
✅ **要** 先运行 `page.wait_for_load_state('networkidle')` 再进行检查

## 最佳实践

- **将捆绑脚本当作黑盒使用**：在 `scripts/` 中寻找能完成任务的脚本。这些脚本可可靠处理复杂场景且不会污染上下文。先用 `--help` 查看用法，再直接调用。
- 使用 `sync_playwright()` 编写同步脚本
- 任务完成后务必关闭浏览器
- 使用描述性选择器：`text=`、`role=`、CSS 选择器或 ID
- 合理添加等待：`page.wait_for_selector()` 或 `page.wait_for_timeout()`

## 参考文件

- **examples/** —— 常见模式示例：
  - `element_discovery.py` —— 查找页面按钮、链接与输入框
  - `static_html_automation.py` —— 通过 file:// URL 处理本地 HTML
  - `console_logging.py` —— 自动化过程中捕获控制台日志
