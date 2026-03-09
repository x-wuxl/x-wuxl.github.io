---
layout: post
title: "OpenClaw 2026.3.2 默认关闭 Agent 工具权限：原因分析与修复方法"
date: 2026-03-09
last_modified_at: 2026-03-09
categories: [AI, 技术教程]
tags: [OpenClaw, Agent, 工具权限, exec, web_fetch, messaging, 配置修复]
keywords: "OpenClaw, Agent 工具权限, messaging profile, full profile, exec 执行权限, web_fetch, 工具配置, OpenClaw 2026.3.2 更新"
description: "OpenClaw 2026.3.2 版本将 Agent 工具权限默认设置为 messaging 模式，导致 exec、web_fetch 等核心功能被禁用。本文解析问题原因，并提供一键修复配置方案。"
author: wuxl
excerpt: "OpenClaw 2026.3.2 更新后，Agent 默认只能对话和发消息，无法调用 exec、web_fetch 等工具。本文为你详解问题原因，并给出完整的配置修复步骤。"
toc: true
toc_sticky: true
---

## 问题概述

OpenClaw **2026.3.2** 版本对 Agent 的工具权限策略进行了调整，将默认配置从**全功能模式**降级为 **messaging（仅消息）模式**。

这意味着更新后，Agent 只能进行对话和发送消息，以下核心功能将 **全部不可用**：

- ❌ `exec` — 命令执行
- ❌ `web_fetch` — 网页抓取
- ❌ 文件读写操作
- ❌ 其他所有工具调用

无论你让 Agent 查资料、跑脚本还是执行实际操作，它都无法调用任何工具来完成任务。

---

## 问题根因

更新后的默认配置如下：

```json
{
    "tools": {
        "profile": "messaging"
    }
}
```

`messaging` 模式将 Agent 的能力严格限制在**纯文本对话**范围内，所有工具调用权限被一刀切关闭。

---

## 修复方法

将 `profile` 从 `messaging` 修改为 `full`，并开启会话的完整可见性：

```json
{
    "tools": {
        "profile": "full",
        "sessions": {
            "visibility": "all"
        }
    }
}
```

### 配置项说明

| 字段 | 值 | 作用 |
|:---|:---|:---|
| `tools.profile` | `"full"` | 启用全部工具权限（exec、web_fetch、文件读写等） |
| `tools.sessions.visibility` | `"all"` | 允许 Agent 访问所有会话上下文 |

---

## 修改步骤

### 1. 定位配置文件

OpenClaw 的配置文件在各操作系统下的路径如下：

| 操作系统 | 配置文件路径 |
|:---|:---|
| **Windows** | `C:\Users\<用户名>\.openclaw\openclaw.json` |
| **macOS** | `~/.openclaw/openclaw.json` |
| **Linux** | `~/.openclaw/openclaw.json` |

> **提示**：macOS 和 Linux 下 `.openclaw` 是隐藏目录，终端中可用 `ls -a ~` 查看，或在文件管理器中开启「显示隐藏文件」。

### 2. 修改配置

1. 用文本编辑器打开 `openclaw.json`
2. 找到 `tools` 配置节点，修改成如下内容：

    ```json
    {
        "tools": {
            "profile": "full",
            "sessions": {
                "visibility": "all"
            }
        }
    }
    ```

3. 保存文件并重启即可

### 3. 验证修复

修改完成后，建议让 Agent 执行一个简单的工具调用来确认权限已恢复，例如：

- 让 Agent 运行一条 `exec` 命令
- 让 Agent 使用 `web_fetch` 抓取一个网页

---

## 总结

OpenClaw 2026.3.2 的这次权限默认值调整，大概率出于安全收敛的考虑，但对日常使用 Agent 工具能力的用户来说影响较大。只需将 `profile` 改回 `full` 并配置 `sessions.visibility` 即可恢复完整功能。
