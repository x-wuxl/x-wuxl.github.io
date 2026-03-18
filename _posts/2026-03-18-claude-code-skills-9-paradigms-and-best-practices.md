---
layout: post
title: "Claude Code Skills 实战指南：构建高效 Skills 的 9 大范式与避坑指南"
date: 2026-03-18
last_modified_at: 2026-03-18
categories: [AI, 开发者工具]
tags: [Claude Code, Anthropic, Agent Skills, 自主智能体, 上下文工程, Context Engineering, 开发者工具, AI 编程, 插件系统, MCP]
keywords: "Claude Code, Agent Skills, 文件夹工程, Folder Engineering, 自主智能体, 上下文工程, Context Engineering, Anthropic, AI 编程助手, 动态钩子, 避坑指南, Gotchas, 模型原生, 自动化运维"
description: "深度剖析 Anthropic 官方在构建 Claude Code 过程中的实战教训。本文揭示了 Agent Skills 如何从简单的提示词进化为包含脚本、数据与动态钩子的‘文件夹级上下文工程’，详述了 9 大核心应用场景及提升智能体可靠性的渐进式披露策略。"
author: wuxl
excerpt: "当 AI Agent 的能力不再仅依赖于提示词，而是内化为一套可发现、可操作的结构化 Skills 时，软件开发范式正发生深刻变革。本文基于 Anthropic 内部数百个 Skills 的实战经验，总结了从库参考、产品验证到基础设施操作的工程化最佳实践，助你构建真正可靠的长程自主智能体。"
toc: true
toc_sticky: true
---


# 构建 Claude Code 的经验教训：我们如何使用 Skills（技能）

Skills 已经成为 Claude Code 中最常用的扩展点之一。它们灵活、易于制作且便于分发。但这种灵活性也使得我们很难判断什么才是最有效的。什么类型的 skills 值得制作？编写一个优秀 skill 的秘诀是什么？什么时候应该与他人分享？

我们在 Anthropic 的 Claude Code 中广泛使用了 skills，目前有数百个在活跃使用中。以下是我们从使用 skills 加速开发的过程中学到的经验教训。

## 什么是 Skills？

如果您对 skills 还不熟悉，我建议您阅读我们的文档或观看我们关于 Agent Skills 的新课程。本文将假设您已经对 skills 有所了解。关于 skills，我们常听到的一个误解是它们“只是一些 markdown 文件”，但 skills 最有趣的地方在于它们不仅仅是文本文件。它们是**文件夹**，可以包含脚本、资产、数据等，AI 代理（Agent）可以发现、探索并操作这些内容。

在 Claude Code 中，skills 还具有各种配置选项，包括注册动态钩子（hooks）。我们发现 Claude Code 中一些最有趣的 skills 都在创造性地利用这些配置选项和文件夹结构。

## Skills 的类型

在对我们所有的 skills 进行分类后，我们发现它们大多集中在几个常见的类别中。最好的 skills 通常能清晰地归入某一类；而比较容易让人混淆的 skills 往往跨越了多个类别。这并不是一个绝对的清单，但它可以很好地帮助你思考你的组织内部是否遗漏了某些类别。

### 1. 库与 API 参考 (Library & API Reference)
这类 skills 解释如何正确使用某个库、CLI 或 SDK。这些既可以是内部库，也可以是 Claude Code 有时会遇到困难的通用库。这类 skills 通常包含一个参考代码片段的文件夹，以及一个让 Claude 在编写脚本时避开的“容易犯错的陷阱（gotchas）”列表。
* **示例：**
    * `billing-lib` — 你的内部计费库：边缘用例（edge cases）、容易犯错的陷阱（footguns）等。
    * `internal-platform-cli` — 你内部 CLI 包装器的每一个子命令，以及关于何时使用它们的示例。
    * `frontend-design` — 让 Claude 更好地掌握你的设计系统。

