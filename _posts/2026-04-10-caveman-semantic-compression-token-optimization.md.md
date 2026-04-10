---
layout: post
title: "token 消耗减少 75%？读完 caveman 源码，我发现它本质是语义压缩"
date: 2026-04-10
last_modified_at: 2026-04-10
categories:
  - AI
  - LLM
  - Prompt Engineering
tags:
  - caveman
  - LLM
  - token优化
  - 提示词工程
  - 语义压缩
  - 输出压缩
  - AI工具
  - 大模型
keywords:
  - caveman
  - caveman 源码分析
  - token 消耗减少 75%
  - token 优化
  - LLM 输出压缩
  - 语义压缩
  - prompt based semantic compression
  - 提示词工程
  - system prompt
  - skill prompt
  - AI 输出 token 优化
  - benchmark
  - evals
  - Claude Code
  - Codex
description: "深入拆解 caveman v1.3.0 源码与评测链路，分析它如何通过 system prompt 与 skill prompt 约束输出风格，减少低信息密度表达，从而显著降低 LLM 输出 token 消耗。文章重点解释 caveman 的核心机制、benchmark 与 evals 的测量方法、输出压缩与输入压缩的区别，以及这种“语义压缩”方案的边界与代价。"
author: "wuxl"
excerpt: "caveman 真正压缩的不是 token 编码，而是语言表达。读完源码和评测脚本后，我发现它本质上是一套可复用、可分发、可量化的输出风格约束系统：通过删掉寒暄、冠词、hedging 和低信息密度胶水词，在尽量保留技术术语与代码块的前提下，把 LLM 回答压得更短。"
toc: true
toc_sticky: true
---

现在的 AI 订阅，大都是 5 小时额度限制、周额度限制

在 AI 有使用额度的前提下，输出本身就是一种资源，**输出 token 既影响成本，也影响响应速度，还影响最终可读性**。很多时候，模型不是不会答，而是太爱铺垫、太爱解释、太爱说那些“技术上没错但信息密度很低”的话。

`caveman` 这个项目有意思的地方就在这。它没有去改模型，也没有做什么花哨的 token 编码优化，核心做法其实是用一套很强的输出风格约束，把模型回答压得更短。它对外宣称能减少大约 75% 的输出 token，同时 benchmark 表里给出的平均值是 **65%**   ，也就是说，这个仓库自己其实已经告诉你：**这是一个“效果因任务而异”的风格约束方案，不是一个稳定恒定的 75% 压缩器。**

如果你是把它当成“提示词工程的小玩具”，那就低估它了。这个仓库真正值得看的点，是它把“少说废话”这件事，做成了：

- 可安装的 skill
- 可同步到多个宿主环境的分发结构
- 可量化评测的实验脚本
- 还能和另一个输入侧压缩工具分开建模

所以这篇文章重点放在：**它到底靠什么减少 token，这套机制的边界在哪，仓库里的代码又是怎么证明这一点的。**

**caveman 不是传统意义上的压缩算法。**

它没有：

- 压缩 token 序列
- 对输出做后处理截断
- 用缓存复用历史片段
- 改 tokenizer
- 改模型推理逻辑

它做的事情其实更接近：

> **prompt-based semantic compression**

翻译过来就是：**在模型生成之前，就用 system prompt / skill prompt 强约束输出风格，让模型从源头上少生成那些低信息密度的自然语言。**

这个思路的核心不在“压缩 token”，而在“压缩表达”。

技术术语保留，代码块不动，错误信息尽量原样保留；但冠词、寒暄、解释性连接词、礼貌话术、冗余句式，大量删掉。这样做的结果，并不是“同一串话被编码得更短”，而是模型从一开始就不去说那些话。

这也是为什么它的效果看起来很猛：因为自然语言里真正费 token 的，往往不是专业名词，反倒是那些“读起来很顺、但信息密度不高”的胶水词。

---

## 核心机制：它本质上是在做“输出风格约束”

### 1. 核心逻辑在哪

这个项目最关键的“算法本体”，其实不在 Python 代码里，而在 `skills/caveman/SKILL.md:11-63`。

这份文件基本定义了 caveman 模式下模型该怎么说话。

