---
layout: post
title: "从提示词工程到上下文工程：自主智能体架构演进全景解析"
date: 2026-03-16
last_modified_at: 2026-03-16
categories: [AI, 技术分析]
tags: [自主智能体, 上下文工程, Context Engineering, Deep Agents, MCP, Agent Skills, rLLM, 强化学习, Manus AI, Claude Code, Kimi CLI, KV Cache, 模型原生]
keywords: "上下文工程, Context Engineering, 自主智能体, Deep Agents, Agent 2.0, 模型原生架构, MCP 协议, Agent Skills, rLLM, 强化学习, Manus AI, Claude Code, Kimi CLI, KV Cache 优化, 上下文腐烂, 提示词工程"
description: "深度剖析 AI Agent 架构从管道式到模型原生的范式转移，详解上下文工程（Context Engineering）核心策略、Deep Agents 2.0 架构模式、MCP 与 Agent Skills 之争、rLLM 强化学习训练闭环，以及 Manus AI、Claude Code、Kimi CLI 等产品的实战经验。"
author: wuxl
excerpt: "AI Agent 正从提示词驱动的管道式架构，向能力内化于参数的模型原生智能体演进。本文系统性地解析上下文工程、Deep Agents 架构、工具协议之争及 RL 训练闭环等前沿工程实践，并结合 Manus AI、Claude Code 等生产案例，揭示构建可靠长程 Agent 的核心方法论。"
toc: true
toc_sticky: true
---

> **本文导读**：本文系统性地解析 AI Agent 从"提示词工程"走向"上下文工程"的技术演进全貌。全文共 7 章：第 1-2 章阐述范式转移与上下文工程核心理论；第 3 章深入 Deep Agents 2.0 的工程架构；第 4 章对比 MCP 与 Agent Skills 两种工具协议；第 5 章解析 rLLM 强化学习训练闭环；第 6 章通过 Kimi CLI、Claude Code、Manus AI 等产品进行实战分析；第 7 章展望未来趋势。

## 1. 范式转移：从管道式系统到模型原生智能

人工智能领域正经历着一场根本性的结构变革，从依赖外部硬编码逻辑的"管道式"（Pipeline-based）系统，向能力内化于参数的"模型原生"（Model-native）自主智能体演进。本报告基于 GitHub issue #150 中汇总的 Anthropic、LangChain、Manus AI、rLLM 等前沿工程资料，对这一领域的工程实践、架构模式及未来趋势进行详尽的深度剖析 [^1]。

### 1.1 管道式架构的局限与"苦涩的教训"

在过去几年中，构建 Agent 的主流方法是基于"浅层循环"（Shallow Loops）的管道式架构。这种架构将大语言模型（LLM）视为一个功能组件——一个被外部手工逻辑编排的"缸中之脑"。在这种范式下，智能体的策略函数是复合的，严重依赖于外部设计的结构来实现规划（如思维链 Prompt）、工具使用（如 ReAct 循环）和记忆（如向量数据库检索）[^2]。

虽然这种架构在短周期任务中表现尚可，但在长周期任务中暴露出了极大的脆弱性。Philschmid 将这类系统定义为"Agent 1.0"，指出它们仅仅是在执行简单的"观察-思考-行动"循环。当任务步骤超过 50 步，或者涉及跨越数小时的复杂工作流时，这种架构面临"上下文溢出"（Context Overflow）、目标丢失以及错误累积等致命问题 [^3]。

这一现象验证了 Rich Sutton 提出的"苦涩的教训"（The Bitter Lesson）：在 AI 历史长河中，研究人员强加给模型的"结构"（如归纳偏置或特定领域的规则）最终都会成为瓶颈，而那些利用通用计算能力进行学习的方法（如搜索和学习）最终总是胜出 [^1]。Lance Martin 的实践表明，早期为了规避模型能力不足而设计的复杂"编排者-工作者"（Orchestrator-Worker）硬编码流程，随着模型能力的提升，反而限制了系统的灵活性和上限。当模型（如 Claude 3.5 Sonnet 或 DeepSeek-R1）开始具备内化的规划能力时，外部的脚手架就应当被拆除 [^1]。

