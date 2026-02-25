---
name: Intelligence Ingestion
version: 2.0.0
description: >
  Analyze and evaluate URLs, links, articles, tweets, and external info sources for strategic
  value. NOT a summarizer — this skill classifies, scores importance, maps to architecture,
  stores structured notes in Obsidian, updates the Strategic Landscape, and auto-generates
  draft Skills when new capabilities are detected (Auto-Skill Synthesis).
  Use when: user shares a link (x.com, github.com, arxiv.org, any URL), pastes article text,
  says "analyze this", "evaluate this", "what do you think about this", or forwards content.
  Do NOT use for: simple summarization requests (use summarize skill instead).
---

# Intelligence Ingestion Skill

> 你在喂养一个比你聪明的东西。
> 这个 Skill 确保你知道它吃了什么，变成了什么。

## 前置要求

| 工具 | 用途 | 必须？ |
|------|------|--------|
| **Obsidian** | 知识存储与关联（笔记落地） | ✅ 必须 |
| **OpenClaw** | Skill 宿主 + Agent 编排 | ✅ 必须 |
| **Chrome** | 用于读取需要登录态的页面（如 X/Twitter） | ✅ 建议 |

## 配置

安装后编辑 `config.json`：

```json
{
  "obsidian_vault_path": "/path/to/your/Obsidian_Vault",
  "intelligence_folder": "20_Intelligence",
  "landscape_path": "STRATEGIC_LANDSCAPE.md",
  "output_dir": "/path/to/your/output"
}
```

| 字段 | 说明 | 默认值 |
|------|------|--------|
| `obsidian_vault_path` | Obsidian Vault 根目录 | 无（必填） |
| `intelligence_folder` | 情报笔记子文件夹 | `20_Intelligence` |
| `landscape_path` | Strategic Landscape 文件路径（相对于 workspace） | `STRATEGIC_LANDSCAPE.md` |
| `output_dir` | 非知识类产出目录 | 无（可选） |

> **存储策略：Obsidian = 知识输入（分析/笔记），Output = 内容产出（帖子/脚本等）**

### 首次安装

如果 `STRATEGIC_LANDSCAPE.md` 不存在，Skill 会自动创建一个空模板。
如果 Obsidian Vault 路径不存在或 `config.json` 格式错误，Skill 会报错并提示修复方法。

---

## 触发条件

**自动触发：**
- 用户发送 URL（x.com、github.com、arxiv.org 等）
- 用户粘贴文章/推文内容
- 用户说「分析这个」「评估一下」「这个怎么样」
- 用户从 Telegram 或任意渠道转发内容

**不触发：**
- 用户问与外部内容无关的一般性问题
- 用户明确说不需要分析
- 购物链接、音乐链接、视频娱乐链接等非知识类 URL

---

## 内容提取策略（重要）

不同来源需要不同的提取方式。按以下优先级执行：

### 通用网页
```
1. read_url_content(url) → 成功则直接使用
2. 失败 → 尝试浏览器工具读取
```

### X/Twitter（反扒严重）
X 平台有严格的反爬机制。使用以下降级链：

```
1. xurl Skill（推荐）→ 调用 OpenClaw 内置的 xurl Skill，通过 X API v2 认证访问
   - 最可靠的方式，不受反扒影响
   - 需要 xurl 已配置认证
2. 失败 → 浏览器工具读取（利用 Chrome 登录态绕过反扒）
   - 依赖用户 Chrome 已登录 X 账号
3. 失败 → Web 搜索该推文内容（搜索引擎通常有缓存）
4. 全部失败 → 请求用户粘贴推文原文
```

> **原则：永远不空手回来。** 即使内容提取失败，也要告诉用户哪一步失败了、为什么、怎么解决。

### GitHub
```
1. read_url_content(url) → 通常可直接读取
2. 如果是 repo 主页 → 重点读取 README.md
3. 如果是 issue/PR → 提取标题、描述、关键评论
```

### 付费墙 / 需登录
```
1. 浏览器工具读取（用户可能已登录）
2. 失败 → 标注「内容不可达」，基于用户提供的摘要分析
```

---

## Pipeline（8 步）

### Step 1: READ — 提取内容
按上述「内容提取策略」执行。记录使用了哪种提取方式。

### Step 2: CLASSIFY — 分类
分配 **1 个主分类** + 最多 2 个标签：

| 分类 | 描述 | 示例 |
|------|------|------|
| `infra` | 基础设施 / 协议 / 网络 | MCP、Pilot Protocol |
| `strategy` | 架构决策 / 路由 / 成本优化 | 模型路由、多账号策略 |
| `skill` | Agent Skills / 工具 / 能力 | Skill 设计模式、MCP 接口 |
| `business` | 商业模式 / 市场机会 / 变现 | SaaS 框架、定价策略 |
| `theory` | 概念框架 / 心智模型 | 贝叶斯、决策理论 |
| `tutorial` | 教程 / 学习材料 | Claude Code、Agent Loop |
| `product` | 工具 / 应用 / 服务 | LM Studio、新模型发布 |
| `threat` | 风险 / 安全 / 弃用警告 | API 变更、安全漏洞 |

