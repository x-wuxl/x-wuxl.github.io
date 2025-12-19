---
title: "Vibe Coding：把'凭感觉用 AI 写代码'升级成一套可交付的方法论"
date: 2025-12-19
last_modified_at: 2025-12-19
categories: [AI, 软件工程]
tags: [vibe-coding, vibe-coding-指南, ai-coding, AI编程, AI辅助开发, intent-driven-development, 意图驱动开发, 工程方法论, prompt-engineering, cursor, copilot, claude, chatgpt, 开发效率]
description: "将 Vibe Coding 从'爽感操作'变成可复用、可验证、可回滚、可维护的工程流程，构建意图驱动开发(IDD)的完整方法论与实践模板。包含 Memory Bank 记忆库、实现计划、验收标准等完整模板。"
excerpt: "Vibe Coding 最吸引人的地方是你不用从零搭框架，甚至不必每段代码都读懂。它最危险的地方也在这里：当你把'工程控制权'让渡出去，复杂度与风险会以更快速度累积。本文将其升级为一套可交付的工程流程。"
keywords: [Vibe Coding, AI编程, 意图驱动开发, Memory Bank, Cursor, Copilot, Claude, ChatGPT, 软件工程, 开发方法论]
toc: true
toc_sticky: true
---

# Vibe Coding：把"凭感觉用 AI 写代码"升级成一套可交付的方法论

Vibe Coding 最吸引人的地方是：你不用从零搭框架、不用逐行敲样板代码，甚至不必每段代码都读懂——只要描述"我想要什么"，AI 就能迅速给出一个**功能性验证通过 (Functional Validation Passed)** 的版本。
它最危险的地方也在这里：**当你把"工程控制权"让渡出去，复杂度、隐性 bug 与安全风险会以更快速度累积。**

这篇文章想做一件事：把 Vibe Coding 从"爽感操作"变成"可复用、可验证、可回滚、可维护"的工程流程，其实质是将其升级为一种高可控的**意图驱动开发 (Intent-Driven Development, IDD)**。

---

## 1. 什么是 Vibe Coding（以及它不是啥）

**Vibe Coding** 可以理解为一种新的开发交互范式：
你主要用自然语言表达意图与约束，AI 负责产出实现，你通过运行、测试、验收来驱动迭代。

它和传统"AI 辅助编程"的分界点在于：

* **AI 辅助编程**：你仍然像工程师那样阅读/理解/审查代码，AI 提升效率。
* **Vibe Coding**：你更像"导演/产品/架构+验收者"，把大量实现细节交给 AI，通过反馈推动前进。

这并不意味着你不需要工程能力，而是工程能力的重心从"具体编码实现"转移到了：

* **规划（Planning）**
* **上下文管理（Context / Memory）**
* **验证（Tests / Acceptance）**
* **版本化与可回滚（Git / Diff / Commit）**
* **生产化与持续稳健 (Productionalization & Stabilization)**

### 维度对比：理解三种开发范式

| 维度 | 传统编程 (Standard) | AI 辅助 (Copilot style) | Vibe Coding + 工程约束 |
| :--- | :--- | :--- | :--- |
| **重心** | 敲代码、细节实现 | 审查代码、采纳建议 | **规划意图、验证交付** |
| **心流状态** | 逻辑构造、语法记忆 | 连续接受/修正辅助 | **高层设计、快速反馈循环** |
| **错误防控** | 依靠经验与静态检查 | 依靠代码审查 (Review) | **依靠规划把关与自动化测试** |
| **交付感** | 砖块堆砌感 | 提速感 | **导演掌控感** |

---

## 2. 为什么 Vibe Coding 会翻车：三类"债"

Vibe Coding 的关键挑战往往不是"实现速度"，而是"交付质量在规模化时不可控"，容易陷入以下三类债务：

### 2.1 上下文债：缺乏对确定性的约束

你没定义边界条件、数据来源、错误策略、权限模型时，AI 会给一个"看起来合理"的默认世界。

### 2.2 复杂度债：架构碎片化 (Architectural Fragmentation)

几轮之后你会得到：

* 多套重复的 helper
* 互相矛盾的状态管理
* 风格不一致的架构拼贴
* 牵一发动全身的"纸牌屋"

### 2.3 验证债：能跑 ≠ 对；能过 demo ≠ 能上生产