### 2. 产品验证 (Product Verification)
描述如何测试或验证代码是否正常运行。通常与 Playwright、tmux 等外部工具配合使用来进行验证。验证 skills 对确保 Claude 输出的正确性非常有用。让一位工程师花一周时间专门把你们的验证 skills 做到极致是值得的。可以考虑一些技巧，比如让 Claude 录制其输出的视频，这样你就能清楚看到它测试了什么；或者在每一步对状态强制执行程序化断言。这些通常通过在 skill 中包含各种脚本来实现。
* **示例：**
    * `signup-flow-driver` — 在无头浏览器中运行 注册 → 邮箱验证 → 入职引导 流程，并带有在每一步断言状态的钩子。
    * `checkout-verifier` — 使用 Stripe 测试卡驱动结账 UI，验证发票确实落入正确的状态。
    * `tmux-cli-driver` — 用于交互式 CLI 测试，适用于你需要验证的内容需要 TTY 的情况。

### 3. 数据获取与分析 (Data Fetching & Analysis)
连接到你的数据和监控技术栈。这些 skills 可能包含带有凭证的数据获取库、特定的仪表盘 ID 等，以及关于常见工作流或获取数据方式的说明。
* **示例：**
    * `funnel-query` — “我应该连接哪些事件来查看 注册 → 激活 → 付费”，以及实际包含规范 `user_id` 的表。
    * `cohort-compare` — 比较两个群组的留存率或转化率，标记出具有统计学意义的差异，并链接到群组的细分定义。
    * `grafana` — 数据源 UID、集群名称，以及 问题 → 仪表盘 的查找表。

### 4. 业务流程与团队自动化 (Business Process & Team Automation)
将重复性的工作流程自动化为一个简单的命令。这些 skills 的指令通常相对简单，但可能对其他 skills 或 MCP 有复杂的依赖。对于这类 skills，将之前的结果保存在日志文件中，可以帮助模型保持一致性，并反思该工作流以前的执行情况。
* **示例：**
    * `standup-post` — 汇总你的工单追踪器、GitHub 活动以及之前的 Slack 记录 → 生成格式化的站会内容，仅包含增量变化。
    * `create-<ticket-system>-ticket` — 强制执行 Schema（有效的枚举值、必填字段）加上创建后的工作流（在 Slack 中提醒审查者并附带链接）。
    * `weekly-recap` — 已合并的 PR + 已关闭的工单 + 部署记录 → 生成格式化的每周回顾帖子。

### 5. 代码脚手架与模板 (Code Scaffolding & Templates)
为代码库中特定的功能生成框架样板文件。你可以将这些 skills 与可组合的脚本结合使用。当你的脚手架有无法纯粹用代码覆盖的自然语言要求时，它们特别有用。
* **示例：**
    * `new-<framework>-workflow` — 使用你的注释为新服务/工作流/处理程序生成脚手架。
    * `new-migration` — 你的迁移文件模板以及常见的易错陷阱。
    * `create-app` — 预先配置好身份验证、日志记录和部署配置的新内部应用。

### 6. 代码质量与审查 (Code Quality & Review)
在组织内部强制执行代码质量并协助审查代码。可以包含确定性的脚本或工具以保证最大的稳健性。你可能希望作为钩子的一部分或在 GitHub Action 内部自动运行这些 skills。
* **示例：**
    * `adversarial-review` — 生成一个以全新视角审视的子代理（subagent）来进行批评、实施修复，并不断迭代直到发现的问题只剩下一些吹毛求疵的细节（nitpicks）。
    * `code-style` — 强制执行代码风格，特别是 Claude 默认做得不好的风格。
    * `testing-practices` — 关于如何编写测试以及测试哪些内容的说明。

### 7. CI/CD 与部署 (CI/CD & Deployment)
帮助你在代码库中获取、推送和部署代码。这些 skills 可能会引用其他 skills 来收集数据。
* **示例：**
    * `babysit-pr` — 监控 PR → 重试不稳定的 CI → 解决合并冲突 → 启用自动合并。
    * `deploy-<service>` — 构建 → 冒烟测试 → 带有错误率比较的逐步流量发布 → 出现衰退时自动回滚。
    * `cherry-pick-prod` — 隔离的工作树 → 摘取（cherry-pick）提交 → 冲突解决 → 带有模板的 PR。