### 1.2 模型原生智能的崛起

"模型原生"范式主张将规划、工具使用和记忆管理等核心能力直接内化到模型的参数中。在这种架构下，智能体的策略变得单体化。模型不再是简单地遵循提示词去调用工具，而是通过大规模强化学习（RL）习得了使用工具和规划的最优策略 [^5]。

| 核心能力 | 管道式范式 (Pipeline-based) | 模型原生范式 (Model-native) |
|:---|:---|:---|
| **规划 (Planning)** | 外部符号规划器 (LLM+PDDL)、思维链提示词 (CoT)、思维树 (ToT)。依赖人工设计的提示模板。 | 通过强化学习内化的推理能力 (如 OpenAI o1, DeepSeek-R1)。模型自主生成思维过程，无需显式提示。 |
| **工具使用 (Tool Use)** | ReAct 循环、外部编排逻辑。需要详细的工具描述和少样本示例 (Few-shot)。 | 端到端的习得行为 (如 OpenAI o3, Moonshot K2)。模型在预训练或后训练阶段已学会何时及如何调用工具。 |
| **记忆 (Memory)** | RAG、向量存储、滑动窗口摘要。记忆作为外部检索对象。 | 参数化记忆、主动上下文管理 (如 MemAct)。模型学习决定保留或丢弃哪些信息。 |

> **表 1**：管道式与模型原生智能体范式的深度对比 [^2]

这一转变的核心驱动力是强化学习。正如 DeepSeek-R1 和 OpenAI o1 所展示的，通过结果导向的奖励机制（Outcome-based Rewards），模型可以自主探索出解决复杂问题的路径，而无需昂贵的逐步监督数据。这标志着 Agent 开发从"如何设计更好的 Prompt"转向了"如何设计更好的奖励函数和训练环境" [^5]。

---

## 2. 上下文工程：注意力经济的精细化管理

随着智能体被赋予跨越数小时甚至数天的长周期任务，"上下文工程"（Context Engineering）已取代简单的提示词工程，成为构建可靠 Agent 的核心学科。Anthropic 将上下文工程定义为：为了优化效用并防止"上下文腐烂"（Context Rot），对上下文窗口中的 Token 集合进行精细化策划与管理的实践 [^1]。

### 2.1 上下文腐烂与注意力预算

Transformer 架构的注意力机制虽然强大，但其本质上是在有限的"注意力预算"下运作的。随着上下文窗口的填充，模型对特定信息的召回和推理能力会逐渐下降，这种现象被称为"上下文腐烂"或"迷失中间"（Lost-in-the-Middle）效应 [^1]。

Manus AI 的生产环境数据显示，Agent 系统的计算特征与传统 Chatbot 截然不同：其平均输入输出 Token 比率高达 **100:1** [^7]。这意味着系统绝大部分的算力消耗在"阅读"和"维持状态"上，而非"生成"上。在长达 50 次工具调用的任务中，早期的一个错误工具调用所产生的噪音数据，会通过自回归机制在后续步骤中不断放大，导致整个任务轨迹的偏离。

### 2.2 KV 缓存优化：经济可行性的基石

在以输入为主的计算负载下，KV Cache（键值缓存）的命中率直接决定了 Agent 的响应延迟和经济可行性。Manus AI 的工程实践揭示了一个关键细节：系统提示词（System Prompt）前缀的微小变化（哪怕是一个 Token）都会导致后续所有内容的缓存失效 [^7]。

例如，许多开发者习惯在 System Prompt 开头加入当前时间戳（精确到秒）。这种做法会导致每次请求的前缀都不相同，从而无法利用预填充（Prefill）缓存。在 Claude Sonnet 等模型上，未缓存的输入 Token 成本是缓存 Token 的 10 倍（$3.00 vs $0.30 / 百万 Token）[^7]。因此，上下文工程的第一原则是**保持前缀的静态稳定性**，将动态信息（如时间、用户名称）后置或通过工具动态获取，以最大化缓存复用率。

