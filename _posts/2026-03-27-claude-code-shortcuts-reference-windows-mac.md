---
layout: post
title: "Claude Code 快捷键速查表：Windows / Mac 对照与高频命令整理"
date: 2026-03-27
last_modified_at: 2026-03-27
categories: [AI, 开发工具, 效率工具]
tags: [Claude Code, 快捷键, Claude Code 快捷键, Claude Code 命令, Windows, Mac, MCP, CLI, AI 编程, 开发效率]
keywords: "Claude Code 快捷键, Claude Code 快捷键速查表, Claude Code Windows 快捷键, Claude Code Mac 快捷键, Claude Code 命令大全, Claude Code slash commands, Claude Code MCP, Claude Code CLI 参数, Claude Code 使用技巧, Claude Code 键位对照"
description: "这是一份适合中文用户检索和收藏的 Claude Code 速查文章，系统整理 Windows 与 Mac 键位差异、常用快捷键、Slash Commands、MCP、CLI 参数、配置项和工作流技巧。"
author: wuxl
excerpt: "想快速查 Claude Code 的快捷键、Slash Commands、MCP 和 CLI 参数？这篇文章把 Windows / Mac 差异和高频操作整理到一处，适合随用随查。"
toc: true
toc_sticky: true
---

如果你平时已经在用 Claude Code，那你大概率会遇到一个很烦的小问题：**很多操作明明见过，但真要用的时候，突然想不起来键位或者命令名。**

如果你是第一次接触 Claude Code，可以把这篇当成入门索引；如果你已经在日常开发里用了，这篇更适合当成一个长期收藏的备忘单。

## 先看平台差异：Windows / Mac 键位怎么对应？

### Windows → Mac 对照

| Windows 显示 | Mac 显示 | 说明 |
|---|---|---|
| Alt | ⌥ Option | 页面会自动按平台切换显示 |
| Shift | ⇧ | 页面会自动按平台切换显示 |
| Ctrl | Ctrl | 不变 |
| Esc | Esc | 不变 |
| Enter | Enter | 不变 |
| Tab | Tab | 不变 |
| 方向键 | 方向键 | 不变 |

### 涉及平台差异的快捷键

| Windows | Mac | 功能 |
|---|---|---|
| Alt + P | ⌥ + P | 切换模型 |
| Alt + T | ⌥ + T | 切换思考模式 |
| Alt + O | ⌥ + O | 切换快速模式 |
| Shift + Tab | ⇧ + Tab | 循环切换权限模式 |

---

## 键盘快捷键总表

### 常用控制

| 快捷键（Win） | 快捷键（Mac） | 说明 |
|---|---|---|
| Ctrl + C | Ctrl + C | 取消输入/生成 |
| Ctrl + D | Ctrl + D | 退出会话 |
| Ctrl + L | Ctrl + L | 清屏 |
| Ctrl + O | Ctrl + O | 切换详细输出 |
| Ctrl + R | Ctrl + R | 反向搜索历史 |
| Ctrl + G | Ctrl + G | 在编辑器中打开提示 |
| Ctrl + X, Ctrl + E | Ctrl + X, Ctrl + E | 在编辑器中打开（别名） |
| Ctrl + B | Ctrl + B | 后台运行任务 |
| Ctrl + T | Ctrl + T | 切换任务列表 |
| Ctrl + V | Ctrl + V | 粘贴图片 |
| Ctrl + X, Ctrl + K | Ctrl + X, Ctrl + K | 终止后台代理 |
| Esc, Esc | Esc, Esc | 回退 / 撤销 |

### 模式切换

| 快捷键（Win） | 快捷键（Mac） | 说明 |
|---|---|---|
| Shift + Tab | ⇧ + Tab | 循环切换权限模式 |
| Alt + P | ⌥ + P | 切换模型 |
| Alt + T | ⌥ + T | 切换思考模式 |
| Alt + O | ⌥ + O | 切换快速模式 |

### 输入

| 快捷键（Win） | 快捷键（Mac） | 说明 |
|---|---|---|
| \ + Enter | \ + Enter | 换行（快捷方式） |
| Ctrl + J | Ctrl + J | 换行（控制序列） |

### 前缀

| 键 | 说明 |
|---|---|
| / | 斜杠命令 |
| ! | 直接执行 bash |
| @ | 引用文件 + 自动补全 |

### 会话选择器

| 键 | 说明 |
|---|---|
| ↑ / ↓ | 导航 |
| ← / → | 展开/折叠 |
| P | 预览 |
| R | 重命名 |
| / | 搜索 |
| A | 所有项目 |
| B | 当前分支 |


在实际使用里，最值得优先记住的其实就几类：

- `Ctrl + C`：中断当前输入或生成
- `Ctrl + L`：清屏，长会话里特别实用
- `Ctrl + R`：快速翻历史
- `Ctrl + B`：把任务放到后台
- `Ctrl + T`：切任务列表
- `Shift + Tab`：切权限模式
- `Alt/⌥ + T`：开关思考模式

