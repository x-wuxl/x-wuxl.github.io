---
layout: post
title: "Antigravity 必装的 AI 额度监控神器，让你告别 Gemini/Claude 配额焦虑"
date: 2026-01-01 15:30:00 +0800
description: "深度介绍 Antigravity Cockpit 这款扩展插件，它能实时监控 Google Antigravity AI 模型配额，支持 Webview 仪表盘、状态栏置顶、配额分组、阈值告警等功能，彻底解决开发者使用 Gemini AI 时的配额焦虑问题。"
author: "像素信标"
categories: [实用教程]
tags: [Antigravity, VSCode, Google Gemini, Claude, AI工具, 开发效率, 配额监控, 效率工具]
keywords: "Antigravity Cockpit, VSCode 扩展, Antigravity插件, Gemini 配额监控, Claude 配额监控, AI 配额管理, Google Antigravity, VS Code 插件"
---

如果你是 Google Antigravity（也就是 Gemini AI 编程助手）的用户，一定深有体会：**最怕的不是 AI 不够智能，而是正写到关键代码时，突然发现配额耗尽了**。

今天介绍的 **Antigravity Cockpit** 就是专门解决这个痛点的 VS Code 扩展——它能让你实时掌握 AI 配额状态，再也不用担心"写到一半没额度"的尴尬。

<img src="/assets/images/antigravity_quota_monitor.png" width="60%" alt="Antigravity 配额监控示例">

## 这款插件能解决什么问题？

使用 Gemini AI 编程时，最常见的焦虑来源于：

1. **不知道还剩多少配额**——只能等用完才发现
2. **不清楚什么时候重置**——盲目等待，效率低下
3. **多模型切换时更迷茫**——哪个模型还能用？完全靠猜

Antigravity Cockpit 提供了一个**一站式监控仪表盘**，让这些信息一目了然。

## 核心功能详解

### 🎛️ 两种显示模式，满足不同需求

**Webview 仪表盘**（推荐）

这是默认模式，提供了一个精美的可视化界面：

- **卡片视图 / 列表视图**：两种布局随心切换
- **分组模式**：按配额池自动归类模型，共享配额的模型一目了然
- **拖拽排序**：自定义模型显示顺序
- **实时进度条**：可视化剩余配额

**QuickPick 模式**

使用 VS Code 原生 API，适合以下场景：

- Webview 加载有问题的环境
- 偏好纯键盘操作的用户
- 需要快速瞄一眼配额状态

### 📊 状态栏监控：一眼掌握关键信息

状态栏是这个插件的精华所在，支持 **6 种显示格式**：

| 格式 | 示例 |
|------|------|
| 仅图标 | `🚀` |
| 仅状态点 | `🟢` / `🟡` / `🔴` |
| 仅百分比 | `95%` |
| 状态点 + 百分比 | `🟢 95%` |
| 名称 + 百分比 | `Sonnet: 95%` |
| 完整显示 | `🟢 Sonnet: 95%` |

**特别实用的功能**：

- **多模型置顶**：可以同时监控多个常用模型
- **自动监控**：没指定模型时，自动显示剩余配额最低的那个

### 🔔 阈值告警：提前预警，从容应对

你可以设定两个阈值：

- **警告阈值**（默认 30%）：配额低于这个值时变黄
- **危险阈值**（默认 10%）：配额低于这个值时变红

当配额触碰阈值时，插件会发送通知提醒你，让你有充足时间调整使用策略。

### ⏰ 配额详情：不只是百分比

每个模型 / 分组都会显示：

- **剩余配额百分比**
- **倒计时**：距离重置还有多久（如 `4h 40m`）
- **重置时间**：具体几点重置（如 `15:16`）
- **可视化进度条**

### 🏷️ 分组功能：管理多模型更轻松

- **按配额池分组**：共享配额的模型自动归类
- **自定义分组名称**：给分组起个好记的名字
- **分组排序**：拖拽调整显示顺序
- **分组置顶**：把最重要的分组固定到状态栏

### 🌐 多语言支持

插件跟随 VS Code 的语言设置，**支持 14 种语言**：

🇺🇸 English · 🇨🇳 简体中文 · 繁體中文 · 🇯🇵 日本語 · 🇩🇪 Deutsch · 🇪🇸 Español · 🇫🇷 Français · 🇮🇹 Italiano · 🇰🇷 한국어 · 🇧🇷 Português · 🇷🇺 Русский · 🇹🇷 Türkçe · 🇵🇱 Polski · 🇨🇿 Čeština

## 如何安装和使用

### 安装方式

**方式一：Open VSX 市场**

1. `Ctrl/Cmd + Shift + X` 打开扩展面板
2. 搜索 `Antigravity Cockpit`
3. 点击安装

**方式二：VSIX 文件安装**

```bash
code --install-extension antigravity-cockpit-x.y.z.vsix
```

### 基本使用

1. **打开仪表盘**：
   - 点击状态栏图标
   - 或按 `Ctrl/Cmd + Shift + Q`
   - 或在命令面板运行 `Antigravity Cockpit: Open Dashboard`

2. **刷新配额**：点击刷新按钮或 `Ctrl/Cmd + Shift + R`

3. **遇到问题**：
   - 显示 "Systems Offline" 时点击 **Retry Connection**
   - 点击 **Open Logs** 查看调试日志

## 常用配置项

| 配置项 | 默认值 | 说明 |
|------|--------|------|
| `agCockpit.displayMode` | `webview` | 显示模式：`webview` / `quickpick` |
| `agCockpit.viewMode` | `card` | 视图模式：`card` / `list` |
| `agCockpit.refreshInterval` | `120` | 刷新间隔（秒，10-3600） |
| `agCockpit.statusBarFormat` | `standard` | 状态栏格式 |
| `agCockpit.groupingEnabled` | `true` | 启用分组模式 |
| `agCockpit.warningThreshold` | `30` | 警告阈值（%） |
| `agCockpit.criticalThreshold` | `10` | 危险阈值（%） |
| `agCockpit.notificationEnabled` | `true` | 启用通知 |

## 写在最后

用 AI 写代码已经成为越来越多开发者的日常，但**配额管理**这个环节往往被忽视——直到用完的那一刻才追悔莫及。

**Antigravity Cockpit** 这款插件虽然功能聚焦，但解决的是一个实实在在的痛点。如果你是 Google Antigravity / Gemini 的重度用户，强烈建议安装试试。

**相关链接**：

- 📦 [Open VSX 市场](https://open-vsx.org/extension/jlcodes/antigravity-cockpit)
- 🔗 [GitHub 仓库](https://github.com/jlcodes99/vscode-antigravity-cockpit)
- 💬 [问题反馈](https://github.com/jlcodes99/vscode-antigravity-cockpit/issues)
