# Codex 学习方法与资源汇总

> 搜索时间：2026-06-13 | 搜索范围：Bing、GitHub、CSDN、博客园、B站

---

## 一、你的当前配置状态

你的 Codex 已经通过 Codex++ 接入了 DeepSeek，配置如下：

| 配置项 | 当前值 |
|--------|--------|
| 模型 | `deepseek-v4-pro` |
| 推理力度 | `xhigh` |
| Provider | `custom` (Codex++ 本地代理) |
| 代理地址 | `http://127.0.0.1:57321/v1` |
| Wire API | `responses` (Codex++ 做协议转换) |
| 上下文窗口 | `1000000` (1M) |
| Codex++ 版本 | v1.2.4 |
| 用户脚本 | Codex Context Used Meter |

---

## 二、DeepSeek + Codex++ 已知问题验证

### 问题 1：上下文窗口被限制在 258K（⚠️ 确认存在）
- **GitHub Issue #931**（今天刚提交）：配置了 1M 上下文，运行时实际只有 258,400 tokens
- 根因：`models_cache.json` 中 `codex-auto-review` 的默认值 272K × 0.95 = 258K 覆盖了自定义配置
- **影响**：20 万上下文实际上是"假 20 万"，实际可用 ~258K  
- 状态：**Open，尚未修复**

### 问题 2：AGENTS.md 中文乱码（已确认）
- **GitHub Issue #842**：DeepSeek API 模式下，个性化面板中文显示为 `???`
- 根因：app-server 的 personality 注入用 GBK 解码 UTF-8 → 乱码
- 状态：**Open**

### 问题 3：DeepSeek 不支持 Responses API
- DeepSeek 只支持 **Chat Completions** 协议
- Codex++ 通过本地代理做 Responses → Chat Completions 转换
- 这导致部分功能受限（如插件、某些高级特性）
- AGENTS.md 中提到 "DeepSeek接入不能用插件" 就是这个原因

### 问题 4：v1.2.5 对话记录读取问题
- **Issue #920**：更新到 v1.2.5 后对话记录无法正常读取
- 建议：**暂时不要升级到 v1.2.5**

---

## 三、优质学习资源（已验证）

### GitHub（最权威）
| 资源 | 链接 | 说明 |
|------|------|------|
| CodexPlusPlus 仓库 | https://github.com/BigPizzaV3/CodexPlusPlus | 18,024 ⭐，Rust+Tauri 开发 |
| GitHub Issues | https://github.com/BigPizzaV3/CodexPlusPlus/issues | 查 bug、提问题 |

### CSDN 文章
| 标题 | 作者 | 阅读量 | 亮点 |
|------|------|--------|------|
| Codex++ 的教程｜增强版下载与使用指南 | 代码简单说 | 4.4w | 插件解锁、会话删除、API中转、Markdown导出、用户脚本 |
| Codex++：给 Codex App 加上删除、导出、插件解锁和 Provider 同步 | 秒懂AI+ | 3.6k | Provider 同步、Timeline、配置详解 |

### 博客园
| 标题 | 作者 | 亮点 |
|------|------|------|
| 【2026最新】Codex++下载、安装和使用全流程图解（附Codex接入DeepSeek） | sdfsafafa | 最详细的图文教程，含截图每一步 |

### B站视频（搜索到的高相关度）
| 标题关键词 |
|-----------|
| 【免费开源】一键接入 DeepSeek！Codex + CC Switch 完整配置教程 |
| 【亲测有效】让 DeepSeek 接入 Codex，还能识图 |
| EchoBird + Codex + DeepSeek：让AI编程触手可及 |
| 保姆级教程 Agnes 接入 Codex |
| 终于！Codex-Windows桌面端上线！5分钟轻松上手！ |
| 5月23日最新使用 Codex 5.5 绘制 CAD 图形 |

---

## 四、哪些适合你 / 哪些不适合

### ✅ 适合现在使用
1. **GitHub README** — 最权威的 Codex++ 配置文档，DeepSeek 配置示例明确
2. **博客园图解教程** — 中文、带截图、每步都有，适合跟着操作
3. **GitHub Issues** — 遇到问题时搜索解决方案（94 个 DeepSeek 相关 issue）
4. **Codex Context Used Meter** — 你已经装了，监控上下文消耗很实用
5. **AGENTS.md 中的 Skill 清单** — 按需安装，不要全装

### ⚠️ 部分适用（有坑）
1. **CSDN 文章** — 信息量大但 JS 渲染，curl 抓不到正文（需要浏览器查看）
2. **B站视频** — 时效性差，版本更新快容易过时
3. **知乎专栏** — 部分内容可能不适用于最新版本

### ❌ 不适合 / 踩坑点
1. **DeepSeek + 插件** — 协议限制，无法使用完整插件功能
2. **v1.2.5 升级** — 有对话读取 bug，建议等修复
3. **配置了 1M 上下文但实际只有 258K** — 等 Issue #931 修复

---

## 五、学习路径建议

```
新手入门：
1. 读 GitHub README（了解 Codex++ 是什么）
2. 跟博客园图文教程走一遍（了解安装配置流程）
3. 看 B站视频 "一键接入 DeepSeek"（直观感受）

进阶使用：
4. 研究 AGENTS.md 中的 Skill 清单（按需装 skill）
5. 学习 Superpower 工作流（TDD 红绿重构）
6. 配置上下文监控（已装 Context Used Meter）

排障：
7. 遇到问题先搜 GitHub Issues
8. 社区争议点看 AGENTS.md 最后一节
```

---

## 六、关键数字速记

| 项目 | 数值 |
|------|------|
| CodexPlusPlus Stars | 18,024 |
| DeepSeek 相关 Issues | 94 个 |
| 实际可用上下文 | ~258K（非配置的 1M） |
| Codex++ 稳定版本 | v1.2.4 |
| Base URL (DeepSeek) | `https://api.deepseek.com` |
| 上游协议 | Chat Completions |