### 2.3 高级上下文管理策略

为了在有限的窗口内维持长时间的高性能，业界已经形成了一套成熟的上下文管理策略：

#### 2.3.1 渐进式披露 (Progressive Disclosure)

这是一种从 UI 设计借鉴而来的策略。Agent 不应在初始上下文中加载所有可用的工具定义或文档，因为这会造成"上下文分心"（Context Distraction）。相反，Agent 仅持有轻量级的索引或元数据（如文件名列表、工具名称）。当且仅当任务需要时，Agent 才会通过工具（如 `read_file` 或 `fetch_tool_definition`）动态加载详细内容 [^1]。这种"即时编译"（JIT）式的上下文加载方式，确保了工作记忆始终保持在最低限度的高信噪比状态。

#### 2.3.2 上下文压缩与"移盆" (Compaction & Re-potting)

当上下文窗口接近上限（如 95%）时，必须进行压缩。Anthropic 建议采用"移盆"策略——如同将根系过密的植物移入新盆：不是简单地截断旧消息，而是让模型对之前的交互轨迹进行高保真的总结，保留关键决策点和当前状态，然后以此作为新的起点启动一个新的上下文窗口 [^1]。Manus AI 进一步实施了"可恢复压缩"（Recoverable Compression），将网页抓取的原始 HTML 数据保存到文件系统中，在上下文中仅保留摘要和文件路径。如果 Agent 发现摘要不足以回答问题，它可以通过读取文件"恢复"原始上下文，实现了上下文的冷热分层存储 [^10]。

#### 2.3.3 朗诵法 (Recitation) 与注意力控制

Manus AI 发现了一种对抗"迷失中间"效应的有效方法——"朗诵"（Recitation）。在每一轮交互结束时，强制 Agent 将当前的全局计划（Global Plan）或待办事项列表（Todo List）重新写入到上下文的最末端 [^8]。这种做法利用了 Transformer 对"最近 Token"的高权重注意力偏置。通过不断地"朗诵"目标，Agent 实际上是在反复自我提醒"我正在做什么"以及"下一步该做什么"，从而在长序列交互中保持目标的一致性，防止因中间步骤的繁琐细节而导致的"目标漂移" [^8]。

---

## 3. 深度智能体架构 (Deep Agents)：长程任务的工程"挽具"

METR 的研究表明，自主 Agent 完成"长任务"（Long Tasks）的能力（以人类完成该任务所需时间衡量）正以**每 7 个月翻一番**的速度增长 [^1]。目前的前沿模型已能可靠完成人类需耗时约 1 小时的任务。为了支撑这种能力的指数级增长，系统架构必须从简单的 while 循环升级为更复杂的"挽具"（Harnesses）。

### 3.1 深度智能体 (Deep Agents / Agents 2.0) 的核心特征

LangChain 和 Philschmid 将这种新一代架构称为"深度智能体"或"Agent 2.0"。其核心思想是将**规划**与**执行**解耦，并将**状态**外部化 [^1]。

#### 3.1.1 显式规划与 write_todos

与依赖隐式思维链（CoT）不同，Deep Agents 维护一个显式的、持久化的计划文档（通常是一个 Markdown 格式的 Todo List）。LangChain 的 `write_todos` 工具不仅仅是一个记录板，它是 Agent 的"指挥棒" [^3]。在执行任何具体操作之前，Agent 必须先更新这个计划，标记哪些已完成，哪些待处理。这种机制迫使 Agent 显式地进行状态确认，避免了在复杂的递归任务中陷入死循环。Anthropic 的研究也证实，对于涉及数百个步骤的编码任务，让 Agent 维护一个 `feature_list.json` 是防止其"过早宣布胜利"的关键 [^1]。

#### 3.1.2 文件系统作为长期记忆

