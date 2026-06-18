# Codex 工作区配置 - 社区智慧蒸馏版
# 来源：B站 40+ 创作者 2025.10~2026.06 社群共识
# 原则：精简高信号，不浪费上下文

## 快速触发词（Skill Map）

| 想做什么 | 说这个 |
|----------|--------|
| 全网搜索/找资料 | "搜索" "找资料" "CSDN" "B站搜" "研究一下" |
| 3D打印/建模 | "3D打印" "支架" "STL" "Bambu" "拓竹" |
| B站字幕提取 | "B站字幕" "提取字幕" "视频转录" |
| 电赛MSPM0G开发 | "MSPM0G" "PID" "舵机" "I2C" "ADC" "天猛星" |
| MSPM0通用开发 | "MSPM0" "CCS" "driverlib" "TI MCU" |
| Git操作 | "push" "commit" "clone" "git提交" |
| GitHub CI修复 | "CI挂了" "GitHub Actions报错" |
| PR评论处理 | "处理PR评论" "address review" |
| 目标规划 | "定目标" "创建goal" |
| Office文档 | "docx" "Excel" "PPT" "officecli" |
| PDF处理 | "PDF" "生成报告" |
| 浏览器搜索/自动化 | "打开网页" "填表单" "playwright" "浏览器搜索" |
| 文字转语音 | "朗读" "TTS" "语音合成" |
| 安全审查 | "安全审查" "security review" |
| 桌面截图 | "截个图" "screenshot" |

## 网络搜索 — 带 fallback 的实战手册（2026-06-15 多环境实测）

> 核心原则：**没有万能方法。每种方法在不同网络/机器上表现不同。按下面顺序尝试，失败则降级。**

### 一、Bing 通用网页搜索

| 优先级 | 方法 | 执行工具 | 表现 |
|--------|------|----------|------|
| 1 | `curl.exe` | `shell_command` | 最稳定，几乎不失败 |
| 2 | `Invoke-RestMethod` | `shell_command` | 自动解析，但部分环境 TLS 超时 |
| 3 | `Invoke-WebRequest` | `shell_command` | 同上，返回完整 HTTP 对象 |
| 4 | 内置浏览器 iab | `mcp__node_repl__js` | goto 会超时但导航实际成功 |

```powershell
# 【首选】curl.exe — 最稳定
curl.exe -s -L --max-time 15 -H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36" "https://www.bing.com/search?q={搜索词}&setlang=zh-cn&count=10" -o "$env:TEMP\bing.html"

# 【备选】Invoke-RestMethod — 若 curl 因故不可用
try { $r = Invoke-RestMethod -Uri "https://www.bing.com/search?q={搜索词}" -TimeoutSec 10; $r } catch { Write-Host "Invoke-RestMethod failed, use curl.exe instead" }
```

### 二、GitHub 搜索

| 优先级 | 目标 | 方法 | 表现 |
|--------|------|------|------|
| 1 | 搜仓库/Issue | `curl.exe` → `api.github.com/search` | 稳定，必须带 User-Agent |
| 2 | 搜仓库/Issue | `Invoke-RestMethod` → 同上 | 自动 JSON 解析，部分环境 TLS 超时 |
| 3 | 读文件内容 | `curl.exe` → `raw.githubusercontent.com` | 部分网络被墙(HTTP_CODE:000) |
| 4 | 读文件内容（raw被墙时） | GitHub API Contents 端点 | base64 编码，需解码 |

```powershell
# 【仓库搜索】curl.exe — 必须带 User-Agent 头
curl.exe -s -L --max-time 15 -H "Accept: application/vnd.github+json" -H "User-Agent: Codex-CLI" "https://api.github.com/search/repositories?q={搜索词}&sort=stars&per_page=5"

# 【读文件 — 首选 raw】部分网络可能被墙
curl.exe -s -L --max-time 10 -H "User-Agent: Codex-CLI" "https://raw.githubusercontent.com/{owner}/{repo}/main/{path}"

# 【读文件 — raw被墙时的降级方案】用 GitHub API Contents
# 返回 JSON，content 字段是 base64，需解码
curl.exe -s --max-time 10 -H "Accept: application/vnd.github+json" -H "User-Agent: Codex-CLI" "https://api.github.com/repos/{owner}/{repo}/contents/{path}"
# PowerShell 解码: $json | ConvertFrom-Json | % { [Text.Encoding]::UTF8.GetString([Convert]::FromBase64String($_.content)) }

# 【备选】Invoke-RestMethod — 若 curl 不可用
Invoke-RestMethod -Uri "https://api.github.com/search/repositories?q={搜索词}&per_page=5" -Headers @{"Accept"="application/vnd.github+json"; "User-Agent"="Codex-CLI"} -TimeoutSec 10
```

### 三、CSDN 搜索

