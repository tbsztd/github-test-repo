# WORKSPACE.md — AI 入职说明书

> 给每个进入此工作区的 Codex 会话：这是你的操作手册。

---

## 一、你是谁，你在哪

- **身份**：你是 Codex 桌面端的 AI Agent，通过 Codex++ 接入 DeepSeek V4 Pro
- **工作区**：`C:\Users\tbsztdgczp\Desktop\codex_work`
- **用户**：电子设计竞赛（电赛）参赛者，同时是 AI 工具重度用户
- **系统**：Windows，PowerShell 为默认 Shell
- **上下文预算**：约 20 万 tokens（实际 ~258K，CodexPlusPlus#931 已知 bug）

---

## 二、你的武器库：16 个 Skill

> 触发规则：用户说出的词必须命中 skill 的 `description` 字段。词不对 = skill 不触发。

| 触发词 | Skill | 用途 |
|--------|-------|------|
| "搜索" "找资料" "CSDN" "B站" "研究" | `multi-source-research` | 四路并行搜索 → 保存 → 汇总文档 |
| "3D打印" "支架" "STL" "Bambu" "拓竹" | `3d-print-ai` | OpenSCAD/Blender 建模 → 切片 → JLC 下单 |
| "MSPM0G" "PID" "舵机" "I2C" "ADC" "天猛星" | `mspm0g-contest` | MSPM0G3507 + K230 双芯架构开发 |
| "MSPM0" "CCS" "driverlib" "TI MCU" | `mspm0-dev` | MSPM0 通用 MCU 开发 |
| "B站字幕" "提取字幕" "视频转录" | `bilibili-transcript` | B站视频字幕提取（API + 浏览器双路径） |
| "push" "commit" "clone" "git提交" | `git-cheatsheet` | Git 操作 + 用户身份配置 |
| "CI挂了" "GitHub Actions" | `gh-fix-ci` | GitHub CI 故障排查 |
| "PR评论" "review" | `gh-address-comments` | 处理 GitHub PR 评审意见 |
| "定目标" "goal" | `define-goal` | 将模糊意图转为可量化目标 |
| "docx" "Excel" "PPT" | `officecli` | Office 文档创建/修改 |
| "PDF" "生成报告" | `pdf` | PDF 生成/提取/渲染验证 |
| "打开网页" "填表单" "playwright" | `playwright` | 终端浏览器自动化 |
| "截个图" "screenshot" | `screenshot` | 桌面截图 |
| "朗读" "TTS" "语音合成" | `speech` | 文字转语音 |
| "安全审查" "security review" | `security-best-practices` | 代码安全审查（仅 Python/JS/Go） |

> 完整 Skill 说明见：`SKILL-MAP.md`

---

## 三、你的工具库

### 可直接调用的工具

| 工具 | 能力 | 限制 |
|------|------|------|
| `shell_command` | 运行 PowerShell 命令 | Windows，不是 bash |
| `apply_patch` | 修改文件（精确 diff） | 不能用于新建大文件 |
| `mcp__node_repl__js` | 运行 JavaScript | 可控制浏览器、调 Playwright |
| `update_plan` | 创建/更新任务计划 | 向用户展示进度 |
| `view_image` | 查看本地图片 | - |

### 可用的插件

| 插件 | 能力 |
|------|------|
| Browser | 应用内浏览器（可开 localhost） |
| Chrome | 控制用户 Chrome（Chrome 未安装，此插件不可用） |
| Spreadsheets | Excel 创建/编辑 |
| Presentations | PPT 创建/编辑 |
| Documents | Word 创建/编辑 |
| Figma | 设计转代码 |
| Canva | 设计创作 |
| Netlify | 部署 |
| Game Studio | 游戏开发 |
| Superpowers | TDD/规划/调试工作流 |

---

## 四、工作区文件地图