比如它在规则里明确要求：

- 去掉冠词、 filler、pleasantries、hedging（`skills/caveman/SKILL.md:15-18`）
- 允许用 fragment，不要求完整句（`skills/caveman/SKILL.md:17-18`）
- 使用更短的同义词（`skills/caveman/SKILL.md:17-18`）
- 技术术语必须保持准确（`skills/caveman/SKILL.md:17-18`）
- 代码块内容不改（`skills/caveman/SKILL.md:17-18`）
- 输出模式尽量收敛到 `[thing] [action] [reason] [next step]`（`skills/caveman/SKILL.md:19`）

它还提供了多档压缩强度：`lite`、`full`、`ultra`，以及几档文言文模式（`skills/caveman/SKILL.md:24-48`）。这说明它不是单一模板，而是一组可调的语言压缩策略。

### 2. 为什么这种方式能减少 token

原因其实不复杂。

大模型默认输出，尤其在“乐于助人”的 system prompt 下，很容易带上这些东西：

- “Sure! I'd be happy to help...”
- “The reason this happens is likely because...”
- “I’d recommend considering...”
- “It might be worth checking whether...”

这些话的问题不是错，而是**信息密度太低**。

`caveman` 的做法是直接在 system/skill 层禁止这类表达，把回答压成更像工程师速记的风格：

- 先说结论
- 少铺垫
- 少修饰
- 少客套
- 用短词
- 技术名词保持原样

从 token 角度看，这种做法本质上是在删掉大量“语言摩擦成本”。

### 3. 它不是简单粗暴的截断

如果只是把输出截断，通常会伤害信息完整性。但 `caveman` 在 prompt 里强调的是：

- **technical substance stay**（`skills/caveman/SKILL.md:11`）
- **technical terms exact**（`skills/caveman/SKILL.md:17`）

也就是说，它的目标不是变短本身，而是**提高单位 token 的信息密度**。

这点很关键，因为它决定了这个项目的哲学不是“少生成”，而是“少说废话，保留技术信息”。

### 4. 它还专门给高风险场景留了后门

这个设计里最成熟的一点，是作者没有把“越短越好”做成绝对规则。

在 `Auto-Clarity` 里，skill 明确写了几类场景要暂时退出 caveman 风格，比如：

- 安全警告
- 不可逆操作确认
- 多步骤且容易误读的说明
- 用户已经困惑的场景

对应说明在 `skills/caveman/SKILL.md:50-59`。

这其实是在承认一个现实：**压缩表达会牺牲一部分可读性和冗余安全垫。**

所以 caveman 不是“永远最短”，而是“默认尽量短，但在高风险地方恢复清晰表达”。

你可以把它理解成一种有底线的语言压缩方式。

---

## 从调用链看源码：真正的执行流长什么样

如果把这个项目当成一个系统来看，它的主链路大概是这样：

1. 用户触发 caveman skill
2. 宿主（Claude Code / Codex 插件等）把 `SKILL.md` 作为 system prompt 的一部分注入
3. 模型按这套风格约束生成回答
4. benchmark / eval 脚本再去统计输出 token 变化

这条链里最值得看的有三段：

- `skills/caveman/SKILL.md`
- `evals/llm_run.py`
- `evals/measure.py`

另外还有一条单独的对外 benchmark 链：

- `benchmarks/run.py`

下面分开看。

---

## 源码探秘一：`SKILL.md` 才是这个项目真正的“算法”

很多人看到这种项目，会下意识去找 `src/`、`lib/`、`compress()` 之类的函数。但这个仓库里，真正决定效果的不是一个算法函数，而是一份 prompt 规范。

`CONTRIBUTING.md:5-16` 写得很清楚：**只需要编辑 `skills/caveman/SKILL.md`，其他副本不要手改。**

而 `.github/workflows/sync-skill.yml:29-43` 又进一步说明，CI 会把这份源文件同步到多个位置：

- `caveman/SKILL.md`
- `plugins/caveman/skills/caveman/SKILL.md`
- `.cursor/skills/caveman/SKILL.md`
- 以及打包生成 `caveman.skill`