并发、边界、集成、安全、性能与可观测性这些"生产级成本"，很难只靠 prompt 兜住。

---

## 3. 核心共识：Planning is everything

社区最一致的最佳实践可以浓缩成一句话：

> **清晰规划比盲目"让 AI 自由发挥"更重要。**

具体怎么做？

1. **先写设计文档**：

* 游戏：GDD
* 应用：PRD
  用 Markdown 写清：目标、用户路径、非目标、约束、体验偏好、风险偏好。

2. **再让 AI 基于文档 + 技术选型生成实现计划（Implementation Plan）**，而不是直接开写。

3. **实现计划必须"小步可验证"**：每一步都要带测试/验收方式。

这一步的意义是：你把 AI 的创造力限定在"实现层"，把系统的可控性留在"规划层"。

---

## 4. 维持上下文一致性：`.ai/` 记忆库是地基

Vibe Coding 最大的问题之一是"上下文漂移"：
你换个对话、过两天再继续、或者中途试过几个方向，模型很容易失忆或混淆。

解决方案就是 **AI 记忆库（Memory Bank）**：把关键上下文固化成文件，让 AI 每次都"从同一份真相开始"。

### 4.1 为什么用 `.ai/` 作为目录名？

我们推荐使用 **`.ai/`** 而不是 `memory-bank/`，理由如下：

| 优点 | 说明 |
|------|------|
| **隐藏目录** | 以 `.` 开头，不污染项目主结构，类似 `.git`、`.vscode`、`.github` |
| **语义明确** | 一眼就知道是"AI 相关配置/上下文"，无需额外解释 |
| **通用性强** | 不绑定特定工具（Cursor、Copilot、Claude 都能用） |
| **未来兼容** | 随着 AI 工具生态发展，`.ai/` 可能成为行业惯例 |

### 4.2 推荐目录结构

```text
.ai/                          # AI 记忆库根目录
├── prd.md                    # 产品需求文档
├── tech-stack.md             # 技术选型
├── architecture.md           # 架构设计
├── implementation-plan.md    # 实现计划
├── progress.md               # 进度日志（每次更新）
└── features/                 # 功能模块文档
    └── feature-xxx.md
```

### 4.3 如何让 AI 读取记忆库？

**核心问题**：AI 不会自动读取这些文件，你需要主动"喂"给它。

有三种方式：

| 方式 | 适用场景 | 操作方法 |
|------|---------|---------|
| **手动引用** | 通用（任何 AI） | 每次对话开头说："请先阅读 `.ai/prd.md`、`.ai/architecture.md`、`.ai/progress.md`" |
| **工具自动加载** | Cursor | 在 `.cursor/rules` 或项目设置中配置自动读取规则 |
| **提示词模板** | 通用 | 把"读取记忆库"写进你的标准开场提示词（见第 11 节） |

> **最佳实践**：把"必须先读记忆库"写进 `.ai/prd.md` 的开头，形成**自引用约束**——AI 一旦读了 PRD，就知道每次都要先读这些文件。

### 4.4 三条硬规则

* **Always Read Rule**：每次开始任务前，AI 必须先读 `.ai/architecture.md`、`.ai/prd.md`、`.ai/progress.md`。
* **Progress is Source of Truth**：每完成一步就更新 `.ai/progress.md`（完成了什么、遇到什么坑、做了什么决策）。
* **One Step, One Commit**：每个 step 完成并验证后立刻 commit，确保随时可回滚。

---

## 5. 标准工作流：Vibe → Plan → Build → Verify → Productionalize

这是一套可以从个人项目一路用到"可上线"的流程。

### 5.1 Vibe：把"感觉"压缩成一页纸

`.ai/prd.md` 建议结构：

* **Goal & Objective**：核心目标与可度量结果
* **Non-goals**：明确不做什么
* **User journeys**：3–5 条核心路径
* **Constraints**：性能/成本/隐私/合规/时间
* **Definition of Done**：什么算成功（可验证）

> 这一步不是写长 PRD，而是写"意图压缩包"。

### 5.2 Plan：让 AI 出实现计划，但你要"把关"

`.ai/implementation-plan.md` 每一步必须包含：

* 改哪些文件/模块
* 新增/修改的接口
* 验收方式（测试/脚本/手工 checklist）
* 风险点（安全、边界、回滚）

