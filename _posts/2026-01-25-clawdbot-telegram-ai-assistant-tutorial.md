---
layout: post
title: "Clawdbot 安装教程：打造 24 小时在线的 Telegram AI 助手"
date: 2026-01-25 14:00:00 +0800
categories: [AI工具, 实用教程]
tags: [Clawdbot, Telegram, AI助手, Claude, GPT, 自动化, 开源项目, VPS]
description: "Clawdbot 安装部署完整教程：在 VPS 上搭建 24 小时在线的 Telegram AI 助手，支持 Claude、GPT 等大模型，可处理邮件、日程、调研等自动化任务。"
excerpt: "想要一个真正能帮你干活的 AI 助手？Clawdbot 支持通过 Telegram 交互，24 小时在线处理邮件、日程、调研等任务。本文手把手教你从零部署。"
author: 像素信标
keywords: 
  - Clawdbot
  - Telegram AI
  - AI助手
  - 自动化助手
  - Claude API
  - GPT API
  - VPS部署
  - 开源AI
seo:
  type: BlogPosting
  canonical: "https://xwuxl.com/2026/01/25/clawdbot-telegram-ai-assistant-tutorial"
  publisher: "像素信标"
og:
  type: article
  locale: zh_CN
twitter:
  card: summary_large_image
---

## 前言

如果你希望有一个 **24 小时在线**、能真正帮你干活的 AI 助手（而不是只会聊天的那种），**Clawdbot** 值得一试。

它可以自动处理邮箱、查询资料、安排日程、写内容、跟进事项，并通过 **Telegram** 或 **WhatsApp** 与你实时交互。

---

## 什么是 Clawdbot？

[Clawdbot](https://clawd.bot/) 是一个开源 AI 助手程序，部署在云服务器上运行：

| 特性 | 说明 |
|------|------|
| **24 小时在线** | 部署在 VPS，随时响应 |
| **多平台交互** | 支持 Telegram / WhatsApp |
| **工具集成** | 可调用邮箱、日历、搜索、文件、GitHub 等 |
| **任务执行** | 能执行连续任务，而不只是回答问题 |
| **多模型支持** | Claude、GPT、OpenRouter、本地模型等 |

### 适合的使用场景

- 📧 邮件整理与回复提醒
- 🔍 公司 / 产品调研
- 📅 日程与提醒管理
- ✍️ 内容草稿撰写
- ⚙️ 简单运维或自动化任务

---

## 准备工作

在开始之前，你需要准备：

1. **一台 VPS**（推荐 Ubuntu 系统）
2. **大模型 API Key**（Claude 或 OpenAI）
3. **Telegram 账号**

---

## 安装步骤

### 第一步：安装 Clawdbot

SSH 登录到你的 VPS，运行以下命令：

```bash
curl -fsSL https://clawd.bot/install.sh | bash
```

等待安装完成即可。

---

### 第二步：运行配置向导

#### 方式一：使用 Claude 模型

按顺序选择：

1. **Quick Start**
2. **Anthropic**
3. **Token paste setup**

系统会提示你在**本地电脑终端**执行一条命令来获取 token：

```
1. 打开你电脑的终端（非 VPS）
2. 粘贴命令并运行
3. 复制生成的 token
4. 粘贴回服务器
```

然后选择模型和通讯方式：

| 配置项 | 推荐选择 |
|--------|----------|
| **模型** | Opus 4.5 |
| **通讯方式** | Telegram bot |

#### 方式二：使用 OpenAI 模型

在服务器中设置环境变量：

```bash
export OPENAI_API_KEY=你的API_Key
```

然后运行配置命令：

```bash
clawdbot onboard --auth-choice openai-api-key
```

模型设置为 `openai/gpt-4o` 即可。

---

### 第三步：创建 Telegram 机器人

1. 打开 Telegram，搜索 `@BotFather`
2. 发送 `/newbot` 命令
3. 按提示完成机器人创建
4. **复制保存 Bot Token**，粘贴回安装向导

接下来获取你的用户 ID：

1. 搜索 `@userinfobot`
2. 发送任意消息，获取你的 **User ID**
3. 粘贴回安装向导

---

### 第四步：初始化 AI 身份

在 Telegram 中，机器人会询问你几个问题：

- 你的称呼
- 它的名字
- 它的职责定位
- 你的时区

回答完成后，Clawdbot 就可以使用了！🎉

---

## 快速测试

在 Telegram 中向机器人发送以下消息，测试功能：

### 🔍 公司调研

```
调研一下 XXX 公司，用 3 点总结
```

### ⏰ 设置提醒

```
明天早上 9 点提醒我做 XXX
```

---

## 扩展能力（可选）

Clawdbot 支持逐步添加更多工具，提升能力：

| 工具 | 用途 |
|------|------|
| **Brave Search** | 网页搜索 |
| **Gmail** | 邮件处理 |
| **Google Calendar** | 日程管理 |
| **Google Drive** | 文件访问 |
| **GitHub** | 代码仓库操作 |

**配置方式**：获取对应服务的 API Key 后，直接在聊天中告诉机器人即可完成配置。
