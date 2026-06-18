# Codex 学习：多渠道搜索方法论与核心资源汇总

> 探索时间：2026-06-13 · 工具：curl + Powershell + Node REPL · 覆盖平台：GitHub、CSDN、B站、博客园、知乎

---

## 零、你对 Codex++ 与 DeepSeek 的现状

### 当前配置分析

```
模型: deepseek-v4-pro    wire_api: responses
代理: 127.0.0.1:57321/v1 (Codex++ 本地)
上下文: 配置1M，实际~258K (GitHub Issue #931)
Codex++: v1.2.4  (不要升v1.2.5——有对话读取bug)
```

### 工作原理

```
Codex Desktop ──(Responses API)──▶ 127.0.0.1:57321 ──(Chat Completions)──▶ api.deepseek.com
                                    ▲ Codex++ 代理
                                    做协议转换
```

### 已验证的 3 个 Bug

| Bug | Issue | 状态 |
|-----|-------|------|
| 上下文被截断到258K | CodexPlusPlus#931 | Open |
| AGENTS.md 中文乱码 | CodexPlusPlus#842 | Open |
| v1.2.5 对话无法读取 | CodexPlusPlus#920 | Open |

---

## 一、搜索方法论（可复用模式）

### 1.1 CSDN —— 攻克JS渲染

**问题**：CSDN 文章正文由 JS 异步加载，curl 抓不到。

**演进过程**：
```
尝试1：直接 curl → 只有空壳 HTML           ❌
尝试2：Googlebot UA → 仍然 JS 渲染         ❌
尝试3：devpress.csdn.net SSR → 还是 JS      ❌
尝试4：博客园 → 成功抓到静态内容            ✅ (但不是CSDN本体)
尝试5：CSDN RSS feed → 成功！              ✅✅✅
```

**最终方案 — CSDN RSS**：
```powershell
# 任何 CSDN 作者的 RSS
curl "https://blog.csdn.net/{username}/rss/list"
# 返回完整 XML，含 <title> <description> <pubDate> <link>
```

**收获**：遇到 JS 渲染站，优先找 RSS/Atom feed，其次找 SSR 版本（如 devpress），最后才考虑浏览器渲染。

### 1.2 GitHub —— 结构化深度搜索

**方案A：GitHub Search API**
```powershell
curl -H "Accept: application/vnd.github+json" `
  "https://api.github.com/search/repositories?q=Codex++DeepSeek&sort=stars"
```

**方案B：直接读取 RAW 文件**
```powershell
curl "https://raw.githubusercontent.com/{owner}/{repo}/main/README.md"
```

**方案C：Issues 搜索**（找 bug/解决方案）
```powershell
curl -H "Accept: application/vnd.github+json" `
  "https://api.github.com/search/issues?q=repo:{owner}/{repo}+DeepSeek+is:issue"
```

**关键发现**：
- CodexPlusPlus (18k⭐): 增强启动器，CDP注入
- mimo2codex (583⭐): 纯协议代理，npm/Docker/桌面版三形态

### 1.3 B站 —— 搜索视频教程

**方案**：直接搜索 HTML，不需要 API
```powershell
curl "https://search.bilibili.com/all?keyword=Codex+DeepSeek+配置" `
  -H "User-Agent: Chrome UA"
# 解析：bili-video-card__info--tit 的 title 属性
```

**高价值视频**：
| 标题 | 关键词 |
|------|--------|
| Codex安装+DeepSeek API接入！免登录！保姆级教程 | 入门首选 |
| Codex使用DeepSeek极简入门教程 | 快速上手 |
| Codex (APP) 保姆级全攻略 | 全面覆盖 |
| Mac系统安装codex并使用DeepSeek V4 Pro | Mac 用户 |

### 1.4 知乎/掘金 —— 受限但可用

**现状**：直接 curl 被安全验证拦截（CAPTCHA）。
**替代**：用 Bing `site:zhuanlan.zhihu.com` 搜索然后手动访问。

### 1.5 搜索引擎策略

**Bing 优于 Google**（Google CAPTCHA 更严格）：
```powershell
curl "https://www.bing.com/search?q=site:csdn.net+Codex&setlang=zh-cn"
```

**Bing 结果中 URL 是 Base64 编码的**，需要解码 `u=` 参数：
```
https://www.bing.com/ck/a?!...&u=a1aHR0cHM6Ly9ibG9nLmNzZG4ubmV0Lw...
                                      └─ Base64 = https://blog.csdn.net/