这说明仓库的设计是：

- **行为定义**：`skills/caveman/SKILL.md`
- **分发复制**：GitHub Actions 同步
- **宿主适配**：不同目录给不同环境使用

也就是说，作者把“输出压缩逻辑”放在最可移植的层——prompt 文本层，而不是某个宿主绑定的 runtime 代码里。

### 最关键的一段规则

下面这段几乎可以看成整个项目的核心：

```markdown
## Rules

Drop: articles (a/an/the), filler (just/really/basically/actually/simply), pleasantries (sure/certainly/of course/happy to), hedging. Fragments OK. Short synonyms (big not extensive, fix not "implement a solution for"). Technical terms exact. Code blocks unchanged. Errors quoted exact.

Pattern: `[thing] [action] [reason]. [next step].`
```

来源：`skills/caveman/SKILL.md:15-19`

这段规则背后的思路可以拆成三层：

#### 第一层：删低信息密度成分

被点名删除的几类东西，几乎都是自然语言里最容易膨胀 token 的部分：

- 冠词
- 填充词
- 礼貌话术
- 试探性表达

这些词让语言更自然，但不一定让技术信息更完整。

#### 第二层：允许不完整句

`Fragments OK` 这个点特别重要。

因为如果还要求模型始终输出完整英语句子，那它很容易重新长回来。允许 fragment，等于直接给模型开放了一种更节省 token 的表达空间。

#### 第三层：保留技术锚点

如果只是“尽量短”，模型很可能连术语也一起压坏。但这里明确要求：

- 技术术语精确
- 代码块不改
- 错误信息原样引用

这相当于在压缩和准确性之间人为画了一条边界。

所以更准确地说，`caveman` 做的并非无差别压缩，而是：

> **删除自然语言外壳，尽量保留技术骨架。**

---

## 源码探秘二：75% 这种说法，仓库里到底是怎么量出来的

这个项目的另一个关键点，是它不只提供了 skill，还提供了评测方法。

这里有两套测量逻辑：

1. **面向 README 展示的 benchmark**：`benchmarks/run.py`
2. **更强调实验设计严谨性的 eval harness**：`evals/llm_run.py` + `evals/measure.py`

这两套都很重要，但定位不一样。

### 路径 A：`benchmarks/run.py` —— 对外展示用的 benchmark

在 `benchmarks/run.py:33-35` 里，脚本先定义了两个 system prompt 起点：

```python
NORMAL_SYSTEM = "You are a helpful assistant."
BENCHMARK_START = "<!-- BENCHMARK-TABLE-START -->"
BENCHMARK_END = "<!-- BENCHMARK-TABLE-END -->"
```

后面 `load_caveman_system()` 直接把 `skills/caveman/SKILL.md` 读进来（`benchmarks/run.py:44-46`），再在 `run_benchmarks()` 里分别跑两种模式：

```python
for mode, system in [("normal", NORMAL_SYSTEM), ("caveman", caveman_system)]:
```

来源：`benchmarks/run.py:93-101`

也就是说，这套 benchmark 的对比方式是：

- **normal**：普通 helpful assistant
- **caveman**：整份 caveman skill prompt

然后 `compute_stats()` 取每个 prompt 多次试验的中位数输出 token（`benchmarks/run.py:108-145`），再算节省比例，最后 `update_readme()` 把结果回写到 README（`benchmarks/run.py:205-220`）。

所以 README 里的大表，本质上更像一个**面向展示的 output token 对比实验**。

### 路径 B：`evals/llm_run.py` —— 更诚实的三臂评测

如果只拿 “普通助手” 对比 “caveman skill”，其实有个问题：

> 你测出来的差值，里面混着两件事：
> 1. “要求简洁”本身带来的收益
> 2. caveman 这套特殊风格额外带来的收益

作者显然意识到了这个问题，所以在 `evals/README.md:7-18` 和 `evals/llm_run.py:1-16` 里，明确把实验设计改成了三臂：

- `__baseline__`：没有 system prompt
- `__terse__`：只有 `Answer concisely.`
- `<skill>`：`Answer concisely.` + `SKILL.md`

代码里也写得很直接：

