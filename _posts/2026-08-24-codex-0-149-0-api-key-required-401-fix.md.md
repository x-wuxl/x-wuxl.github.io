---
layout: post
title: "Codex 0.149.0 升级后 API_KEY_REQUIRED / 401 Unauthorized 解决办法"
date: 2026-08-24
last_modified_at: 2026-08-24
categories:
  - 开发工具
  - 故障排查
tags:
  - Codex
  - OpenAI Codex
  - API_KEY_REQUIRED
  - 401 Unauthorized
  - config.toml
  - API 配置
keywords: "Codex 0.149.0, API_KEY_REQUIRED, 401 Unauthorized, Missing API key, requires_openai_auth, auth.json, config.toml, Codex 升级, Codex 401 解决办法"
description: "Codex 升级至 0.149.0 后出现 API_KEY_REQUIRED 或 401 Unauthorized？本文说明新版鉴权行为变化，并提供 macOS 与 Windows 下修改 config.toml 中 requires_openai_auth 配置的解决方法。"
author: ""
excerpt: "Codex 0.149.0 升级后，若出现 API_KEY_REQUIRED 或 401 Unauthorized，可将 config.toml 中的 requires_openai_auth 设置为 true，以恢复正常鉴权与使用。"
toc: true
toc_sticky: true
---

# Codex 0.149.0 升级后 API_KEY_REQUIRED / 401 Unauthorized 解决办法

升级 Codex 至 0.149.0 后，如果遇到 `API_KEY_REQUIRED` 或 `401 Unauthorized` 错误，通常与新版的鉴权行为变化有关。

新版不再允许自定义 Provider 在 `requires_openai_auth = false` 时，自动继承 `auth.json` 中的鉴权信息。因此，需要手动调整 Codex 配置文件中的鉴权开关。

## 解决方案：修改 config.toml

请打开 Codex 配置文件：

- **macOS：** `~/.codex/config.toml`
- **Windows：** `%USERPROFILE%\.codex\config.toml`

找到以下配置项：

```toml
requires_openai_auth = false
```

将其修改为：

```toml
requires_openai_auth = true
```

保存配置文件后，重新启动 Codex 或重新执行相关命令即可。

如果配置文件中没有这行配置，则在对应的 `model_providers.xxx` 下新添加这一配置项即可。

## 原因说明

在 Codex 0.149.0 中，自定义 Provider 的认证逻辑发生调整。当 `requires_openai_auth` 设置为 `false` 时，Codex 不会再自动使用 `auth.json` 内的鉴权信息，进而可能导致 API 密钥缺失或服务端返回 401 未授权错误。

将该配置改为 `true` 后，Codex 会启用 OpenAI 鉴权流程，通常即可恢复正常使用。

## 注意事项

- 修改前建议备份 `config.toml`，避免误改其他 Provider 配置。
- 若修改后仍然报错，请进一步确认 `auth.json` 中的登录或 API 鉴权信息是否有效。
- 如果配置文件中存在多个 Provider，请确认修改的是当前实际使用的 Provider 配置。