一个好用的 step 模板：

```markdown
## Step N: <目标>

### Changes
- Files/modules:
- Public API changes:

### Acceptance
- [ ] Unit tests:
- [ ] Integration / E2E:
- [ ] Manual checklist:

### Risks & Rollback
- Risks:
- Rollback plan:
```

### 5.3 Build：只允许"小步提交"，拒绝一口吞项目

* 一次只做一个 Step
* AI 交付后：你先跑验收（或让它提供运行命令，你执行）
* 不通过：修到通过再进入下一步
* 通过：更新 progress + commit

### 5.4 Verify：把正确性从"感觉"变成"证据"

最靠谱的习惯是：

* **先写验收/测试，再写实现**（至少先写测试骨架与边界用例）
* 每一步都能重复运行：`npm test` / `pytest` / `make test` 等

### 5.5 Productionalize：生产级交付与"去感性重构" (De-vibe Refactoring)

当代码进入准生产环境或用户侧，必须切换到"专业工程模式"：

*   **架构收敛与重构**：执行 **De-vibe Refactoring**，统一目录、抽象层、状态流，消除 AI 随机生成的架构毛刺。
*   **安全加固**：输入校验、权限、依赖审计、密钥管理。
*   **可观测性**：日志、指标、告警、错误追踪。

---

## 6. Feature-specific 文档：加功能不再"拆家"

当 base app/game 完成后，再加大功能（UI、音效、支付、导出、同步……）时最容易失控。
正确姿势是：**每个大功能单独写一个 `.ai/features/feature-xxx.md`**，列出小步骤 + 验收。

模板示例：

```markdown
# Feature: <name>

## Goal
- ...

## Non-goals
- ...

## Design
- UI/UX notes:
- Data flow:
- Error handling:

## Steps
### Step 1 ...
- Acceptance: ...
### Step 2 ...
- Acceptance: ...

## Risks
- ...
```

---

## 7. 卡住与错误处理：别"继续抽卡"，要系统诊断

当 AI 生成出错或你陷入循环时，用这套顺序：

1. **回滚到上一步**（/rewind 的精神）：先回到"最后一个通过验收的 commit"
2. **提供可诊断输入**：完整报错、复现步骤、环境信息、预期 vs 实际
3. **缩小问题面**：让 AI 先解释模块与数据流（/explain），再提出最小 diff 修复方案
4. **必要时全局诊断**：把 repo 汇总成单文件或提供关键目录，让它从整体视角找架构漂移/循环依赖/状态错位

---

## 8. 工具使用策略：控制变量比省钱更重要

社区经验非常务实：**可预测性优先于效率 (Predictability over Efficiency)**。

* **小任务**：要求收敛、少发散、快速验证
* **大任务**：要求先方案、后实现、带风险清单

同时强烈建议"CLI + IDE"组合：

* CLI 看 diff、跑测试、做 commit（节奏更像工程）
* IDE 插件让你更快定位与编辑（效率更像创作）

再加一条"命令化工作流"会非常稳：

* `/plan`：先产出计划
* `/explain`：先解释现状与关键模块
* `/risk`：先列风险与边界
* `/tests`：先生成验收与测试

---

## 9. 关于"Vibe Coding 能不能直接做生产级应用"：正确答案是"取决于你是否工程化"

有人说"完全靠 vibe coding 就能做出 SAP / Salesforce 级系统"——这确实不现实。
原因不是"AI 不够强"，而是生产级软件的成本大头从来不只在"写代码"：

* 边界条件与异常流
* 集成与迁移
* 安全、合规、审计
* 性能、稳定性、可用性
* 长期维护与持续迭代

但这并不否认 Vibe Coding 的价值：

* 它是**强大的原型加速器**：极快验证想法、跑通主路径。
* 经验丰富的工程师使用 AI，确实可能快速完成大量**编码实现**工作量，但仍需要用工程流程兜底。

更贴近现实的定位是：

> **AI 是加速器，不是替代品；
> vibe 是创造力，verify 才是交付力。**

---

## 10. 实战案例：用这套流程做「RSS 智能摘要推送」

光讲方法论太抽象，下面用一个真实场景走一遍完整流程：**做一个 RSS 智能摘要工具，自动抓取订阅源、用 AI 生成摘要、推送到 Slack/邮件**。