```
codex_work/
├── AGENTS.md              ← 每次会话自动加载的指令（最高优先级）
├── SKILL-MAP.md           ← 任务→Skill 完整对照表
├── codex-research-summary.md  ← Codex 学习资源汇总（Bug 验证+资源）
├── codex-learning-methodology.md ← 多渠道搜索方法论（可复用模式）
├── *.py                   ← 电赛相关 Python 脚本
├── *.tex / *.pdf          ← LaTeX 实验报告
├── circuit_*              ← 电路设计文件
└── 【电赛*】.txt           ← B站视频字幕/笔记
```

### 各文档用途

| 文档 | 给 AI 的指示 |
|------|-------------|
| `AGENTS.md` | **每次会话必读**。包含触发词速查、Skill 清单、工作流、排障 |
| `SKILL-MAP.md` | 详细的任务→触发词→Skill 映射，用户说"我不知道该用哪个 skill"时看这个 |
| `codex-research-summary.md` | Codex 使用经验汇总：DeepSeek 兼容性、已知 Bug、资源列表 |
| `codex-learning-methodology.md` | 搜索方法论：如何攻克 JS 渲染、GitHub API 用法、RSS 抓取策略 |

---

## 五、解决问题的方法论

### 接到任务后的标准流程

```
1. 理解任务类型
   → 查 AGENTS.md 触发词表
   → 确定激活哪个 Skill

2. 制定计划
   → 用 update_plan 列出步骤
   → 向用户确认方向

3. 获取信息
   → 如果是"搜索/研究"类 → multi-source-research 自动激活
   → 如果是"开发"类 → 先读相关 Skill 正文
   → 如果是"未知"类 → 用 Bing/curl 搜索，边做边学

4. 执行和验证
   → 修改文件用 apply_patch
   → 运行测试用 shell_command
   → 每次改完验证一次

5. 沉淀输出
   → 结果存成 .md 文档
   → 方法论写入 SKILL-MAP 或更新 AGENTS
```

### 遇到未知问题时的搜索策略

```
步骤1：Bing 搜索定位
  curl "https://www.bing.com/search?q=site:csdn.net+{topic}&setlang=zh-cn"

步骤2：GitHub 搜索深挖
  curl -H "Accept: application/vnd.github+json" \
    "https://api.github.com/search/repositories?q={topic}&sort=stars"

步骤3：CSDN RSS 获取正文
  curl "https://blog.csdn.net/{author}/rss/list"

步骤4：B站搜索视频教程
  curl "https://search.bilibili.com/all?keyword={topic}"

步骤5：汇总成文档，保存到工作区
```

---

## 六、关键约束与避坑

### 技术限制
- **DeepSeek 协议**：只支持 Chat Completions，Codex++ 做 Responses→Chat 转换。部分插件功能不可用
- **上下文实际 ~258K**：不是配置的 1M（等 CodexPlusPlus#931 修复）
- **中文编码**：PowerShell 读文件用 `[System.Text.Encoding]::UTF8` 避免乱码
- **Chrome 未安装**：不要尝试用 Chrome 插件，用 Browser（应用内浏览器）代替

### 行为准则
- 修改文件前先读当前内容（避免覆盖用户的手动修改）
- 不确定时先问再动手（不做不可逆操作）
- 每次完成一个阶段用 `update_plan` 更新进度
- 搜索结果存到 `$env:TEMP`，不要在工作区留临时文件

### PowerShell 特有注意事项
- 变量用 `$env:VAR` 而不是 `${VAR}`
- 中文文件名在 PowerShell 中容易乱码，优先用英文
- `curl.exe` 是真正的 curl，`curl` 是 PowerShell 别名（指向 Invoke-WebRequest）

---

## 七、快速自检清单

每次开始对话时，在心里过一遍：

- [ ] 读了 AGENTS.md 吗？
- [ ] 用户的任务命中哪个 Skill 的 description？
- [ ] 需要先 Plan 吗？
- [ ] 搜索策略：Bing → GitHub API → CSDN RSS → B站？
- [ ] 结果要存成文档吗？
- [ ] 做完要更新 SKILL-MAP 吗？

---

> **记住**：你不是从零开始的通用 AI。你有 16 个 Skill、4 个插件、完整的搜索方法论。用户的每一句话都是触发开关——对准 description 里的词，skill 就会自动激活。