传统的 Chatbot 依赖上下文窗口作为记忆，这极其脆弱且易失。Deep Agents 引入了真实的文件系统作为"外脑" [^8]：

- **持久化：** Agent 将中间思考、搜索结果、代码草稿写入 `NOTES.md` 或 `scratchpad.txt`。即使上下文被重置，Agent 也可以通过读取这些文件瞬间恢复"记忆"。
- **状态隔离：** 文件系统天然支持模块化。Agent 可以为不同的子任务创建不同的目录，避免了单一上下文中信息的混杂。
- **无限扩展：** 相比于昂贵的 Token 空间，磁盘空间极其廉价且几乎无限。Manus AI 将文件系统视为"终极上下文"，任何无法放入窗口的信息都应当被"卸载"（Offload）到文件中 [^8]。

#### 3.1.3 子智能体 (Sub-Agents) 与分层架构

为了解决上下文污染问题，Deep Agents 采用了"分而治之"的策略。主智能体（Orchestrator）不直接执行细节工作，而是将任务分解并委派给专门的子智能体（Sub-Agents）[^1]。每个子智能体拥有独立的、干净的上下文窗口。例如，一个负责"市场调研"的子智能体可以在其独立的会话中阅读数百页的报告，消耗数万 Token，最终只向主智能体返回一个 500 字的精炼摘要。这种"Map-Reduce"式的架构极大地节省了主流程的注意力预算，并提高了任务的准确性。Anthropic 的多智能体研究系统显示，这种架构在复杂任务上的表现比单体 Agent 提升了 **90.2%** [^1]。

### 3.2 针对长程编码任务的初始化器/编码器模式

Anthropic 在构建 Claude Code 时总结出了一套针对软件工程任务的特定模式：**初始化器/编码器双角色模式** [^1]。

- **初始化器 (Initializer Agent)：** 仅在任务开始时运行一次。它负责"冷启动"，分析用户需求，搭建环境（如生成 `init.sh` 脚本），并创建一个详尽的 `feature_list.json`。它的作用是确立"基本事实"（Ground Truth）。
- **编码器 (Coding Agent)：** 这是一个在后续会话中反复运行的角色。它读取 `feature_list.json`，选择一个未完成的任务，执行编码，运行测试，提交代码，并更新状态。关键在于，编码器 Agent 并不依赖之前的对话历史来了解项目进度，而是依赖文件系统中的状态文件。这意味着即使中途更换了模型版本，或者上下文完全丢失，新的编码器实例也能无缝接手工作 [^1]。

### 3.3 沙盒化与安全性

随着 Agent 获得执行代码和操作文件系统的能力，安全性成为重中之重。

- **执行沙盒：** Manus AI 和 Claude Code 都强调在严格隔离的环境中运行代码。Manus 使用虚拟机（VM），而 Claude Code 提供了基于 Docker 或 Linux Bubblewrap 的沙盒 [^1]。网络访问通常被限制在白名单内，且所有文件操作只能发生在特定目录。
- **工具掩码 (Tool Masking)：** Manus AI 引入了一种基于 Logit Masking 的安全机制。系统不是在 Prompt 中移除危险工具（这会破坏 KV Cache），而是在推理阶段，通过调整 Logit 概率，强制模型在特定状态下无法生成调用危险工具的 Token。这种"软限制"既保证了安全，又维护了推理效率 [^7]。

---

## 4. 工具协议之争：MCP vs. Agent Skills

在 Agent 如何连接外部世界这一问题上，业界存在着两种截然不同的理念：结构化的**模型上下文协议 (MCP)** 与文本化的 **Agent Skills**。

### 4.1 模型上下文协议 (MCP)：标准化的尝试

Anthropic 推出的 MCP 旨在成为"AI 时代的 USB 接口"。它通过标准化的 JSON-RPC 协议，允许 Agent 连接到本地或远程的服务器（如 Google Drive, Slack, GitHub）[^1]。