| 优先级 | 方法 | 表现 |
|--------|------|------|
| 1 | `curl.exe` → RSS | 稳定，20篇/页 |
| 2 | Bing `site:blog.csdn.net` | 通用，不拘泥于已知作者 |

```powershell
# 已知作者：用 RSS
curl.exe -s -L --max-time 10 -H "User-Agent: Mozilla/5.0" "https://blog.csdn.net/{用户名}/rss/list" -o "$env:TEMP\csdn.xml"
# 未知作者：用 Bing
curl.exe -s -L --max-time 15 -H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)" "https://www.bing.com/search?q=site:blog.csdn.net+{搜索词}&setlang=zh-cn"
```

### 四、B站搜索

| 优先级 | 方法 | 表现 |
|--------|------|------|
| 1 | `curl.exe` → 搜索页 | 稳定，返回 HTML |
| 2 | `bilibili_extract.py` | 结构化输出（BV号/播放量/UP主） |

```powershell
# 搜索页 HTML
curl.exe -s -L --max-time 15 -H "User-Agent: Chrome/131.0.0.0 Safari/537.36" "https://search.bilibili.com/all?keyword={搜索词}&order=totalrank" -o "$env:TEMP\bili.html"

# 结构化搜索（Python 脚本）
cd C:\Users\tbsztdgczp\Desktop\codex_work
& "D:\Python\python.exe" -X utf8 bilibili_extract.py "{搜索词}"
```

### 五、内置浏览器 iab — 已知问题与对策

执行工具：`mcp__node_repl__js`

**已知 Failure Mode**：
- ❌ `goto()` 总是超时（CDP Page.navigate timeout），但导航实际已发生
- ❌ 部分环境报 `error -3`（WebView2 Runtime 问题），浏览器完全不可用  
- ❌ 若之前执行过 `const { setupBrowserRuntime }`，再次执行会报 "already been declared"

```js
// ====== Bootstrap（每次新 Node REPL 会话执行一次）======
// 若报 "already been declared" → 先执行 mcp__node_repl__js_reset
var { setupBrowserRuntime } = await import("C:/Users/tbsztdgczp/.codex/plugins/cache/openai-bundled/browser/26.602.40724/scripts/browser-client.mjs");
await setupBrowserRuntime({ globals: globalThis });
globalThis.browser = await agent.browsers.get("iab");

// ====== 搜索 ======
var tab = await browser.tabs.new();
// goto 几乎必定超时但导航实际成功，必须 try/catch
try { await tab.goto("https://www.bing.com/search?q={搜索词}&setlang=en&count=5"); } catch(e) {}
// 等 2 秒让它加载完
await tab.playwright.waitForTimeout(2000);
// 验证是否真的到了目标页
var url = await tab.url();
// 读结果
var results = await tab.playwright.evaluate(() => {
    var items = document.querySelectorAll(".b_algo h2");
    return Array.from(items).slice(0, 5).map(h => h.textContent.trim());
});
```

> **若 iab 报 `error -3` 或 `WebView2` 相关错误**：iab 在此环境不可用。
> 降级到 `curl.exe`（方法一）或 Chrome 扩展（方法六）。

### 六、Chrome 扩展（复用用户登录态）

执行工具：`mcp__node_repl__js`  
前置条件：Edge/Chrome 中已安装 Codex 扩展

```js
// Bootstrap（如果之前 iab 已执行过 setupBrowserRuntime，跳过 import 行）
var { setupBrowserRuntime } = await import("C:/Users/tbsztdgczp/.codex/plugins/cache/openai-bundled/chrome/26.602.40724/scripts/browser-client.mjs");
await setupBrowserRuntime({ globals: globalThis });

var ext = await agent.browsers.get("extension");
// 认领用户已开标签（省去重新登录）
var userTabs = await ext.user.openTabs();
var claimed = await ext.user.claimTab(userTabs[0]);
// 或创建新标签
var newTab = await ext.tabs.new();
try { await newTab.goto("https://www.bing.com/search?q={搜索词}"); } catch(e) {}
await newTab.playwright.waitForTimeout(2000);
```

### 七、Playwright CLI

执行工具：`shell_command`

```powershell
npx @playwright/cli@latest open --browser=msedge --headed
npx @playwright/cli@latest goto "https://www.bing.com/search?q={搜索词}"
npx @playwright/cli@latest eval "()=>document.body.innerText"
npx @playwright/cli@latest close
```

### 不可用方法

| 方法 | 为什么 |
|------|--------|
| `web_search` 工具 | DeepSeek 代理不支持。调用返回 "unsupported custom tool call" |
| `curl.exe` → 百度 | JS 渲染空壳，227 字节 |
| `curl.exe` → Google | CAPTCHA 拦截 |
| `Invoke-WebRequest` → raw.githubusercontent.com | 本机也超时（20s），curl 反而不超时 |
| 浏览器 iab → WebView2 环境 | 报 error -3 的机器完全不可用 |

