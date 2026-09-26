---
layout: post
title: "通过反代 Excel 解决 ChatGPT 降智问题"
date: 2026-09-26
last_modified_at: 2026-09-26
categories:
  - AI开发
  - 逆向工程
tags:
  - ChatGPT
  - Codex
  - OpenAI Excel
  - 反代 Excel
  - API
  - 反向代理
  - AI降智
keywords: "ChatGPT降智, Codex降智, ChatGPT API, Basispoints API, OpenAI Excel插件, 降智修复, ChatGPT反代, Excel反代"
description: "详细介绍如何利用 OpenAI 官方 Excel 插件的底层 Basispoints API 绕过 ChatGPT Web 端的风控与模型降智限制，恢复满血模型调用的操作指南。"
author: kanonouta
excerpt: "发现 ChatGPT/Codex 账号被降智？本文教你利用官方 Excel 插件的底层反代接口（Basispoints API），避开 Web 端风控检测，轻松恢复满血模型调用。"
toc: true
toc_sticky: true
---

#### 1. 核心原理

通过直接调用 OpenAI 官方 **Excel 插件**的底层 API 接口（`basispoints`），避开网页端严格的风控与检测，直接获取未降智的模型响应。

#### 2. 前置准备

在浏览器登录 ChatGPT 网页版，通过开发者工具（F12 -> Application/Storage 或 Network）获取以下两个关键参数：

- **`access_token`**：账号的 JWT Token。
- **`chatgpt_account_id`**：解析 JWT 中的 `https://api.openai.com/auth` 或从请求头提取出的 Account ID。

#### 3. API 请求配置

- **请求方式**：`POST`

- **接口地址**：`https://bps.openai.com/basispoints/api/responses)`

- **Headers（请求头）**：

  JSON

  ```
  {
    "Authorization": "Bearer <你的access_token>",
    "chatgpt-account-id": "<你的chatgpt_account_id>",
    "x-openai-account-id": "<你的chatgpt_account_id>",
    "x-basispoints-auth-mode": "chatgpt",
    "Content-Type": "application/json"
  }
  ```

  *(注：格式需完全一致)*

- **Body（请求体）**：传入标准的 ChatGPT 对话 Payload。

#### 注意事项

- 该接口不支持深度思考模型的 `effort: max` 挡位。