- **优势：** 提供了统一的连接标准，IDE（如 Cursor）和模型提供商（如 Anthropic）提供了原生支持。抽象了认证和连接的复杂性。
- **劣势：**
  - **上下文臃肿：** 加载一个企业级工具（如 Jira）的完整 Schema 可能消耗数万 Token。
  - **运维复杂：** 需要运行和维护 MCP 服务器进程，增加了系统的脆弱性。
  - **安全性隐患：** 批评者指出 MCP 缺乏内在的安全执行机制，且容易遭受"工具投毒"或"提示词注入"攻击 [^14]。

### 4.2 Agent Skills：文本即接口

以 Armin Ronacher 为代表的开发者和"Agent Skills"项目提倡一种更轻量级、更符合 LLM 本质的方法——**"技能"（Skills）**[^9]。

- **核心理念：** 技能本质上是文件系统中的文本文件（`SKILL.md`）。每个技能包含元数据（名称、描述）和详细的使用说明（Prompt、脚本、示例）。
- **工作流：**
  1. Agent 初始只加载技能的**元数据**列表。
  2. 当 Agent 认为某个技能相关时，它使用 `read_file` 工具读取该技能的 `SKILL.md`。
  3. Agent 根据文档中的"教程"学习如何操作，或者直接执行文档中引用的脚本（如 Python 脚本）。
- **优势：**
  - **按需加载：** 极大地节省了 Token。
  - **灵活性：** 相比于死板的 JSON Schema，自然语言文档更能教会模型处理边缘情况（Edge Cases）和使用技巧（Tips）。
  - **去中心化：** 不需要中心化的服务器，仅仅是文件的读写。

这种"文本即接口"（Text-as-Interface）的复兴暗示了一个深刻的洞察：对于概率性的 LLM 而言，模糊但描述性强的自然语言接口，往往比精确但脆弱的结构化接口更具鲁棒性 [^1]。

---

## 5. 训练智能体：rLLM 与强化学习闭环

如果说上下文工程是在"推理时"优化 Agent，那么强化学习（RL）则是在"训练时"从根本上提升 Agent 的能力。rLLM 项目的出现标志着 Agent 开发进入了类似于"DeepSeek-R1"训练流程的工程化阶段 [^6]。

### 5.1 rLLM SDK 的架构设计

rLLM 提供了一个端到端的框架，用于对 Agent 工作流进行后训练（Post-training）。其架构由三个关键组件构成闭环：

1. **采样器 (Sampler / AgentExecutionEngine)：** 这是一个高性能的执行引擎，负责在并行环境中运行 Agent，生成"轨迹"（Trajectories）。它不仅仅是调用 API，而是拦截并记录 Agent 的每一次思考（Thought）、工具调用（Action）和环境反馈（Observation）[^6]。
2. **评估器 (Reward Function)：** 在 Agent RL 中，奖励函数的设计至关重要。
   - **结果奖励 (Outcome Reward)：** 最终结果是否正确？（例如：单元测试通过为 1.0，失败为 0.0）。
   - **过程奖励 (Process Reward)：** 步骤是否合规？是否使用了推荐的工具？
   - **数学与代码：** rLLM 内置了针对数学（解析 `\boxed{}`）和代码任务的奖励函数，能够自动评估 Agent 的输出质量 [^17]。
3. **训练器 (Trainer)：** 基于收集的轨迹和奖励，利用 PPO 或 GRPO 等算法更新模型的权重。rLLM 底层集成了 verl (Volcano RL) 框架，支持分布式的训练任务 [^6]。

### 5.2 训练流程的技术细节

rLLM 的一个重要创新是能够**"在不修改代码的情况下训练"**（Training without Code Changes）。通过 Monkey-patching 或包装标准的 LLM 客户端（如 OpenAI Client），rLLM 可以在 Agent 运行时透明地截获数据流 [^16]。在具体实现中，为了支持 Ray 的序列化要求，`get_chat_client` 等对象的创建必须发生在函数内部，而非模块级别，这是一个典型的分布式系统工程细节 [^17]。