```python
TERSE_PREFIX = "Answer concisely."
```

来源：`evals/llm_run.py:40`

真正构造 skill 组 system prompt 的地方在这里：

```python
skill_md = (SKILLS / skill / "SKILL.md").read_text()
system = f"{TERSE_PREFIX}\n\n{skill_md}"
```

来源：`evals/llm_run.py:93-97`

这个设计的意思是：

- 先控制住“简短回答”这个公共变量
- 再测 caveman skill 相比普通 concise instruction 的额外收益

从评测方法上说，这比 README 那个二元对比更干净，也更接近“这份 skill 本身贡献了多少”。

### `measure.py` 怎么算 token

`evals/measure.py` 负责从 `results.json` 里读出不同 arm 的输出，然后用 `tiktoken` 计数（`evals/measure.py:23-30`）。

关键计算在这里：

```python
savings = [
    1 - (s / t) if t else 0.0 for s, t in zip(skill_tokens, terse_tokens)
]
```

来源：`evals/measure.py:85-89`

它测的不是 “skill 相对 baseline 节省了多少”，而是：

> **skill 相对 terse control 又额外节省了多少**

这也是 `evals/README.md:15-19` 一直在强调的“honest delta”。

所以如果你问：**仓库里最严谨的评测口径是什么？**

答案其实不是 README 那个“普通 Claude vs caveman”的对比，而是 `evals/` 里的三臂实验。

---

## 一个很容易被忽略的点：这个仓库其实在分开处理“输出压缩”和“输入压缩”

如果只看主 README，容易把所有“省 token”的能力混在一起。

但仓库实际上把两件事分得很开：

### 1. `caveman`
负责让模型**说得更短**。

这是输出侧优化，核心文件是 `skills/caveman/SKILL.md`。

### 2. `compress`
负责把 `CLAUDE.md`、todo、preferences 这类自然语言上下文文件压短，让模型**读得更少**。

这部分对应：

- `compress/SKILL.md:10-110`
- `caveman-compress/README.md:13-28`

这套工具会把自然语言 memory 文件改写成更紧凑的形式，同时保留代码块、URL、路径、命令、技术术语等（`compress/SKILL.md:47-64`）。

所以更准确地说：

- `caveman` 解决 **output token**
- `compress` 解决 **input/context token**

这点很值得单独强调，因为很多人会把仓库里的“token savings”理解成一种统一魔法。实际上，它是两个不同方向的优化。

---

## 技术边界与代价：少 75% 不是没有代价

任何“显著减少输出”的方案，都会带来 trade-off。`caveman` 也不例外。

### 1. 牺牲的是语言自然度，不是技术术语

从 `SKILL.md` 的规则就能看出来，它尽量保留的是技术骨架，牺牲掉的是语言外壳。

这意味着回答会更像：

- 工程师短评
- review comment
- debugging note
- CLI 助手的直接结论

而不太像：

- 面向普通用户的解释型文案
- 有完整铺垫和情绪照顾的客服回复
- 适合教学场景的慢节奏讲解

### 2. 它省的是输出 token，不是全部 token

这一点仓库自己也写得很明白：

> Caveman only affects output tokens — thinking/reasoning tokens are untouched.

来源：`README.md:244-247`

所以如果把它理解成“模型总成本一定下降 75%”，那就过头了。

尤其在 `evals/README.md:75-80` 里也专门提醒：**skill 本身会增加输入 token**。这意味着真实的经济收益，不应该只看输出侧数字。

换句话说，`caveman` 更像是在用更多的前置约束，换更短的最终回答。

### 3. 75% 不是固定常数

仓库内部就能看到这个问题。

- README 文案说“~75%”（`README.md:28`）
- README benchmark 平均值是 **65%**（`README.md:227-241`）
- 单条任务区间是 **22%–87%**（`README.md:241`）

这其实已经说明：

- 对话类型不同，节省比例差异会很大
- 越是本来就冗长、解释性强的问题，压缩空间越大
- 越是本来就短、结构紧的回答，额外收益越有限

所以“75%”更像一个 marketing headline，而不是严格保证值。

### 4. 评测里的 token 计数也是近似值

