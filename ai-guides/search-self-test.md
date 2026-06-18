# Codex 搜索能力总结（新对话自测用）
> 2026-06-15 多环境实测 | 复制到新对话让 AI 逐条验证

---

## 可用方法一览

| # | 方法 | 执行工具 | 速度 | 适用 |
|---|------|----------|------|------|
| 1 | `curl.exe` → Bing | `shell_command` | 2s | 通用网页搜索 |
| 2 | `curl.exe` → GitHub API | `shell_command` | 1s | 搜仓库/Issue |
| 3 | `curl.exe` → CSDN RSS | `shell_command` | 1s | CSDN 文章 |
| 4 | `curl.exe` → B站搜索 | `shell_command` | 2s | B站视频 |
| 5 | 内置浏览器 iab | `mcp__node_repl__js` | 5s | JS 渲染页面 |
| 6 | Chrome 扩展 | `mcp__node_repl__js` | 5s | 复用登录态 |
| 7 | Playwright CLI | `shell_command` | 5s | 终端浏览器 |
| 8 | `bilibili_extract.py` | `shell_command` | 2s | B站结构化数据 |

不可用：`web_search`（DeepSeek 代理不支持）、`curl` → 百度（JS 空壳）、`curl` → Google（CAPTCHA）

---

## 方法一：curl.exe → Bing（首选，最稳定）

```powershell
curl.exe -s -L --max-time 15 -H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36" "https://www.bing.com/search?q=Codex+CLI&setlang=zh-cn&count=10" -o "$env:TEMP\bing_test.html"
```
验证：文件 > 50KB 且含 `<li class="b_algo"` 即成功。
失败降级：`Invoke-RestMethod -Uri "https://www.bing.com/search?q=Codex+CLI" -TimeoutSec 10`

## 方法二：curl.exe → GitHub API

```powershell
# 搜仓库
curl.exe -s -L --max-time 15 -H "Accept: application/vnd.github+json" -H "User-Agent: Codex-CLI" "https://api.github.com/search/repositories?q=codex+cli&sort=stars&per_page=3"
# 读文件 — 优先 raw
curl.exe -s -L --max-time 10 -H "User-Agent: Codex-CLI" "https://raw.githubusercontent.com/openai/codex/main/README.md"
```
raw 被墙(HTTP_CODE:000)降级：用 API Contents 端点 → `"https://api.github.com/repos/openai/codex/contents/README.md"` → content 字段 base64 解码

## 方法三/四：CSDN RSS / B站搜索

```powershell
# CSDN
curl.exe -s -L --max-time 10 -H "User-Agent: Mozilla/5.0" "https://blog.csdn.net/csdnnews/rss/list"
# B站
curl.exe -s -L --max-time 15 -H "User-Agent: Chrome/131.0.0.0 Safari/537.36" "https://search.bilibili.com/all?keyword=MSPM0&order=totalrank"
```

## 方法五：内置浏览器 iab

Bootstrap（新 Node REPL 会话执行一次）：
```js
var { setupBrowserRuntime } = await import("C:/Users/tbsztdgczp/.codex/plugins/cache/openai-bundled/browser/26.602.40724/scripts/browser-client.mjs");
await setupBrowserRuntime({ globals: globalThis });
globalThis.browser = await agent.browsers.get("iab");
```
搜索：
```js
var tab = await browser.tabs.new();
try { await tab.goto("https://www.bing.com/search?q=Codex+CLI&setlang=en&count=5"); } catch(e) {}
await tab.playwright.waitForTimeout(2000);
var results = await tab.playwright.evaluate(() => { var items = document.querySelectorAll('.b_algo h2'); return Array.from(items).slice(0,5).map(h => h.textContent.trim()); });
```
- goto 超时是正常的，导航实际成功了
- 报 "already been declared" → 先 `mcp__node_repl__js_reset`
- 报 error -3 / WebView2 → iab 不可用，降级 curl
- 路径版本号 26.602.40724 如果变了 → `Get-ChildItem "$env:USERPROFILE\.codex\plugins\cache\openai-bundled\browser" -Recurse -Filter browser-client.mjs` 查新路径

## 方法六：Chrome 扩展

```js
var ext = await agent.browsers.get("extension");
var userTabs = await ext.user.openTabs(); // 查看用户标签
var newTab = await ext.tabs.new();
try { await newTab.goto("https://www.bing.com/search?q=Codex+CLI"); } catch(e) {}
await newTab.playwright.waitForTimeout(2000);
var text = await newTab.playwright.evaluate(() => document.body.innerText);
```

## 方法七：Playwright CLI

```powershell
npx @playwright/cli@latest open --browser=msedge --headed
npx @playwright/cli@latest goto "https://www.bing.com/search?q=Codex+CLI"
npx @playwright/cli@latest eval "()=>document.body.innerText"
npx @playwright/cli@latest close
```

## 方法八：bilibili_extract.py

```powershell
cd C:\Users\tbsztdgczp\Desktop\codex_work
& "D:\Python\python.exe" -X utf8 bilibili_extract.py "MSPM0"
```

---

## 自测清单（逐一验证）

| # | 测试项 | 成功标志 |
|---|--------|---------|
| 1 | curl → Bing | HTML > 50KB，含 `b_algo` |
| 2 | curl → GitHub API | 返回 JSON，含 `total_count` |
| 3 | curl → GitHub Raw | HTTP 200（被墙则 000，跳过） |
| 4 | curl → CSDN RSS | XML > 10KB，含 `<item>` |
| 5 | curl → B站 | HTML > 200KB，含 `bili-video-card` |
| 6 | iab → 新标签+Bing搜索 | 返回5条结果数组 |
| 7 | Chrome → 新标签+Bing搜索 | 同上 |
| 8 | bilibili_extract.py | 返回 BV 号列表 |