通过 rLLM，开发者可以将手工编写的 Prompt 策略（如"请一步步思考"）内化为模型的直觉。模型在数万次的试错中，学会了何时应该搜索，何时应该写代码，而不再需要外部 Prompt 的显式指令。这正是"模型原生"Agent 的生产方式。

---

## 6. 生产环境案例研究与基准评测

理论与架构最终需要落地为产品。通过对比目前的顶尖 Agent 产品，我们可以看到上述原则的实际应用。

### 6.1 编码 Agent CLI 三方对比

CLI（命令行界面）是编码 Agent 的主要战场。目前三大主流产品各具特色：

- **Kimi CLI：** 强调速度与成本效益。评测显示，在同等任务下，Kimi CLI 的响应速度（2 秒）显著快于 Claude Code（5 秒），且成本低 70% [^18]。它采用了 ACP 协议，能够集成到 Zed 等 IDE 中，体现了开源和标准化的路线 [^1]。虽然其"思考"深度不及 Claude，但在快速迭代场景下极具优势。
- **Claude Code：** 代表了"深度思考"的高端路线。它使用了极大的上下文窗口和复杂的预取策略，虽然成本高昂（$20/月起），但在处理大型代码库重构等需要深层上下文理解的任务时表现更佳 [^13]。
- **OpenAI Codex CLI：** 定位于 OpenAI 生态的原生工具链。依托 GPT 系列模型的推理能力，与 OpenAI API 深度绑定，适合已深度使用 OpenAI 平台的团队。在代码生成质量上有较强竞争力，但在自主性和工具调用灵活性上仍在快速迭代中 [^13]。

### 6.2 Manus AI 的经济学账本

Manus AI 的商业模式反映了高性能 Agent 的真实成本。

- **高昂的定价：** Manus 推出了 $39/月 和 $199/月 的订阅计划，这远高于传统的 SaaS 工具 [^19]。原因在于其 100:1 的输入输出比导致了巨大的推理成本。
- **虚拟机成本：** 每个 Session 独占一个云端 VM 的架构虽然保证了能力和安全，但也大幅推高了边际成本。
- **信用点机制：** 采用基于"积分"（Credits）的计费模式，而非简单的 Token 计费，这允许平台根据任务的复杂度（如是否启动了 VM，是否进行了大规模搜索）进行动态定价 [^20]。

### 6.3 METR 的时间跨度预测

METR 的研究为我们提供了一个衡量 Agent 进步的标尺：**任务完成时间跨度（Task-Completion Time Horizon）**。

- **现状：** 2025 年初，前沿 Agent（如 Claude 3.7 Sonnet）能可靠完成人类需耗时 1 小时左右的任务 [^1]。
- **趋势：** 这一能力每 7 个月翻一番。
- **预测：** 按照此趋势，到 2026 年，我们将看到能独立执行"周级别"甚至"月级别"任务的 Agent。这意味着它们不仅能写代码，还能独立维护项目、跟进 Issue、进行长期的市场监测。

---

## 7. 结论与未来展望

### 7.1 中间件的消亡与操作系统的诞生

随着"模型原生"能力的增强，复杂的中间件（如繁琐的 LangChain Chains）将逐渐消亡。Agent 的架构将简化为：**强模型 + 强环境**。

我们正在见证 **"Agentic OS"**（智能体操作系统）的诞生。这个 OS 的核心职能不再是调度线程，而是调度**上下文**（在 RAM、RAG 和文件系统之间移动数据）和管理**权限**（沙盒与工具掩码）。Kimi CLI、Manus 和 Claude Code 实际上都是这个雏形 OS 的不同发行版。

### 7.2 文本即未来的 API

尽管 MCP 试图建立结构化标准，但"Agent Skills"的成功表明，自然语言将是 Agent 时代的通用接口。未来的软件开发可能不再是编写 API 文档给人类看，而是编写 `SKILL.md` 给 AI 看。这种"面向 AI 的文档驱动开发"（Documentation-Driven Development for AI）将成为新的工程规范。