### Step 3: ANALYZE — 战略价值评估

```markdown
## 战略评估
- **是什么？** [一句话]
- **对我们有什么用？** [具体能力/收益]
- **能用它做什么？** [具体产出/项目]
- **战略价值：** [🔴 关键 / 🟡 高 / 🟢 中 / ⚪ 低]
- **竞争优势：** [不知道这个的人会有什么劣势？]
- **能力边界变化：** [摄取后，Agent 能做什么之前做不到的事？]
```

### Step 4: MAP — 关联现有架构

读取 `{landscape_path}`（Strategic Landscape 全景图），对照回答：
- 这条信息影响架构的哪一层？
- 与现有组件有什么依赖/协同？
- 是否需要更新全景图？

如果 Landscape 文件不存在 → 跳过此步，在 Step 7 提示用户初始化。

### Step 5: STORE — 写入 Obsidian

笔记路径：`{obsidian_vault_path}/{intelligence_folder}/YYYYMMDD_来源_标题.md`

模板：
```markdown
# [标题]

**来源：** [Link](URL)
**日期：** YYYY-MM-DD
**分类：** [主分类] / [标签]
**战略价值：** [🔴/🟡/🟢/⚪] + 一句话原因
**提取方式：** [read_url / browser / search / user_paste]

---

## 摘要
[2-3 段核心内容]

## 关键要点
[编号列表]

## 对架构的影响
[与现有系统的关系]

## 能力边界变化
[摄取后 Agent 能做什么之前做不到的事？如果没有变化写「无」]

## 行动项
[下一步做什么，如果没有写「暂无」]

---

**分析笔记：** [直接、犀利的分析]
```

### Step 6: SYNTHESIZE — 自动生成 Skill 草案（核心进化步骤）

**触发条件：** 当摄取的内容描述了一个 **可操作的工具、API、协议或技术**，且 Agent 当前尚未具备该能力时，自动触发。

**不触发：** 纯理论 / 纯观点 / 已具备的能力 / 战略价值 ⚪ 低的内容。

**执行流程：**

1. **能力差距判断：** 对比当前 `skills/` 目录和 Strategic Landscape，确认这是一个 Agent 尚不具备的新能力。
2. **草案生成：** 在 `{workspace}/skills/_drafts/` 目录下创建新 Skill 文件夹：

```
skills/_drafts/[skill-name]/
├── SKILL.md          # 自动生成的 Skill 行为定义
└── README.md         # 简要说明（来源、用途、依赖）
```

3. **SKILL.md 草案模板：**

```markdown
---
name: [Skill 名称]
version: 0.1.0-draft
description: >
  [基于摄取内容自动生成的描述]
  Auto-synthesized by Intelligence Ingestion from: [来源 URL]
---

# [Skill 名称]

> 🧬 此 Skill 由 Intelligence Ingestion 自动生成，需人工审核后安装。
> 来源: [URL]
> 生成日期: YYYY-MM-DD

## 前置要求
[从摄取内容中提取的依赖项：API Key、SDK、服务等]

## 触发条件
[基于能力描述推断的触发场景]

## 执行流程
[基于摄取内容提炼的操作步骤]

## 输入/输出
- **输入：** [预期输入]
- **输出：** [预期产出]
```

4. **标记状态：** 草案 Skill 放在 `_drafts/` 目录，不会被 OpenClaw 自动加载。只有用户审核后移动到 `skills/` 目录才会激活。

> **原则：Agent 不能自己给自己装模组。** 所有自动生成的 Skill 都是草案状态，必须经过用户审核。这是安全边界。

### Step 7: REMEMBER — 更新记忆

1. **必做：** 追加到当日日志 `{workspace}/memory/YYYY-MM-DD.md`
2. **如果 🔴 关键：** 同步更新 `{landscape_path}`（Strategic Landscape）
3. **如果涉及新工具：** 标记待更新 `TOOLS.md`
4. **如果生成了 Skill 草案：** 记录到日志，标注草案路径和审核状态

### Step 8: RESPOND — 回复用户

```
📥 已摄取: [标题]
📂 分类: [分类]
🎯 战略价值: [🔴/🟡/🟢/⚪] [一句话]
🔄 能力边界变化: [有变化则说明 / 无变化]
🧬 Skill 草案: [已生成 → skills/_drafts/[name]/ | 不适用]
💾 已存档: Obsidian → {intelligence_folder}/[文件名]
🗺️ 全景图: [已更新 / 未更新 / 未初始化]
⚡ 建议行动: [审核并安装 Skill / 其他下一步]
```