`evals/measure.py` 使用的是 `tiktoken o200k_base`（`evals/measure.py:23-30`），而 `evals/README.md:79-80` 明确说明这只是 **Claude tokenizer 的近似**，不是官方精确值。

这意味着：

- 比例比较是有参考价值的
- 绝对 token 数不要过度解读

### 5. 过度压缩会增加误解风险

这也是为什么 `Auto-Clarity` 要存在。

因为在这些场景里，语言冗余不是浪费，而是安全缓冲：

- 删除数据
- 改权限
- 危险 shell 命令
- 多步骤操作说明
- 用户已经没跟上上下文时

这些地方如果也强行“caveman 化”，可能会让指令更短，但更容易误解。

所以项目在设计上其实是承认：

> **最省 token，不等于最安全，也不等于最清晰。**

---

## 优缺点与适用场景

### 优点

#### 1. 实现极轻

核心逻辑基本就是一份 skill prompt，没有重 runtime，没有模型微调，没有复杂后处理链路。

#### 2. 可移植

从仓库结构看，它能同时服务 Claude Code、Codex plugin 等不同宿主。说明这套方法天然适合跨平台迁移。

#### 3. 可评测

这不是“我感觉变短了”。仓库里有明确脚本去跑对比和记录结果：

- `benchmarks/run.py`
- `evals/llm_run.py`
- `evals/measure.py`

#### 4. 对工程场景很贴合

很多开发场景其实不需要完整作文式回答，只需要：

- 问题在哪
- 为什么
- 怎么改

`caveman` 恰好把回答压到这个粒度。

### 缺点

#### 1. 可读性会下降

尤其是对新手、跨语言用户、需要上下文铺垫的场景，fragment 风格并不友好。

#### 2. 不适合所有任务

越需要完整论证、教学解释、情绪安抚、业务沟通的场景，它越容易显得太硬。

#### 3. 真实成本收益没法只看 output token

因为 skill prompt 本身也要占输入 token，仓库也明确承认了这一点。

#### 4. 它优化的是表达，不是能力

它不会让模型更会推理，也不会自动让答案更正确。它只是把模型已有能力用更短的形式表达出来。

### 适合的场景

- IDE / CLI 编程助手
- Code review 评论
- Commit message / PR comment
- 高频开发问答
- 团队内部已经共享上下文，不需要大段背景解释的场景

### 不适合的场景

- 面向终端用户的产品文案
- 教学型回答
- 高风险运维确认
- 涉及很多步骤且依赖上下文连续性的说明
- 需要语气柔和、可读性强的沟通场景

如果只从产品表面看，`caveman` 像是在玩一种搞笑说话风格。

但从实现角度看，它背后其实有一个很实用的工程启发：

> **很多 token 优化，不一定要动模型层；只要输出约束足够明确，接口层也能拿到很可观的收益。**

这个项目做对了几件事：

1. 把风格约束写成独立 skill，而不是散落在每次对话里
2. 把 skill 做成多个宿主可复用的分发单元
3. 给出评测脚本，而不是只展示几个 before/after 样例
4. 明确承认 trade-off，而不是硬吹“又短又全又没代价”

这比单纯喊一句“让模型简洁点”要成熟得多。

---

回到最开始的问题：**caveman 是怎么减少 LLM 输出 token 的？**

答案其实很直接：

它压缩的不是 token，本质上是语言表达。

它把一套明确的语言约束写进 `SKILL.md`，逼模型少说冠词、少说寒暄、少说 hedging、少说解释性废话，同时保留技术术语、代码块和关键结论。然后再通过 benchmark 和 eval 脚本去验证，回答到底短了多少。

*所以，*`caveman`  的本质，与其说是“模型优化器”，不如说是一个**可复用、可分发、可评测的输出风格控制层**。

如果你做的是开发者工具、代码助手、review 机器人，这套思路很值得借鉴。

如果你做的是面向普通用户的长解释系统，那它的收益和副作用就要重新算。

但不管怎么说，这个仓库至少证明了一件事：

**很多时候，想让模型更省 token，未必要让它更聪明，只要让它别那么啰嗦。**

---

参考源码版本：v1.3.0