如果你不想一上来背太多，先把这些高频项记住，已经够用了。

---

## MCP 服务器相关操作

### 添加服务器

| 项目 | 说明 |
|---|---|
| --transport http | 远程 HTTP（推荐） |
| --transport stdio | 本地进程 |
| --transport sse | 远程 SSE |

### 作用域

| 位置 | 说明 |
|---|---|
| Local `settings.local.json` | 仅限本人 |
| Project `.mcp.json` | 共享/版本控制 |
| User `~/.claude.json` | 全局 |

### 管理

| 命令 / 项目 | 说明 |
|---|---|
| /mcp | 交互式界面 |
| `claude mcp list` | 列出所有服务器 |
| `claude mcp serve` | CC 作为 MCP 服务器 |
| Elicitation 服务器 | 在任务中请求输入 |

---

## ⚡ 斜杠命令

### 会话

| 命令 | 说明 |
|---|---|
| /clear | 清除对话 |
| /compact [focus] | 压缩上下文 |
| /resume | 恢复/切换会话 |
| /rename [name] | 命名当前会话 |
| /branch [name] | 分支对话（/fork 别名） |
| /cost | Token 用量统计 |
| /context | 可视化上下文（网格） |
| /diff | 交互式差异查看器 |
| /copy [N] | 复制最近（或第 N 条）回复 |
| /rewind | 回退对话 / 代码检查点 |
| /export | 导出对话 |

### 配置

| 命令 | 说明 |
|---|---|
| /config | 打开设置 |
| /model [model] | 切换模型（←→ 调整力度） |
| /fast [on\|off] | 切换快速模式 |
| /vim | 切换 vim 模式 |
| /theme | 更换颜色主题 |
| /permissions | 查看/更新权限 |
| /effort [level] | 设置力度（low/med/high/max/auto） |
| /color [color] | 设置提示栏颜色 |
| /keybindings | 自定义键盘快捷键 |
| /terminal-setup | 配置终端快捷键 |

### 工具

| 命令 | 说明 |
|---|---|
| /init | 创建 `CLAUDE.md` |
| /memory | 编辑 `CLAUDE.md` 文件 |
| /mcp | 管理 MCP 服务器 |
| /hooks | 管理钩子 |
| /skills | 列出可用技能 |
| /agents | 管理代理 |
| /chrome | Chrome 集成 |
| /reload-plugins | 热重载插件 |
| /add-dir <path> | 添加工作目录 |

### 特殊

| 命令 | 说明 |
|---|---|
| /btw <question> | 旁问（不消耗上下文） |
| /plan [desc] | 规划模式（+ 自动启动） |
| /loop [interval] | 定时循环任务 |
| /voice | 按住说话语音（20 种语言） |
| /doctor | 诊断安装问题 |
| /pr-comments [PR] | 获取 GitHub PR 评论 |
| /stats | 使用统计与偏好 |
| /insights | 会话分析报告 |
| /desktop | 在桌面应用中继续 |
| /remote-control | 桥接到 `claude.ai/code`（`/rc`） |
| /usage | 套餐限额与速率状态 |
| /schedule | 云端定时任务 |
| /security-review | 变更安全审查 |
| /help | 显示帮助 + 命令 |
| /feedback | 提交反馈（别名：`/bug`） |
| /release-notes | 查看完整更新日志 |
| /stickers | 订购贴纸！ |

---

## 📁 记忆与文件

### CLAUDE.md 位置

| 位置 | 说明 |
|---|---|
| `./CLAUDE.md` | 项目级（团队共享） |
| `~/.claude/CLAUDE.md` | 个人级（所有项目） |
| `/etc/claude-code/` | 托管级（组织范围） |

### 规则与导入

| 项目 | 说明 |
|---|---|
| `.claude/rules/*.md` | 项目规则 |
| `~/.claude/rules/*.md` | 用户规则 |
| `paths:` frontmatter | 按路径生效的规则 |
| `@path/to/file` | 在 `CLAUDE.md` 中导入 |

### 自动记忆

| 项目 | 说明 |
|---|---|
| `~/.claude/projects/<proj>/memory/` | 自动记忆目录 |
| `MEMORY.md` + 主题文件 | 自动加载 |

---

## 🧠 工作流与技巧

### 规划模式

| 项目 | 说明 |
|---|---|
| Shift + Tab | 普通 → 自动接受 → 规划 |
| --permission-mode plan | 以规划模式启动 |

### 思考与力度

| 项目 | 说明 |
|---|---|
| Alt + T / ⌥ + T | 开启/关闭思考 |
| `"ultrathink"` | 本轮最大力度 |
| Ctrl + O | 查看思考过程（详细模式） |
| /effort | `low / med / high` |

### Git Worktrees