### 8. 运行手册 (Runbooks)
接收一种症状（例如 Slack 帖子、警报或错误签名），引导执行多工具组合的调查，并生成结构化的报告。
* **示例：**
    * `<service>-debugging` — 将症状映射到工具，再映射到针对你最高流量服务的查询模式。
    * `oncall-runner` — 获取警报 → 检查常见问题源头 → 格式化输出发现的问题。
    * `log-correlator` — 给定一个请求 ID，从所有可能接触过该请求的系统中提取匹配的日志。

### 9. 基础设施操作 (Infrastructure Operations)
执行日常维护和操作程序——其中一些涉及破坏性操作，因此有安全护栏会更好。这些让工程师在关键操作中更容易遵循最佳实践。
* **示例：**
    * `<resource>-orphans` — 查找孤立的 Pod/卷 → 发送到 Slack → 等待期 → 用户确认 → 级联清理。
    * `dependency-management` — 你所在组织的依赖审批工作流。
    * `cost-investigation` — “为什么我们的存储/出口账单激增”，包含具体的存储桶和查询模式。

## 制作 Skills 的技巧

一旦你决定了要制作什么 skill，该如何编写它呢？以下是我们发现的一些最佳实践和技巧。我们最近还发布了 [Skill Creator](https://claude.com/blog/improving-skill-creator-test-measure-and-refine-agent-skills)，让在 Claude Code 中创建 skills 变得更简单。

### 不要陈述显而易见的事物
Claude Code 已经非常了解你的代码库，Claude 也非常懂编程，包括许多默认的观点。如果你要发布一个偏重知识的 skill，尽量专注于**促使 Claude 跳出其固有思维方式的信息**。[frontend design skill](https://github.com/anthropics/skills/blob/main/skills/frontend-design/SKILL.md) 就是一个很好的例子——它是 Anthropic 的一位工程师通过与客户不断迭代来提高 Claude 设计品味而构建的，旨在避开像 Inter 字体和紫色渐变这样的经典陈旧模式。

### 建立“避坑（Gotchas）”部分
任何 skill 中最具价值的内容就是“避坑”部分。这些部分应基于 Claude 使用你 skill 时常见的失败节点来建立。理想情况下，你应该随着时间的推移更新你的 skill 以捕捉这些陷阱。

### 使用文件系统和渐进式披露
就像我们之前说的，skill 是一个文件夹，而不仅仅是一个 markdown 文件。你应该把整个文件系统视为一种上下文工程和渐进式披露。告诉 Claude 你的 skill 里有哪些文件，它会在适当的时候读取它们。渐进式披露最简单的形式就是指向其他 markdown 文件让 Claude 使用。例如，你可以将详细的函数签名和用法示例分离到 `references/api.md` 中。另一个例子：如果你的最终输出是一个 markdown 文件，你可能会在 `assets/` 目录中包含一个模板文件供其复制和使用。你可以拥有参考、脚本、示例等文件夹，这有助于 Claude 更有效地工作。

### 避免强制限制（Railroading） Claude
Claude 通常会尽量遵守你的指令，因为 Skills 的可复用性很强，所以你要小心不要把指令写得太具体。给 Claude 提供所需的信息，但要给予它根据情况进行调整的灵活性。

### 仔细考虑设置阶段
有些 skills 可能需要用户提供上下文来进行设置。例如，如果你正在制作一个将站会内容发布到 Slack 的 skill，你可能需要 Claude 询问发布到哪个 Slack 频道。一个好的模式是将这些设置信息存储在 skill 目录下的 `config.json` 文件中。如果配置未设置，代理可以提示用户提供信息。如果你希望代理提出结构化的多项选择题，你可以指示 Claude 使用 `AskUserQuestion` 工具。

### “描述 (Description)”字段是给模型看的
当 Claude Code 启动会话时，它会建立一个包含所有可用 skills 及其描述的列表。Claude 通过扫描这个列表来决定“是否有针对当前请求的 skill？” 这意味着描述字段不是一个总结——它是用来描述何时触发此 PR（注：此处原文 PR 可能是指流程或动作，但原意是何时触发这个 skill）的说明。

### 记忆与数据存储
一些 skills 可以通过在内部存储数据来实现某种形式的记忆。你可以将数据存储在简单的只追加文本日志文件或 JSON 文件中，或者复杂到 SQLite 数据库中。例如，一个 `standup-post` skill 可能会保留一个包含它写过的每篇帖子的 `standups.log`，这意味着下次运行它时，Claude 可以读取自己的历史记录，并能分辨出自昨天以来发生了什么变化。当您升级 skill 时，存储在 skill 目录中的数据可能会被删除，因此您应将其存储在稳定的文件夹中。目前我们提供 `${CLAUDE_PLUGIN_DATA}` 作为每个插件用于存储数据的稳定文件夹。

### 存储脚本并生成代码
你能给 Claude 最强大的工具就是代码。给 Claude 提供脚本和库可以让 Claude 将它的交互轮次花在组合上，决定下一步该做什么，而不是重构样板代码。例如，在你的数据科学 skill 中，你可能有一个函数库来从你的事件源获取数据。为了让 Claude 进行复杂的分析，你可以给它提供一组辅助函数。随后 Claude 就可以动态生成脚本来组合这些功能，为诸如“星期二发生了什么？”之类的提示进行更高级的分析。

### 按需触发的 Hooks
Skills 可以包含仅在调用该 skill 时激活、并在会话持续期间有效的钩子（Hooks）。将此用于那些你不想一直运行，但有时极其有用的主观倾向性更强的钩子。
* **示例：**
    * `/careful` — 通过 Bash 上的 PreToolUse 匹配器阻止 `rm -rf`、`DROP TABLE`、`force-push`、`kubectl delete`。你只有在知道自己要操作生产环境时才需要这个——如果它一直开启会把你逼疯的。
    * `/freeze` — 阻止任何不在特定目录下的编辑/写入操作。在调试时非常有用：“我想添加日志，但我总是意外地‘修复’了不相关的代码”。

## 分发 Skills
Skills 最大的好处之一就是你可以将它们分享给团队的其他成员。有两种分享方式：
1. 将 skills 提交到你的代码库中（存放在 `./.claude/skills` 下）。
2. 制作一个**插件（plugin）**，并建立一个 Claude Code 插件市场，用户可以在那里上传和安装插件。

对于在相对较少的代码库中工作的小团队来说，将 skills 提交到代码库非常有效。但每一个提交的 skill 也会稍微增加模型的上下文。随着规模扩大，内部插件市场允许你分发 skills，并让你的团队自行决定安装哪些。

## 管理市场
你如何决定哪些 skills 进入市场？人们如何提交它们？我们没有一个集中化的团队来决定；相反，我们试图有机地寻找最有用的 skills。如果你有一个想让人们尝试的 skill，你可以将其上传到 GitHub 中的沙盒文件夹，并在 Slack 或其他论坛中将人们引导至那里。一旦一个 skill 获得了足够的关注度（这由 skill 所有者决定），他们可以提交 PR 将其移入市场。需要提醒的是，创建糟糕或冗余的 skills 是很容易的，因此在发布之前确保有某种策展审核方法非常重要。

## 组合 Skills
你可能希望有相互依赖的 skills。例如，你可能有一个用于上传文件的 `file upload` skill，以及一个生成 CSV 并上传它的 `CSV generation` skill。这种依赖管理目前尚未原生构建到市场或 skills 中，但你可以直接通过名称引用其他 skills，如果模型安装了它们，就会自动调用。

## 衡量 Skills
为了了解 skill 的运行情况，我们使用了一个 `PreToolUse` 钩子，让我们能够记录公司内部的 skill 使用情况。这意味着我们可以发现哪些 skills 很受欢迎，或者哪些 skills 的触发率低于我们的预期。

## 结论
Skills 对于代理来说是极其强大、灵活的工具，但这仍在早期阶段，我们都在探索如何最好地使用它们。请将此视为一个关于我们发现的有效技巧的锦囊，而不是权威指南。理解 skills 最好的方法就是开始动手、尝试，看看什么对你有用。我们的大多数 skills 最初只是几行代码和一个容易犯错的陷阱，由于人们在 Claude 遇到新的边缘情况时不断补充，它们变得越来越好。

希望这对您有帮助，如果您有任何问题请告诉我。

[原文链接](https://x.com/trq212/status/2033949937936085378)