```

---

## 二、核心资源全景图

### GitHub (最权威)

| 仓库 | Stars | 定位 | 对你价值 |
|------|-------|------|----------|
| [BigPizzaV3/CodexPlusPlus](https://github.com/BigPizzaV3/CodexPlusPlus) | 18,024 | 增强启动器 | ⭐⭐⭐ 你正在用 |
| [7as0nch/mimo2codex](https://github.com/7as0nch/mimo2codex) | 583 | 协议代理 | ⭐⭐⭐ 备选方案 |

**mimo2codex 对比 Codex++**：
| 维度 | Codex++ | mimo2codex |
|------|---------|------------|
| 安装 | Windows/Mac 安装包 | npm/Docker/桌面版 |
| 协议转换 | CDP注入+本地代理 | 纯HTTP代理 |
| 管理界面 | Tauri 桌面应用 | Web UI (127.0.0.1:8788) |
| 模型切换 | cc-switch 集成 | 内置 Codex Enable |
| 配置备份 | 有 | 有(.preserve 不丢失原始配置) |
| 多Provider | 内置 | 内置+通用Provider机制 |

### CSDN (通过 RSS 可读)

| 作者 | Codex文章数 | 代表文章 |
|------|-------------|----------|
| weixin_41961749 | 6篇 | Codex++教程、插件修复、会话删除 |
| xianyu120 | 26篇 | Provider同步、插件解锁、Timeline |
| qq_46708746 | 7篇 | DeepSeek接入相关 |

### 博客园 (静态HTML，直接可抓)

| 文章 | 亮点 |
|------|------|
| 【2026最新】Codex++下载安装全流程图解 | 每一步都有截图，最详细的中文教程 |

### 知乎 (搜索到但需浏览器)

`zhuanlan.zhihu.com/p/2048066147011110823` — Codex DeepSeek 相关
`zhuanlan.zhihu.com/p/2041479669141206141` — 另一个配置教程

---

## 三、学习路径建议

### Level 1: 快速上手（今天就能做）

1. 确认 Codex++ 版本是 v1.2.4（不要升级）
2. 从 Codex++ 快捷方式启动（不要从原版 Codex 启动）
3. 看 B站视频 "Codex安装+DeepSeek API接入！保姆级教程"
4. 跟博客园图解教程走一遍配置流程

### Level 2: 理解原理

1. 读 CodexPlusPlus GitHub README — 了解 CDP 注入机制
2. 读 mimo2codex 文档 — 了解 Responses↔Chat Completions 转换
3. 看 CodexPlusPlus Issues — 了解当前 bug 和 workaround
4. 通过 CSDN RSS 跟踪 "代码简单说" 和 "秒懂AI+" 的更新

### Level 3: 高级定制

1. 配置 Codex Context Used Meter（已装）监控上下文消耗
2. 研究 AGENTS.md 中的 Skill 按需安装
3. 关注 mimo2codex 作为 Codex++ 的备选（它支持通用 Provider 机制）
4. 参与 GitHub Issues 讨论，报告你遇到的问题

### 持续学习

```powershell
# 每日检查：GitHub 最新 Issues
curl -H "Accept: application/vnd.github+json" `
  "https://api.github.com/search/issues?q=repo:BigPizzaV3/CodexPlusPlus+is:issue&sort=created&per_page=5"

# 每周检查：CSDN RSS 新文章
curl "https://blog.csdn.net/weixin_41961749/rss/list"
curl "https://blog.csdn.net/xianyu120/rss/list"

# 按需搜索：Bing + B站
curl "https://www.bing.com/search?q=site:csdn.net+Codex+DeepSeek&setlang=zh-cn&freshness=week"
curl "https://search.bilibili.com/all?keyword=Codex+DeepSeek&order=pubdate"
```

---

## 四、辟谣与避坑

| 说法 | 真相 |
|------|------|
| "DeepSeek不能用Codex" | 可以用，但需要Codex++/mimo2codex做协议转换 |
| "Codex插件全能用" | DeepSeek下部分插件功能受限(Chat Completions协议限制) |
| "1M上下文" | 实际~258K，等CodexPlusPlus#931修复 |
| "API Key就能登录" | 还需要先有OpenAI账号（或Google/Apple/MS登录） |
| "v1.2.5更好" | v1.2.5有对话读取bug，建议等修复 |

---

## 五、自检清单

- [ ] 是否从 Codex++ 快捷方式启动？（不是原版 Codex）
- [ ] 版本是 v1.2.4？（不是 v1.2.5）
- [ ] 供应商配置：Base URL `https://api.deepseek.com`，协议 `Chat Completions`
- [ ] 已安装 Context Used Meter 监控上下文
- [ ] 了解上下文实际只有 ~258K
- [ ] 知道中文 AGENTS.md 可能乱码（Issue #842）

---

> **这套方法论的核心**：遇到JS渲染站 → 找RSS/SSR → 不行就用Bing做代理搜索 → GitHub API做结构化查询 → Issues排查bug。
> 关键是形成"试错→找到规律→沉淀模式"的循环，而不是每次都从零开始。