| 项目 | 说明 |
|---|---|
| --worktree name | 每个功能独立分支 |
| isolation: worktree | 代理在独立 worktree 中运行 |
| sparsePaths | 仅检出所需目录 |
| /batch | 自动创建 worktrees |

### 语音模式

| 项目 | 说明 |
|---|---|
| /voice | 启用按住说话 |
| Space（按住） | 录音，松开发送 |
| 20 种语言 | EN、ES、FR、DE、CZ、PL… |

### 上下文管理

| 项目 | 说明 |
|---|---|
| /context | 用量 + 优化建议 |
| /compact [focus] | 带焦点压缩 |
| Auto-compact | 约 95% 容量时触发 |

---

## ⚙️ 配置与环境

### 配置文件位置

| 位置 | 说明 |
|---|---|
| `settings.json` | 主设置 |
| `settings.local.json` | 仅限本地 |
| `~/.claude.json` | OAuth、MCP、状态 |
| `.mcp.json` | 项目 MCP 服务器 |

### 关键设置

| 配置项 | 说明 |
|---|---|
| modelOverrides | 模型选择器映射 → 自定义 ID |
| autoMemoryDirectory | 自定义记忆目录 |
| worktree.sparsePaths | 稀疏检出目录 |

### 关键环境变量

| 变量名 | 说明 |
|---|---|
| ANTHROPIC_API_KEY | API Key |
| ANTHROPIC_MODEL | 模型 |
| CLAUDE_CODE_EFFORT_LEVEL | `low/med/high` |
| MAX_THINKING_TOKENS | `0 = 关闭` |
| ANTHROPIC_CUSTOM_MODEL_OPTION | 自定义 `/model` 条目 |
| CLAUDE_CODE_PLUGIN_SEED_DIR | 多个插件种子目录 |
| CLAUDECODE | 检测 CC shell（=1） |
| IS_DEMO | 演示模式（隐藏邮箱/组织） |

---

## 🔧 Skills与代理

### 内置Skills

| 项目 | 说明 |
|---|---|
| /simplify | 代码审查（3 个并行代理） |
| /batch | 大规模并行变更（5-30 个 worktrees） |
| /debug [desc] | 从调试日志排查问题 |
| /loop [interval] | 定时循环任务 |
| /claude-api | 加载 API + SDK 参考 |

### 自定义Skills位置

| 位置 | 说明 |
|---|---|
| `.claude/skills/<name>/` | 项目技能 |
| `~/.claude/skills/<name>/` | 个人技能 |

### 技能 Frontmatter

| 项目 | 说明 |
|---|---|
| description | 自动调用触发条件 |
| allowed-tools | 跳过权限提示 |
| model | 覆盖技能模型 |
| effort | 覆盖力度级别 |
| context: fork | 在子代理中运行 |
| $ARGUMENTS | 用户输入占位符 |
| ${CLAUDE_SKILL_DIR} | 技能自身目录 |
| !`cmd` | 动态上下文注入 |

### 内置代理

| 代理 | 说明 |
|---|---|
| Explore | 快速只读（Haiku） |
| Plan | 规划模式调研 |
| General | 全工具，复杂任务 |
| Bash | 终端独立上下文 |

### 代理 Frontmatter

| 项目 | 说明 |
|---|---|
| permissionMode | `default / acceptEdits / plan / dontAsk / bypass` |
| isolation: worktree | 在 git worktree 中运行 |
| memory: user\|project | 持久化记忆 |
| background: true | 后台任务 |
| maxTurns | 限制代理轮次 |
| SendMessage | 恢复代理（替代 `resume`） |

---

## 🖥️ CLI 与参数

### 核心命令

| 命令 | 说明 |
|---|---|
| `claude` | 交互式 |
| `claude "q"` | 带提示启动 |
| `claude -p "q"` | 无头模式 |
| `claude -c` | 继续上次 |
| `claude -r "n"` | 恢复会话 |
| `claude update` | 更新 |

### 关键参数

| 参数 | 说明 |
|---|---|
| `--model` | 设置模型 |
| `-w` | Git worktree |
| `-n` / `--name` | 会话名称 |
| `--add-dir` | 添加目录 |
| `--agent` | 使用代理 |
| `--allowedTools` | 预授权 |
| `--output-format` | `json/stream` |
| `--json-schema` | 结构化 |
| `--max-turns` | 限制轮次 |
| `--max-budget-usd` | 费用上限 |
| `--console` | 通过 Anthropic Console 认证 |
| `--verbose` | 详细输出 |
| `--bare` | 最小无头模式（无钩子/LSP） |
| `--channels` | 权限中继 / MCP 推送 |
| `--remote` | Web 会话 |
| `--effort` | `low/med/high/max` |
| `--permission-mode` | `plan/default/...` |
| `--dangerously-skip-permissions` | 跳过所有提示 ⚠️ |