### 10.1 Vibe 阶段：写 PRD

```markdown
# Goal
每天早上 8 点，自动抓取 5 个 RSS 源的最新文章，用 AI 生成 3 句话摘要，推送到 Slack。

# Non-goals
- 不做 UI 界面（MVP 用配置文件 + Cron）
- 不做用户系统

# Constraints
- 单次推送消息 <2000 字符（Slack 限制）
- 运行成本 < $5/月
```

### 10.2 Plan 阶段：拆解 Implementation Plan

| Step | 目标 | 验收标准 |
|------|------|---------|
| 1 | RSS 解析模块 | 能解析 3 种格式（RSS 2.0 / Atom / RDF），单元测试覆盖 |
| 2 | AI 摘要模块 | 输入文章内容，输出 ≤3 句话，有 fallback 策略 |
| 3 | Slack 推送 | Webhook 发送成功，消息格式正确 |
| 4 | 调度与集成 | 本地 Cron 运行通过，日志可查 |
| 5 | 生产化 | 部署云函数，添加重试与告警 |

### 10.3 Build 阶段：逐步实现

每完成一个 Step，先跑验收：

```bash
# Step 1 验收
pytest tests/test_rss_parser.py -v

# Step 3 验收
python -c "from slack import send; send('测试消息')"
```

**发现问题**：Step 2 时遇到了"长文章 token 超限"——于是进入 **卡住处理流程**：
1. 回滚到 Step 1 的 commit
2. 让 AI 先解释 token 计算逻辑
3. 增加"分段摘要 + 合并"策略，更新 Plan

### 10.4 三类债的实际暴露

| 债务类型 | 具体表现 | 解决方式 |
|---------|---------|---------|
| 上下文债 | AI 第一版没处理 Atom 格式 | PRD 里明确列出支持格式 |
| 复杂度债 | 生成了 3 个重复的 HTTP 请求封装 | De-vibe 重构，统一到 `utils/http.py` |
| 验证债 | 摘要质量无法自动测 | 加人工 checklist：每周抽查 10 条 |

这个案例浓缩了本文的核心：**Vibe 提速、Plan 兜底、Verify 交付、Productionalize 上线**。

---

## 11. 给你一份"直接可用"的把关提示词

把这段放进你的每次新对话开头（尤其是换 chat 的时候）：

```text
在开始写代码前请执行把关流程：

1) 先阅读 .ai/prd.md、.ai/architecture.md、.ai/progress.md
2) 复述需求（含你不确定的点清单）
3) 给出 implementation plan（小步、每步带验收）
4) 仅在我确认 plan 或 Step 1 验收标准后，才开始生成代码
5) 每次只实现一个 step，输出最小 diff，并告诉我如何运行验收
```

---

## 结语：把 Vibe 留给探索，把工程留给交付

Vibe Coding 最好的用法，不是"省掉工程"，而是**把工程前置与外化**：

* 用规划避免架构漂移
* 用 `.ai/` 记忆库把上下文固定成可重复使用的"项目记忆"
* 用测试与验收把"感觉正确"变成"证据正确"
* 用 commit 把迭代变成"可回滚的进化"
* 用生产化加固把原型变成系统

下面是一套可以直接复制到项目里的 `.ai/` 记忆库初始文件模板：

---

## 附录：`.ai/` 记忆库完整模板

> **使用说明**：在项目根目录创建 `.ai/` 文件夹，将以下模板复制进去。每次与 AI 对话前，提示它先读取这些文件。

### 📄 .ai/prd.md

```markdown
# 产品需求文档 (PRD)

> ⚠️ **AI 必读规则**：每次开始任务前，请先阅读本文件、`architecture.md` 和 `progress.md`，确保理解项目上下文。

## 项目名称
<!-- 填写你的项目名称 -->

## Goal & Objective
<!-- 核心目标是什么？可度量的成功标准是什么？ -->
- 目标：
- 成功指标：

## Non-goals
<!-- 明确不做什么，避免 scope creep -->
- 不做：
- 不做：

## User Journeys
<!-- 3-5 条核心用户路径 -->
1. 用户想要 ___，所以他 ___，最终 ___
2. 用户想要 ___，所以他 ___，最终 ___
3. 用户想要 ___，所以他 ___，最终 ___

## Constraints
<!-- 性能/成本/隐私/合规/时间等约束 -->
- 性能：
- 成本：
- 时间：
- 其他：

## Definition of Done
<!-- 什么算完成？如何验证？ -->
- [ ] 
- [ ] 
- [ ] 
```

