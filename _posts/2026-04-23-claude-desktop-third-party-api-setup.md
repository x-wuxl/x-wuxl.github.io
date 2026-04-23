---
layout: post
title: "Claude Desktop 怎样设置使用第三方 API 中转站"
date: 2026-04-23
last_modified_at: 2026-04-23
categories:
  - AI工具
  - 教程指南
tags:
  - Claude
  - Claude Desktop
  - API
  - 中转服务
  - 开发者模式
keywords: "Claude Desktop, 第三方 API, 中转 API, Claude 客户端设置, Configure third-party inference, Claude 开发者模式教程, Claude Base URL 配置"
description: "本文为您提供了一份简明详细的教程，指导您如何在 Claude Desktop 桌面客户端中开启开发者模式，并配置第三方中转 API 的 Base URL 和 API Key，实现本地顺畅调用。"
author: "wuxl"
excerpt: "想要在 Claude Desktop 桌面客户端中使用第三方中转 API 吗？只需简单的 4 步开启开发者模式，即可轻松配置自定义的 Base URL 和 API Key，摆脱网络和官方限制。"
toc: true
toc_sticky: true
---

想要在 Claude Desktop 桌面版中使用第三方或中转站提供的 API 吗？很多用户由于各种原因需要使用第三方接口来运行 Claude，其实官方客户端本身就隐藏了自定义 API 的入口。只需简单的几个步骤开启开发者模式，即可轻松实现。

以下是详细的设置步骤：

### 1. 打开客户端（勿登录）
首先，启动 Claude Desktop 客户端。**注意：此时请务必保持未登录状态！** 如果已经登录，请先退出账号。

### 2. 启用开发者模式
在系统顶部菜单栏依次点击：
`Help` → `Troubleshooting` → `Enable developer mode`

### 3. 进入第三方配置
开启开发者模式后，菜单栏会新增一个 `Developer` 选项。依次点击：
`Developer` → `Configure third-party inference`

### 4. 填写 API 信息
在弹出的配置窗口中，填入由您的第三方中转站提供的相关信息：
*   **Base URL**：填入中转服务的接口地址（注意是否需要以 `/v1` 结尾，具体视服务商要求而定）
*   **API Key**：填入您的令牌密钥

填写完后，点下面的 **`Apply locally`** 即可应用配置。