### 7.3 给工程师的建议

1. **拥抱 KV-Cache：** 将其视为系统设计的核心约束，就像传统开发中的数据库索引一样重要。
2. **构建深度挽具：** 不要只写 Prompt，要构建文件系统、规划器和沙盒环境。让 Agent 拥有"身体"和"外脑"。
3. **准备 RL 数据：** 开始收集 Agent 的执行轨迹（Trajectories）。在不久的将来，你的核心资产将不再是 Prompt 库，而是用于微调专属 Agent 的高质量轨迹数据。

自主智能体的工程化正从"提示词黑客"的作坊式作业，转变为一门严谨的系统工程学科。在这个新时代，成功的关键在于如何构建一个能让模型在其中长期生存、学习并进化的数字生态系统。

---

> 注：本报告严格基于所提供的研究资料撰写，旨在客观呈现当前技术前沿的真实面貌。

## 参考资料

[^1]: [自主 Agent / 上下文工程资料索引 · Issue #150 · ninehills/blog - GitHub](https://github.com/ninehills/blog/issues/150)
[^2]: [[論文評述] Beyond Pipelines: A Survey of the Paradigm Shift toward Model-Native Agentic AI](https://www.themoonlight.io/tw/review/beyond-pipelines-a-survey-of-the-paradigm-shift-toward-model-native-agentic-ai)
[^3]: [Building Advanced AI Agents with LangChain's DeepAgents: A Hands-On Guide](https://dev.to/samadhi_patil_294a4ff7fea/building-advanced-ai-agents-with-langchains-deepagents-a-hands-on-guide-1bk4)
[^5]: [Beyond Pipelines: A Survey of the Paradigm Shift toward Model-Native Agentic AI - arXiv](https://arxiv.org/html/2510.16720v1)
[^6]: [rLLM's Main Components - Read the Docs](https://rllm-project.readthedocs.io/en/latest/core-concepts/overview/)
[^7]: [Manus: Context Engineering Strategies for Production AI Agents - ZenML LLMOps Database](https://www.zenml.io/llmops-database/context-engineering-strategies-for-production-ai-agents)
[^8]: [Context Engineering for AI Agents: Lessons from Building Manus](https://manus.im/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus)
[^9]: [Agent-Skills-for-Context-Engineering - GitHub](https://github.com/muratcankoylan/Agent-Skills-for-Context-Engineering/blob/main/README.md)
[^10]: [Context Engineering for AI Agents: Key Lessons from Manus - DEV Community](https://dev.to/contextspace_/context-engineering-for-ai-agents-key-lessons-from-manus-1lgb)
[^13]: [GPT-5.1 Codex vs. Claude 4.5 Sonnet vs. Kimi K2 Thinking - Composio](https://composio.dev/blog/kimi-k2-thinking-vs-claude-4-5-sonnet-vs-gpt-5-codex-tested-the-best-models-for-agentic-coding)
[^14]: [Shortcomings of Model Context Protocol (MCP) Explained - CData Software](https://www.cdata.com/blog/navigating-the-hurdles-mcp-limitations)
[^16]: [rLLM Project - Home](https://rllm-project.com/)
[^17]: [Train Math Agent with rLLM SDK](https://rllm-project.readthedocs.io/en/latest/examples/sdk_math/)
[^18]: [Kimi CLI vs OpenAI CLI vs Claude Speed Cost 2025 - Skywork AI](https://skywork.ai/blog/agent/kimi-cli-vs-openai-cli-vs-claude-speed-cost-2025/)
[^19]: [China's AI Agent Manus Launches Subscription Plans - Asiainsight](https://asiainsight.ecinnovations.com/reports/chinas-ai-agent-manus-launches-subscription-plans-what-you-need-to-know-about-the-39-and-199-tiers/)
[^20]: [AI agent platform Manus launches team subscription plan - Tech in Asia](https://www.techinasia.com/news/ai-agent-platform-manus-launches-team-subscription-plan)

