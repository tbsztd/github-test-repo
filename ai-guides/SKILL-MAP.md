# 技能地图 — Skill Map

> 怎么用：找到你想做的事 → 说出触发词 → Skill 自动激活
> 搜索降级策略见 `AGENTS.md` § 搜索降级决策链

---

## 你的 Skill 全景

| 我要... | 说这些话 | 触发 Skill |
|---------|----------|------------|
| 🖨️ 3D 打印/建模/切片 | "帮我设计个3D打印件" "做个支架" "STL切片" "Bambu" | `3d-print-ai` |
| 📺 提取B站字幕 | "提取这个B站视频字幕" "Get?? 转录" "B站视频内容分析" | `bilibili-transcript` |
| 🎯 定义目标/任务规划 | "帮我定个目标" "创建goal" "明确成功标准" | `define-goal` |
| 🔀 处理 GitHub PR 评论 | "处理PR评论" "address review comments" | `gh-address-comments` |
| 🔧 修复 GitHub CI 失败 | "CI挂了帮我看看" "GitHub Actions报错" | `gh-fix-ci` |
| 📝 Git 操作/提交/推送 | "帮我push" "git提交" "clone仓库" "SSH配置" | `git-cheatsheet` |
| 🔌 MSPM0G 电赛开发 | "MSPM0G驱动" "PID控制" "舵机控制" "I2C通信" "ADC采样" "电赛方案" "天猛星" | `mspm0g-contest` |
| ⚙️ MSPM0 通用开发 | "MSPM0" "CCS开发" "driverlib" "TI MCU" | `mspm0-dev` |
| 🔍 全网搜索/找资料 | **"搜索一下" "找资料" "CSDN" "B站搜" "研究" "学习"** | `multi-source-research` |
| 🌐 浏览器搜索/自动化 | "打开网页" "填表单" "浏览器搜索" "playwright" | `playwright` / Browser / Chrome |
| 📄 Office 文档处理 | "创建docx" "修改Excel" "PPT制作" "officecli" | `officecli` |
| 📕 PDF 读写/生成 | "处理PDF" "生成PDF报告" "提取PDF内容" | `pdf` |
| 📸 桌面截图 | "截个图" "screenshot" "截取窗口" | `screenshot` |
| 🔒 安全审查代码 | "安全审查" "security review" "代码安全检查" | `security-best-practices` |
| 🔊 文字转语音 | "朗读这段" "文字转语音" "TTS" "语音合成" | `speech` |

---

## 搜索能力速查（多环境实测，2026-06-15）

### 通用网页搜索（Bing）

| 方法 | 稳定性 | 已知 Failure |
|------|--------|-------------|
| `curl.exe` → Bing | ★★★★★ 首选 | 极少失败 |
| `Invoke-RestMethod` → Bing | ★★★☆☆ | 部分环境 TLS/SNI 超时 |
| `Invoke-WebRequest` → Bing | ★★★☆☆ | 同上 |
| iab 浏览器 → Bing | ★★☆☆☆ | goto 超时(可兜底) / error -3(不可用) |

### GitHub

| 目标 | 方法 | 稳定性 | 已知 Failure |
|------|------|--------|-------------|
| 搜仓库/Issue | `curl.exe` → api.github.com | ★★★★★ | 几乎不失败 |
| 读文件 | `curl.exe` → raw.githubusercontent.com | ★★★☆☆ | 部分网络 HTTP_CODE:000（被墙） |
| 读文件（raw被墙时） | GitHub API Contents 端点 | ★★★★★ | base64 需解码 |
| 搜仓库/Issue | `Invoke-RestMethod` → api.github.com | ★★★☆☆ | 部分环境 TLS 超时 |

### 其他平台

| 平台 | 方法 | 稳定性 |
|------|------|--------|
| CSDN | `curl.exe` → RSS | ★★★★★ |
| B站 | `curl.exe` → 搜索页 | ★★★★★ |
| B站（结构化） | `bilibili_extract.py` | ★★★★★ |
| 知乎/掘金/博客园 | `curl.exe` → Bing `site:` | ★★★★★ |

### 浏览器方法陷阱速查

| 陷阱 | 症状 | 对策 |
|------|------|------|
| `goto()` 超时 | CDP Page.navigate timeout | try/catch + waitForTimeout(2000)，导航实际成功了 |
| error -3 | WebView2 Runtime 不可用 | 放弃 iab，降级 curl.exe 或 Chrome 扩展 |
| const 重复声明 | "already been declared" | `mcp__node_repl__js_reset`，全部用 `var` |

### 不可用

| 方法 | 原因 |
|------|------|
| `web_search` 工具 | DeepSeek 代理不支持 |
| `curl.exe` → 百度 | JS 渲染空壳 |
| `curl.exe` → Google | CAPTCHA |

---

## 使用原则

### ✅ 正确姿势
```
"帮我搜索一下 CSDN 和 GitHub 上关于 DeepSeek 的资料"
→ 命中 multi-source-research → 自动执行搜索→保存→汇总流程
```

### ❌ 错误姿势
```
"帮我看看这个"  （太模糊，不知道触发哪个）
"做个东西"      （同样模糊）
```

### 🔑 关键技巧
1. **一句话里包含触发词** — 比如 "搜索" 对应 `multi-source-research`
2. **触发词可以组合** — "用 B站 和 GitHub 搜索 xxx" 会触发多渠道搜索
3. **Skill 之间不冲突** — 说 "搜索然后生成PDF" 会先触发研究再触发 PDF
4. **搜索优先用 curl.exe** — 最稳定，比浏览器快 3-5 倍
5. **每个方法都有 fallback** — 见 AGENTS.md 降级决策链，不要一条路走到黑

---

## Skill 触发原理

```
你说 "搜索一下 Codex DeepSeek"
  → 系统扫描所有 skill 的 description 字段
  → "搜索" 命中 multi-source-research 的 description
  → 加载 SKILL.md 正文
  → 执行搜索→保存→汇总流程
```

**description 就是开关。** 如果 description 里没有你用的词，skill 就永远不会触发。

---

## 自检：你的 Skill 都健康吗？

| Skill | description | 状态 |
|-------|-------------|------|
| 3d-print-ai | ✅ 有（但中文可能乱码） | ⚠️ |
| bilibili-transcript | ✅ | ✅ |
| define-goal | ✅ | ✅ |
| gh-address-comments | ✅ | ✅ |
| gh-fix-ci | ✅ | ✅ |
| git-cheatsheet | ✅ | ✅ |
| mspm0g-contest | ✅ 已修复 | ✅ |
| mspm0-dev | ✅ 已修复 | ✅ |
| multi-source-research | ✅ 含平台状态表+降级方案 | ✅ |
| officecli | ✅ | ✅ |
| pdf | ✅ | ✅ |
| playwright | ✅ | ✅ |
| screenshot | ✅ | ✅ |
| security-best-practices | ✅ | ✅ |
| speech | ✅ | ✅ |

---

> 更新：2026-06-15 · 多环境实测，加入已知 Failure + 降级链