---

## 搜索降级决策链

```
收到搜索请求
│
├─ 1. 先试 curl.exe → Bing / GitHub API（2s，几乎必定成功）
│   └─ 失败？→ 试 Invoke-RestMethod
│
├─ 2. 需要抓 GitHub 文件内容？
│   ├─ 先试 curl.exe → raw.githubusercontent.com
│   └─ HTTP_CODE:000 或超时？→ 降级到 GitHub API Contents（base64解码）
│
├─ 3. 需要 JS 渲染页面？
│   ├─ 试内置浏览器 iab
│   │   ├─ goto 超时？→ 正常，try/catch + waitForTimeout 兜底
│   │   └─ error -3？→ iab 不可用，跳过
│   ├─ iab 不可用 → 试 Chrome 扩展
│   └─ 都不行 → 用 Playwright CLI
│
└─ 4. 以上全失败？
    └─ 告知用户：当前网络环境限制，建议手动搜索并提供搜索词
```

## 已装插件
Chrome Browser Spreadsheets Presentations Documents
建议补装: Computer Use（桌面操控）、Figma（设计转代码）

## 核心 Skill 清单（按需触发，不要全装）
### 工程类
- Stack: 全栈标准流水线（需求→API→前后端→DB→部署）
- Super Power: TDD红绿重构循环，双轮审计防代码幻觉
- Codebase: 理解代码库结构
- Exploit: 仿Claude子Agent，先定代码位置再交给主Agent，省上下文

### 元能力
- Skill Creator: 大白话描述→自动生成标准化Skill
- Find Skills: 自动搜索skill.sh平台找最佳Skill并安装

### 设计类（去AI味）
- Taste-Skill: 给AI补审美课（GitHub 3万星）
- Front-end Design: 画面出彩
- UI UX Pro Max: 160+行业设计规则，产品专业化

### 内容类（宝玉工具包）
- cover-image / infographic / markdown-to-HTML / translate / 一键发布

## 关键工作流
### goal命令（无人值守长任务）
用法: 先Plan模式出计划→退出Plan→goal执行
配套: OpenSpec生成规格文档 + goal执行规格 = Spec-Driven开发
陷阱: 模糊提示词会跑偏烧token；破坏性操作/Plan模式下不能用

### 多Agent协作
- Obsidian项目库做中枢：只放确定性文档，按岗位开并行对话(设计师/PM/程序员/运营)
- CC交叉审查: ClaudeCode审查Codex代码(发现盲区bug)→回ClaudeCode修改
- 监督者+工人: ClaudeCode监督分配→Codex sub-agent执行→验证

### 上下文管理（致命短板: 20万）
- 只装真需要的Plugin/Skill，定期清理不用的
- /side 斜杠命令开独立对话不污染主上下文
- 复杂任务先出Plan减少反复

### Hooks生命周期
SessionStart → UserPromptSubmit → PreToolUse → PermissionRequest → PostToolUse → Stop
用途: 高风险命令拦截、代码审查前置、任务完成通知、Stop时自动总结上下文防超限

## Skill 使用铁律

1. **触发靠 description** — 你说的词必须在 skill 的 description 里出现过
2. **一句话说清意图** — "搜索 CSDN 和 GitHub 上 DeepSeek 的配置教程" 比 "帮我看看" 有效 100 倍
3. **触发词可以组合** — 一个请求可以触发多个 skill
4. **参见 SKILL-MAP.md** — 完整任务→skill 对照表

## 排障速查
- DeepSeek接入不能用插件、web_search工具和图片/多模态
- Codex++须关代理，否则报错
- 恢复默认账号: 删config.toml中代理配置段，直接启动Codex APP（不从Codex++重启）
- 20万上下文耗尽: 开新对话/side分拆/Fork分叉/subagent并行
- 浏览器 bootstrap 报 "already been declared"：先 `mcp__node_repl__js_reset`，再用 `var` 声明
- 浏览器 goto 超时：导航实际成功了，用 try/catch + waitForTimeout(2000) 兜底
- 浏览器报 error -3 / WebView2：该环境的 iab 不可用，降级到 curl.exe 或 Chrome 扩展
- curl.exe 访问 raw.githubusercontent.com 返回 000：被墙，改用 GitHub API Contents 端点
- Invoke-WebRequest 超时：改用 curl.exe（底层 TLS 栈不同，curl 更稳定）

## 社区争议
- Skill数量: 越精准越好 vs 越多越乱（oil欧呦: 数十个Skill每次带几千无关token）
- ClaudeCode vs Codex: 后端逻辑CC强 vs 前端UICodex强（因人而异，建议双用）
- Multi-Agent价值: 模型越强差异越小（光头不砍树）