---

## 搭档：Strategic Landscape（战略全景图）

Intelligence Ingestion 搭配 **Strategic Landscape** 一起使用：

- **Intelligence Ingestion** = 信息的「入口」，负责摄取、过滤和分类
- **Strategic Landscape** = 信息的「地图」，展示全局态势和能力边界

每当摄取到 🔴 关键信息时，Landscape 自动更新。
你随时能看到 Agent 的完整能力地图 —— 能做什么、在开发什么、还有什么空白。

首次使用时如果没有 Landscape 文件，Skill 会创建一个包含以下结构的空模板：
- Architecture Stack（架构层次）
- Resource Inventory（资源清单）
- Intelligence Tracking（情报追踪）
- Competitive Advantages（能力优势）

---

## 完整示例

### 输入
```
分析这个: https://x.com/kaboraAI/status/1234567890
```

### 执行过程
```
Step 1: READ
  → xurl Skill 调用 X API v2 → 成功
  → 内容：「MCP is the new TCP for agents. 97M downloads...」

Step 2: CLASSIFY
  → 主分类: infra
  → 标签: skill, strategy

Step 3: ANALYZE
  → 战略价值: 🔴 关键
  → 能力边界变化: 所有 Skill 需要 MCP 接口才能被其他 Agent 发现

Step 4: MAP
  → 影响层: Layer 2 (Communication) + Layer 4 (Orchestration)
  → 需要更新 Landscape: 是

Step 5: STORE
  → 已写入 Obsidian: 20_Intelligence/20260225_kaboraAI_MCP_Agent_TCP.md

Step 6: SYNTHESIZE
  → 能力差距判断: Agent 当前没有 MCP 兼容接口生成能力
  → 草案生成: skills/_drafts/mcp-schema-generator/SKILL.md
  → 草案内容: 读取现有 Skill，自动生成 MCP JSON Schema

Step 7: REMEMBER
  → 已更新 memory/2026-02-25.md
  → 已更新 STRATEGIC_LANDSCAPE.md
  → 已记录 Skill 草案: mcp-schema-generator (待审核)

Step 8: RESPOND
```

### 输出
```
📥 已摄取: MCP is the new TCP for agents
📂 分类: infra / skill, strategy
🎯 战略价值: 🔴 关键 — MCP 97M 下载量，24个月窗口期
🔄 能力边界变化: 所有 Skill 需要 MCP 兼容接口才能被 Agent 发现和调用
🧬 Skill 草案: 已生成 → skills/_drafts/mcp-schema-generator/
   → 功能: 读取现有 Skill，自动生成 MCP JSON Schema
   → 依赖: 无（纯文件操作）
   → 审核: mv skills/_drafts/mcp-schema-generator skills/mcp-schema-generator
💾 已存档: Obsidian → 20_Intelligence/20260225_kaboraAI_MCP_Agent_TCP.md
🗺️ 全景图: 已更新（Communication 层 + Orchestration 层）
⚡ 建议行动: 审核 mcp-schema-generator 草案，安装后审计现有 Skill
```

---

## 边界情况

| 情况 | 处理方式 |
|------|---------|
| 多个 URL | 分别处理，各创建独立笔记 |
| 重复内容 | 检查已有笔记，合并或引用 |
| 非英文内容 | 原文分析，用户语言写笔记 |
| 内容不可达 | 标注提取失败原因，基于用户提供的内容分析 |
| 用户自带分析 | 整合用户判断，不覆盖 |
| config.json 缺失 | 报错并输出修复指引 |
| Landscape 文件不存在 | Step 4 跳过，Step 8 提示初始化 |
| X/Twitter 全部提取方式失败 | 请求用户粘贴推文原文 |
| 内容是纯观点/理论 | Step 6 SYNTHESIZE 跳过，标注「不适用」 |
| 已具备该能力 | Step 6 SYNTHESIZE 跳过，标注「能力已存在」 |
| `_drafts/` 目录不存在 | 自动创建 `{workspace}/skills/_drafts/` |

---

## 质量检查

完成后验证（可用 `ls` 确认文件是否存在）：

- [ ] Obsidian 笔记已创建，文件名格式：`YYYYMMDD_来源_标题.md`
- [ ] 当日记忆日志已更新
- [ ] 来源 URL 已保留在笔记中
- [ ] 战略价值已评估
- [ ] 能力边界变化已评估
- [ ] Skill 草案按需生成（如适用，确认 `_drafts/` 下文件存在）
- [ ] Strategic Landscape 按需更新
- [ ] 用户收到确认摘要（Step 8 格式）
- [ ] 若内容提取失败，已告知用户原因和解决方案
