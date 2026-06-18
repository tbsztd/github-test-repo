# Codex 网络搜索能力 — 终版测试报告
> 测试日期：2026-06-15 | 环境：Windows 10 / PowerShell  
> 模型：DeepSeek V4 Pro (custom proxy `127.0.0.1:57321`)  
> 工作区：`C:\Users\tbsztdgczp\Desktop\codex_work`

---

## 总览：7 种方法全部测试完毕

| # | 方法 | 初始状态 | 最终状态 | 关键发现 |
|---|------|----------|----------|----------|
| 1 | `web_search` 工具 | ❌ | ❌ 不可修复 | DeepSeek 代理后端不支持此工具 |
| 2 | `curl.exe` 多平台搜索 | ✅ | ✅ | 无需改动，始终可用 |
| 3 | 内置浏览器 (iab) | ⚠️ | ✅ 已跑通 | API 是 `browser.tabs.new()` |
| 4 | Chrome 扩展 | ⚠️ | ✅ 已跑通 | 可认领用户标签页+创建新标签 |
| 5 | Playwright CLI | ⚠️ | ✅ 已安装测试 | 用 `--browser=msedge` |
| 6 | OpenAI Docs MCP | ❌ | ✅ 已安装 | 需重启 Codex 激活 |
| 7 | B站字幕提取 | ⚠️ | ✅ | Python 脚本搜索正常 |

---

## 一、各方法详解

### 1. `web_search` 工具 — ❌ 不可修复

- **原因**：当前使用 DeepSeek V4 Pro 自定义代理（`http://127.0.0.1:57321/v1`），该后端不支持 Codex 的 `web_search` 自定义工具
- **AGENTS.md 佐证**："DeepSeek接入不能用插件；图片/多模态不支持" — 自定义工具属于同样限制
- **如果切换为 OpenAI 模型**，此工具有望直接可用

### 2. `curl.exe` 多平台搜索（multi-source-research Skill）— ✅ 始终可用

经过完整测试，以下平台全部可用：

| 平台 | 命令模板 | 实际测试结果 |
|------|----------|-------------|
| **Bing** | `curl.exe → bing.com/search?q=` | ✅ 98KB HTML，10 个结果块 |
| **GitHub API** | `curl.exe → api.github.com/search/repositories` | ✅ 返回 JSON，含 `oven-sh/bun` (93K⭐) |
| **CSDN RSS** | `curl.exe → blog.csdn.net/{user}/rss/list` | ✅ 15KB XML，20 篇文章 |
| **B站搜索** | `curl.exe → search.bilibili.com/all?keyword=` | ✅ 277KB HTML，标题正确 |
| **百度** | `curl.exe → baidu.com/s?wd=` | ❌ 仅 227B（JS 渲染空壳） |

**推荐指数**：★★★★★ — 零依赖、极速、稳定。百度除外全部可用。

### 3. 内置浏览器插件 (Browser Plugin / iab) — ✅ 已跑通

**问题诊断**：API 不是 `browser.user.newTab()`，正确调用链是：

```js
// Bootstrap（每次新 Node REPL 会话执行一次）
const { setupBrowserRuntime } = await import(".../browser-client.mjs");
await setupBrowserRuntime({ globals: globalThis });
globalThis.browser = await agent.browsers.get("iab");

// 创建标签页并搜索
var tab = await browser.tabs.new();
await tab.goto("https://www.bing.com/search?q=关键词");
var results = await tab.playwright.evaluate(() => { ... });
```

**实测结果**：
- Bing 搜索 → ✅ 返回 5 条结果（"OpenAI Codex CLI 2025"）
- GitHub 搜索 → ✅ 返回 `codex-switcher`、`awesome-codex-cli` 等仓库

**已知 quirk**：`tab.goto()` 可能超时但导航实际成功，用 `waitForTimeout(2000)` 兜底即可。

### 4. Chrome 扩展 — ✅ 已跑通

