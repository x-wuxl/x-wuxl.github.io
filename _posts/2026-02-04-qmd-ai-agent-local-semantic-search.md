---
title: "让 OpenClaw 节省 10 倍 Token 消耗：qmd 本地语义搜索配置指南"
date: 2026-02-04
description: "使用 qmd 本地语义搜索引擎，让 OpenClaw 节省 10 倍 token 消耗。零 API 成本，精准度 95% 以上，10 分钟即可配置完成，支持 MCP 集成。"
keywords:
  - qmd
  - OpenClaw
  - AI Agent
  - MCP
  - 节省 token
  - Claude token 限制
  - 本地语义搜索
  - AI 记忆管理
  - AI 编程工具
  - Cursor 插件
  - 向量搜索
  - AI 工具推荐
categories:
  - AI 工具
tags:
  - qmd
  - AI Agent
  - MCP
  - 语义搜索
  - 本地部署
---

如果你在用 OpenClaw，应该已经感受到了 token 消耗的速度——尤其 Claude 用户，没谈几轮下来就 hit limit 了。而且，很多时候 agent 塞了一堆无关信息进 context，不仅费钱，还影响精准度。

**有没有办法让 agent "精准回忆"，同时完全零成本？**

有。**qmd** —— 本地运行，免费永久，精准度 95% 以上。

## 什么是 qmd？

qmd 是 Shopify 创始人 Tobi 开发的本地语义搜索引擎，基于 Rust 构建，专为 AI Agent 设计。

**核心特性：**

- **混合搜索**：BM25 全文 + 向量语义 + LLM 重排序
- **零 API 成本**：完全本地运行，无需联网
- **MCP 集成**：与 AI Agent 无缝对接

## 3 步配置，10 分钟搞定

### 第 1 步：安装 qmd

首先安装 Bun，然后执行：

```bash
bun install -g github:tobi/qmd
```

> 首次运行会自动下载模型（Embedding 和 Reranker），下载完成后即可完全离线运行。

### 第 2 步：创建记忆库 + 生成 Embeddings

进入 OpenClaw 工作目录，索引你的 memory 文件夹：

```bash
# 创建记忆库
qmd collection add memory --name daily-logs --mask "**/*.md"

# 生成 embeddings
qmd embed
```

索引速度极快，本地运行不联网。

### 第 3 步：测试搜索

```bash
# 混合搜索（最精准）
qmd query "关键词"

# 纯语义搜索
qmd vsearch "关键词"
```

## 进阶：MCP 集成

让 AI Agent 直接调用 qmd，在 `mcporter.json` 里添加配置：

```json
{
  "mcpServers": {
    "qmd": {
      "command": "/Users/你的用户名/.bun/bin/qmd",
      "args": ["mcp"]
    }
  }
}
```

配置完成后，agent 会主动从历史 log 中找最相关的段落，跨文件精准回忆，不再需要手动提醒。

## 实际效果

| 场景 | 优化前 | 优化后 |
|------|--------|--------|
| 用户偏好回忆 | 整个 2000 token 的 MEMORY.md | 仅返回相关的 200 token |
| 跨文件检索 | 手动查找或全量加载 | 自动精准匹配，准确率极高 |

## 总结

如果你在用 OpenClaw 觉得 token 烧得心疼，qmd 是目前最实用的解决方案——**免费、本地、精准**，赶紧折腾起来！
