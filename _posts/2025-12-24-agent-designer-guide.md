---
layout: post
title: "Agent Designer 深度指南：文档驱动的 AI Agent 自动化设计流程"
date: 2025-12-24 09:27:00 +0800
author: x-wuxl
categories: [AI 工具, 自动化]
tags: [Agent Designer, AI Agent, Agentic Workflow, Codex, Claude Code, MCP]
description: "深入了解 Agent Designer 项目：一种基于文档驱动、设计优先的 AI Agent 开发标准。本文将带你掌握如何利用这一框架实现跨平台的 AI 技能标准化，并优化你的 Agentic Workflow。"
seo_title: "Agent Designer 教程：文档驱动的 AI Agent 开发与自动化"
seo_description: "探索 Agent Designer 如何通过文档驱动和设计优先的理念，为 Codex 和 Claude Code 提供统一的 AI 技能开发标准。学习 3 步快速入门及核心 E2E 工作流。"
---

# Agent Designer 深度指南：文档驱动的 AI Agent 自动化设计流程

[Agent Designer](https://github.com/appautomaton/agent-designer) 是一个由文档驱动的工作区，旨在标准化和简化 AI Agent “技能”（Skills）的创建、维护和执行。它特别针对 **Codex CLI (Google)** 和 **Claude Code (Anthropic)** 进行了优化，通过统一的“技能”标准桥接了主流的 Agent 框架。

## 核心理念：设计优先 & 文档驱动

传统的 AI 交互往往依赖于零散的 Prompt 工程，而 Agent Designer 提倡一种**代理工程（Agentic Engineering）**的方法：

- **设计中心化（Design-Centric）**：Agent 的能力被视为正式的资产。每一个技能都由一个 `SKILL.md` 文件定义，作为用户与 LLM 之间的可移植契约。
- **文档先行（Documentation-First）**：绝大多数项目变更都以文本形式发生。仓库本身即是 Agent 行为、规则和工具集的“真相之源”。

---

## 快速上手：三步开启新项目

如果你想快速将这一框架应用到新项目中，可以遵循以下三个核心步骤：

### 第一步：克隆与清理
首先将项目克隆到本地，并移除原有的 Git 信息：
```bash
git clone https://github.com/appautomaton/agent-designer.git [your-project-name]
cd [your-project-name]
rm -rf .git
```

### 第二步：生成 MCP 工具清单
由于 Codex 等工具在调用 MCP（Model Context Protocol）时可能存在滞后性，推荐先行生成一份工具清单。
- **目标技能**：`./.codex/skills/mcp-tools-catalog/`
- **目的**：明确列出所有可用工具，让 Agent 在规划（Planning）阶段能“指哪打哪”。

### 第三步：项目初始化与 Agents 引导
使用 `agents-bootstrap` 技能，通过 Artifacts 技术根据项目需求生成核心框架。
- **目标技能**：`./.codex/skills/agents-bootstrap/`
- **操作建议**：先与 Codex 充分交流，明确大方向，然后下达指令：`给我写个 AGENTS.md, bootstrap my AGENTS.md`。这会将由 Issues 主导的开发模式框架导入到 `AGENTS.md` 中。

---

## 核心工作流：从规划到执行

在完成初始化后，即可进入正式的开发规划阶段：

1. **自动化规划**：利用 `./.codex/skills/plan` 技能进行 Epic 级别的规划。它会覆盖默认配置，生成符合项目框架的任务清单。
2. **模板化输出**：项目自带了成熟的规划模板：
   - `.codex/skills/plan/assets/_template.csv` (任务跟踪)
   - `.codex/skills/plan/assets/_template.md` (详细计划)
3. **Issue 主导的开发**：所有的执行都基于 Issue 追踪，确保每一步都有验收标准（Acceptance Criteria）和验证方法。

## 总结

Agent Designer 不仅仅是一个工具库，它代表了 AI 交互从“试错模式”向“结构化开发模式”的转变。通过定义清晰的技能契约和严谨的 E2E 循环，它让开发者能够构建出跨平台、可移植且高可靠的 AI Agent 解决方案。