比内置浏览器更强：可**认领用户已打开的标签页**（复用登录态），也可创建新标签。

```js
// 获取 Chrome 扩展浏览器
var extBrowser = await agent.browsers.get("extension");

// 查看用户打开的标签页
var tabs = await extBrowser.user.openTabs(); // 本次检测到8个标签页
var claimed = await extBrowser.user.claimTab(tabs[0]); // 认领第一个

// 或创建新标签
var newTab = await extBrowser.tabs.new();
await newTab.goto("https://www.bing.com/search?q=...");
```

**实测结果**：
- 成功检测到用户 Edge 中 8 个标签页（B站、QQ邮箱等）
- 成功认领 B站视频页
- 新标签搜索 Bing → ✅ 返回 5 条结果，用后自动关闭

### 5. Playwright CLI — ✅ 已安装并测试

**安装**（由子 Agent Avicenna 完成）：
```powershell
npm install -g @playwright/cli@latest  # v0.1.14
```

**使用**（Chrome 未安装，用 Edge）：
```powershell
npx @playwright/cli@latest open --browser=msedge --headed
npx @playwright/cli@latest goto "https://www.bing.com/search?q=hello+world"
npx @playwright/cli@latest eval "()=>document.title"
npx @playwright/cli@latest close
```

**实测结果**：Bing 搜索成功，返回 "hello world - 搜索" 标题 + 3 条结果

**注意**：PowerShell 中 `eval` 的 JS 函数须压缩为单行；或用 `--filename` 传脚本文件。

### 6. OpenAI Docs MCP — ✅ 已安装（需重启）

**安装**（由子 Agent Plato 完成）：
```powershell
codex mcp add openaiDeveloperDocs --url https://developers.openai.com/mcp
# 输出：Added global MCP server 'openaiDeveloperDocs'.
```

`config.toml` 已自动新增 `[mcp_servers.openaiDeveloperDocs]` 配置段。

**待办**：重启 Codex 后 `mcp__openaiDeveloperDocs__search_openai_docs` 和 `fetch_openai_doc` 工具将可用。

### 7. B站字幕提取 — ✅ Python 脚本正常

**脚本**：`C:\Users\tbsztdgczp\Desktop\codex_work\bilibili_extract.py`（217 行，纯标准库）

**搜索测试**（由子 Agent Parfit 完成）：
- `python bilibili_extract.py "MSPM0"` → ✅ 返回 20 条结果，含 BV号、播放量、UP主
- `python bilibili_extract.py --ccs` → ✅ 3 组预置查询各 5 条结果

---

## 二、搜索方法选择指南

| 场景 | 推荐方法 | 理由 |
|------|----------|------|
| **快速技术搜索** | `curl.exe` → Bing/GitHub | 零延迟，纯终端，可并行 |
| **需要登录态** | Chrome 扩展 `claimTab` | 复用用户浏览器 Cookie |
| **复杂页面交互** | 内置浏览器 (iab) | Playwright API 完整 |
| **批量自动化** | Playwright CLI | 脚本化，headless 可选 |
| **OpenAI 官方文档** | Docs MCP (重启后) | 权威来源，结构化搜索 |
| **B站视频检索** | `bilibili_extract.py` | 专用工具，数据完整 |
| **百度内容搜索** | Bing `site:baidu.com` | 百度反 curl，须间接 |

---

## 三、不可用项与根因

| 项 | 状态 | 根因 |
|----|------|------|
| `web_search` 工具 | ❌ | DeepSeek 代理不支持自定义工具 |
| `curl.exe` → 百度 | ❌ | 百度纯 JS 渲染，curl 只能拿空壳 |

---

## 四、配置变更记录

本次测试对系统做了一处配置变更：

- **`C:\Users\tbsztdgczp\.codex\config.toml`**：新增 `[mcp_servers.openaiDeveloperDocs]` 段（由 `codex mcp add` 自动写入）
- **npm 全局安装**：`@playwright/cli@0.1.14`

无其他文件被修改。