---

### 📄 .ai/tech-stack.md

```markdown
# 技术选型

## 语言 & 运行时
- 语言：
- 版本：

## 框架
- 前端：
- 后端：
- 其他：

## 数据存储
- 数据库：
- 缓存：
- 文件存储：

## 基础设施
- 部署平台：
- CI/CD：
- 监控：

## 第三方服务
- 认证：
- 支付：
- 其他 API：

## 开发工具
- IDE：
- AI 工具：
- 其他：
```

---

### 📄 .ai/architecture.md

```markdown
# 架构设计

## 系统概览
<!-- 用一句话描述系统做什么 -->

## 核心模块

### 模块 1: [名称]
- 职责：
- 输入：
- 输出：
- 依赖：

### 模块 2: [名称]
- 职责：
- 输入：
- 输出：
- 依赖：

## 数据流
<!-- 描述数据如何在系统中流动 -->
```
用户输入 → [模块A] → [模块B] → 输出/存储
```

## 目录结构
```
project/
├── src/
│   ├── core/           # 核心业务逻辑
│   ├── api/            # 接口层
│   ├── utils/          # 工具函数
│   └── config/         # 配置
├── tests/              # 测试
├── docs/               # 文档
└── .ai/                # AI 记忆库
```

## 关键设计决策
| 决策 | 选项 | 选择 | 理由 |
|------|------|------|------|
| | | | |

## 已知限制 & 技术债
- 
```

---

### 📄 .ai/implementation-plan.md

```markdown
# 实现计划

## 概览
- 预计总步骤数：
- 当前进度：Step __ / __

---

## Step 1: [目标]

### Changes
- Files/modules：
- Public API changes：

### Acceptance
- [ ] Unit tests：
- [ ] Integration / E2E：
- [ ] Manual checklist：

### Risks & Rollback
- Risks：
- Rollback plan：

---

## Step 2: [目标]

### Changes
- Files/modules：
- Public API changes：

### Acceptance
- [ ] Unit tests：
- [ ] Integration / E2E：
- [ ] Manual checklist：

### Risks & Rollback
- Risks：
- Rollback plan：

---

## Step 3: [目标]

<!-- 按需继续添加... -->
```

---

### 📄 .ai/progress.md

```markdown
# 项目进度日志

> **规则**：每完成一个 Step 就更新本文件。记录：完成了什么、遇到什么坑、做了什么决策。

---

## [日期] - Step N 完成

### ✅ 完成内容
- 

### 🐛 遇到的问题
- 问题：
- 解决：

### 📝 决策记录
- 决策：
- 理由：

### 📌 下一步
- 

---

## [日期] - 项目启动

### ✅ 完成内容
- 初始化项目结构
- 创建 .ai/ 记忆库
- 完成 PRD 和技术选型

### 📌 下一步
- 开始 Step 1
```

---

### 📄 .ai/features/feature-template.md

```markdown
# Feature: [功能名称]

## Goal
<!-- 这个功能要实现什么？ -->
- 

## Non-goals
<!-- 这个功能不做什么？ -->
- 

## Design

### UI/UX Notes
<!-- 界面设计要点 -->
- 

### Data Flow
<!-- 数据如何流动 -->
- 

### Error Handling
<!-- 错误处理策略 -->
- 

## Steps

### Step 1: [子目标]
- Changes：
- Acceptance：

### Step 2: [子目标]
- Changes：
- Acceptance：

## Risks
- 
```

---

**快速开始**：

```bash
# 在项目根目录创建 .ai/ 记忆库
mkdir -p .ai/features

# 创建核心文件（复制上面的模板内容）
touch .ai/prd.md .ai/tech-stack.md .ai/architecture.md
touch .ai/implementation-plan.md .ai/progress.md
touch .ai/features/feature-template.md

# 将 .ai/ 加入版本控制
git add .ai/
git commit -m "chore: init AI memory bank"
```

现在你已经有了一套完整的 Vibe Coding 工程化工具包，开始用它来驯服 AI 吧！
