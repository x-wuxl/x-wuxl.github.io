---
layout: post
title: "Claude Agent SDK 完全开发指南：从零开始构建智能代理"
date: 2026-01-10 14:00:00 +0800
categories: [AI开发, Claude]
tags: [Claude, AI Agent, SDK, TypeScript, 智能代理]
description: 深入解析 Claude Agent SDK 的核心原理与实战应用，从基础概念到生产级代码审查代理的完整开发流程，帮助开发者快速掌握 AI 代理开发的最佳实践。
keywords: Claude Agent SDK, AI代理开发, Claude Code, 智能代理, TypeScript, 代码审查, MCP协议, 子代理, 结构化输出
image: /assets/images/claude-agent-sdk.png
seo:
  type: TechArticle
  datePublished: 2026-01-10
  dateModified: 2026-01-10
---

# Claude Agent SDK 完全开发指南：从零开始构建智能代理

如果你用过 Claude Code，一定对 AI 代理的能力印象深刻：它能够自主读取文件、执行命令、编辑代码，像一位经验丰富的工程师一样拆解任务、逐步完成目标。

这不仅仅是一个代码助手，而是真正能够接管问题、像资深工程师一样思考和解决问题的智能系统。

[Claude Agent SDK](https://platform.claude.com/docs/en/agent-sdk/overview) 将这套核心引擎完全开放给开发者，让你可以针对任何场景快速构建自己的 AI 代理。

它本质上是 Claude Code 背后的基础设施，以 SDK 形式对外提供。你将获得完整的代理循环、内置工具集、上下文管理机制——这些原本需要从零实现的复杂功能，现在开箱即用。

本文将通过从零构建一个代码审查代理，带你深入理解 SDK 的核心工作机制。完成后，你将拥有一个能够自主分析代码库、发现 Bug 和安全问题、并返回结构化反馈的智能代理。

更重要的是，你将掌握 SDK 的底层原理，从而能够构建任何你实际需要的智能代理系统。

## 我们要构建什么

我们的代码审查代理将具备以下能力：

1. 自动分析代码库中的 Bug 和安全隐患
2. 自主读取文件并搜索代码
3. 提供结构化、可执行的反馈建议
4. 实时跟踪工作进度

## 技术栈

- **运行环境** - [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code)
- **核心 SDK** - [@anthropic-ai/claude-agent-sdk](https://www.npmjs.com/package/@anthropic-ai/claude-agent-sdk)
- **开发语言** - TypeScript
- **AI 模型** - Claude Opus 4.5

## SDK 的核心价值

如果你用原生 API 构建过代理系统，一定熟悉这个模式：调用模型 → 检查是否需要工具 → 执行工具 → 将结果反馈给模型 → 循环直到完成。在构建复杂系统时，这个过程会变得相当繁琐。

SDK 为你接管了整个循环：

```typescript
// 不使用 SDK：你需要手动管理循环
let response = await client.messages.create({...});
while (response.stop_reason === "tool_use") {
  const result = yourToolExecutor(response.tool_use);
  response = await client.messages.create({ tool_result: result, ... });
}

// 使用 SDK：Claude 自动管理循环
for await (const message of query({ prompt: "修复 auth.py 中的 Bug" })) {
  console.log(message); // Claude 会自动读取文件、发现问题、编辑代码
}
```

SDK 还提供了开箱即用的工具集：

- **Read** - 读取工作目录中的任何文件
- **Write** - 创建新文件
- **Edit** - 精确编辑现有文件
- **Bash** - 执行终端命令
- **Glob** - 按模式查找文件
- **Grep** - 使用正则表达式搜索文件内容
- **WebSearch** - 网络搜索
- **WebFetch** - 获取并解析网页

这些工具无需你自己实现。

## 前置要求

1. 安装 Node.js 18+
2. 获取 Anthropic API Key（[点击获取](https://console.anthropic.com/)）

## 快速开始

**步骤 1：安装 Claude Code CLI**

Agent SDK 使用 Claude Code 作为运行环境：

```bash
npm install -g @anthropic-ai/claude-code
```

安装后，在终端运行 `claude` 并按提示完成身份验证。

**步骤 2：创建项目**

```bash
mkdir code-review-agent && cd code-review-agent
npm init -y
npm install @anthropic-ai/claude-agent-sdk
npm install -D typescript @types/node tsx
```

**步骤 3：设置 API Key**

```bash
export ANTHROPIC_API_KEY=your-api-key
```

## 第一个代理

创建 `agent.ts`：

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

async function main() {
  for await (const message of query({
    prompt: "列出当前目录下的所有文件",
    options: {
      model: "opus",
      allowedTools: ["Glob", "Read"],
      maxTurns: 250
    }
  })) {
    if (message.type === "assistant") {
      for (const block of message.message.content) {
        if ("text" in block) {
          console.log(block.text);
        }
      }
    }
    
    if (message.type === "result") {
      console.log("\n完成:", message.subtype);
    }
  }
}

main();
```

运行：

```bash
npx tsx agent.ts
```

Claude 会使用 **Glob** 工具列出文件，并告诉你发现了什么。

## 理解消息流

`query()` 函数返回一个异步生成器，会在 Claude 工作时流式返回消息。核心消息类型如下：

```typescript
for await (const message of query({ prompt: "..." })) {
  switch (message.type) {
    case "system":
      // 会话初始化信息
      if (message.subtype === "init") {
        console.log("会话 ID:", message.session_id);
        console.log("可用工具:", message.tools);
      }
      break;
      
    case "assistant":
      // Claude 的响应和工具调用
      for (const block of message.message.content) {
        if ("text" in block) {
          console.log("Claude:", block.text);
        } else if ("name" in block) {
          console.log("工具调用:", block.name);
        }
      }
      break;
      
    case "result":
      // 最终结果
      console.log("状态:", message.subtype); // "success" 或错误类型
      console.log("成本:", message.total_cost_usd);
      break;
  }
}
```

## 构建代码审查代理

现在来构建一个真正实用的系统。创建 `review-agent.ts`：

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

async function reviewCode(directory: string) {
  console.log(`\n🔍 开始代码审查：${directory}\n`);
  
  for await (const message of query({
    prompt: `审查 ${directory} 目录中的代码，重点关注：
            1. Bug 和潜在崩溃风险
            2. 安全漏洞  
            3. 性能问题
            4. 代码质量改进建议

            请指出具体的文件名和行号。`,
    options: {
      model: "opus",
      allowedTools: ["Read", "Glob", "Grep"],
      permissionMode: "bypassPermissions", // 自动批准读取操作
      maxTurns: 250
    }
  })) {
    // 实时显示 Claude 的分析过程
    if (message.type === "assistant") {
      for (const block of message.message.content) {
        if ("text" in block) {
          console.log(block.text);
        } else if ("name" in block) {
          console.log(`\n📁 正在使用 ${block.name}...`);
        }
      }
    }
    
    // 显示完成状态
    if (message.type === "result") {
      if (message.subtype === "success") {
        console.log(`\n✅ 审查完成！成本：$${message.total_cost_usd.toFixed(4)}`);
      } else {
        console.log(`\n❌ 审查失败：${message.subtype}`);
      }
    }
  }
}

// 审查当前目录
reviewCode(".");
```

**测试代理**

创建一个包含故意问题的文件 `example.ts`：

```typescript
function processUsers(users: any) {
  for (let i = 0; i <= users.length; i++) { // 数组越界错误
    console.log(users[i].name.toUpperCase()); // 缺少 null 检查
  }
}

function connectToDb(password: string) {
  const connectionString = `postgres://admin:${password}@localhost/db`;
  console.log("使用连接字符串:", connectionString); // 记录敏感数据
}

async function fetchData(url) { // 缺少类型注解
  const response = await fetch(url);
  return response.json(); // 缺少错误处理
}
```

运行审查：

```bash
npx tsx review-agent.ts
```

Claude 会识别出这些 Bug、安全问题，并提供修复建议。

## 添加结构化输出

对于程序化使用，你需要结构化数据。SDK 支持 JSON Schema 输出：

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

const reviewSchema = {
  type: "object",
  properties: {
    issues: {
      type: "array",
      items: {
        type: "object",
        properties: {
          severity: { type: "string", enum: ["low", "medium", "high", "critical"] },
          category: { type: "string", enum: ["bug", "security", "performance", "style"] },
          file: { type: "string" },
          line: { type: "number" },
          description: { type: "string" },
          suggestion: { type: "string" }
        },
        required: ["severity", "category", "file", "description"]
      }
    },
    summary: { type: "string" },
    overallScore: { type: "number" }
  },
  required: ["issues", "summary", "overallScore"]
};

async function reviewCodeStructured(directory: string) {
  for await (const message of query({
    prompt: `审查 ${directory} 的代码，识别所有问题。`,
    options: {
      model: "opus",
      allowedTools: ["Read", "Glob", "Grep"],
      permissionMode: "bypassPermissions",
      maxTurns: 250,
      outputFormat: {
        type: "json_schema",
        schema: reviewSchema
      }
    }
  })) {
    if (message.type === "result" && message.subtype === "success") {
      const review = message.structured_output as {
        issues: Array<{
          severity: string;
          category: string;
          file: string;
          line?: number;
          description: string;
          suggestion?: string;
        }>;
        summary: string;
        overallScore: number;
      };
      
      console.log(`\n📊 代码审查结果\n`);
      console.log(`评分：${review.overallScore}/100`);
      console.log(`总结：${review.summary}\n`);
      
      for (const issue of review.issues) {
        const icon = issue.severity === "critical" ? "🔴" :
                     issue.severity === "high" ? "🟠" :
                     issue.severity === "medium" ? "🟡" : "🟢";
        console.log(`${icon} [${issue.category.toUpperCase()}] ${issue.file}${issue.line ? `:${issue.line}` : ""}`);
        console.log(`   ${issue.description}`);
        if (issue.suggestion) {
          console.log(`   💡 ${issue.suggestion}`);
        }
        console.log();
      }
    }
  }
}

reviewCodeStructured(".");
```

## 权限控制

默认情况下，SDK 在执行工具前会请求批准。你可以自定义这一行为：

**权限模式**

```typescript
options: {
  // 标准模式 - 提示批准
  permissionMode: "default",
  
  // 自动批准文件编辑
  permissionMode: "acceptEdits",
  
  // 无提示（谨慎使用）
  permissionMode: "bypassPermissions"
}
```

**自定义权限处理器**

对于细粒度控制，使用 `canUseTool`：

```typescript
options: {
  canUseTool: async (toolName, input) => {
    // 允许所有读取操作
    if (["Read", "Glob", "Grep"].includes(toolName)) {
      return { behavior: "allow", updatedInput: input };
    }
    
    // 阻止写入某些文件
    if (toolName === "Write" && input.file_path?.includes(".env")) {
      return { behavior: "deny", message: "不能修改 .env 文件" };
    }
    
    // 允许其他所有操作
    return { behavior: "allow", updatedInput: input };
  }
}
```

## 创建子代理

对于复杂任务，可以创建专用的子代理：

```typescript
import { query, AgentDefinition } from "@anthropic-ai/claude-agent-sdk";

async function comprehensiveReview(directory: string) {
  for await (const message of query({
    prompt: `对 ${directory} 进行全面代码审查。
使用 security-reviewer 处理安全问题，使用 test-analyzer 分析测试覆盖率。`,
    options: {
      model: "opus",
      allowedTools: ["Read", "Glob", "Grep", "Task"], // Task 启用子代理
      permissionMode: "bypassPermissions",
      maxTurns: 250,
      agents: {
        "security-reviewer": {
          description: "安全漏洞检测专家",
          prompt: `你是安全专家。重点关注：
                  - SQL 注入、XSS、CSRF 漏洞
                  - 暴露的凭证和密钥
                  - 不安全的数据处理
                  - 认证/授权问题`,
          tools: ["Read", "Grep", "Glob"],
          model: "sonnet"
        } as AgentDefinition,
        
        "test-analyzer": {
          description: "测试覆盖率和质量分析专家",
          prompt: `你是测试专家。分析：
                  - 测试覆盖率缺口
                  - 缺失的边界情况
                  - 测试质量和可靠性
                  - 额外测试建议`,
          tools: ["Read", "Grep", "Glob"],
          model: "haiku" // 使用更快的模型进行简单分析
        } as AgentDefinition
      }
    }
  })) {
    if (message.type === "assistant") {
      for (const block of message.message.content) {
        if ("text" in block) {
          console.log(block.text);
        } else if ("name" in block && block.name === "Task") {
          console.log(`\n🤖 委派给：${(block.input as any).subagent_type}`);
        }
      }
    }
  }
}

comprehensiveReview(".");
```

## 会话管理

对于多轮对话，可以捕获并恢复会话：

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

async function interactiveReview() {
  let sessionId: string | undefined;
  
  // 初始审查
  for await (const message of query({
    prompt: "审查这个代码库并识别前 3 个问题",
    options: {
      model: "opus",
      allowedTools: ["Read", "Glob", "Grep"],
      permissionMode: "bypassPermissions",
      maxTurns: 250
    }
  })) {
    if (message.type === "system" && message.subtype === "init") {
      sessionId = message.session_id;
    }
    // ... 处理消息
  }
  
  // 使用相同会话进行后续提问
  if (sessionId) {
    for await (const message of query({
      prompt: "现在告诉我如何修复最严重的问题",
      options: {
        resume: sessionId, // 继续对话
        allowedTools: ["Read", "Glob", "Grep"],
        maxTurns: 250
      }
    })) {
      // Claude 会记住之前的上下文
    }
  }
}
```

## 使用钩子

钩子让你可以拦截和自定义代理行为：

```typescript
import { query, HookCallback, PreToolUseHookInput } from "@anthropic-ai/claude-agent-sdk";

const auditLogger: HookCallback = async (input, toolUseId, { signal }) => {
  if (input.hook_event_name === "PreToolUse") {
    const preInput = input as PreToolUseHookInput;
    console.log(`[审计] ${new Date().toISOString()} - ${preInput.tool_name}`);
  }
  return {}; // 允许操作
};

const blockDangerousCommands: HookCallback = async (input, toolUseId, { signal }) => {
  if (input.hook_event_name === "PreToolUse") {
    const preInput = input as PreToolUseHookInput;
    if (preInput.tool_name === "Bash") {
      const command = (preInput.tool_input as any).command || "";
      if (command.includes("rm -rf") || command.includes("sudo")) {
        return {
          hookSpecificOutput: {
            hookEventName: "PreToolUse",
            permissionDecision: "deny",
            permissionDecisionReason: "危险命令已被阻止"
          }
        };
      }
    }
  }
  return {};
};

for await (const message of query({
  prompt: "清理临时文件",
  options: {
    model: "opus",
    allowedTools: ["Bash", "Glob"],
    maxTurns: 250,
    hooks: {
      PreToolUse: [
        { hooks: [auditLogger] },
        { matcher: "Bash", hooks: [blockDangerousCommands] }
      ]
    }
  }
})) {
  // ...
}
```

## 通过 MCP 添加自定义工具

使用模型上下文协议（MCP）扩展 Claude 的工具能力：

```typescript
import { query, tool, createSdkMcpServer } from "@anthropic-ai/claude-agent-sdk";
import { z } from "zod";

// 创建自定义工具
const customServer = createSdkMcpServer({
  name: "code-metrics",
  version: "1.0.0",
  tools: [
    tool(
      "analyze_complexity",
      "计算文件的圈复杂度",
      {
        filePath: z.string().describe("要分析的文件路径")
      },
      async (args) => {
        // 你的复杂度分析逻辑
        const complexity = Math.floor(Math.random() * 20) + 1; // 占位符
        return {
          content: [{
            type: "text",
            text: `${args.filePath} 的圈复杂度：${complexity}`
          }]
        };
      }
    )
  ]
});

// 使用 MCP 服务器的流式输入
async function* generateMessages() {
  yield {
    type: "user" as const,
    message: {
      role: "user" as const,
      content: "分析 main.ts 的复杂度"
    }
  };
}

for await (const message of query({
  prompt: generateMessages(),
  options: {
    model: "opus",
    mcpServers: {
      "code-metrics": customServer
    },
    allowedTools: ["Read", "mcp__code-metrics__analyze_complexity"],
    maxTurns: 250
  }
})) {
  // ...
}
```

## 成本跟踪

跟踪 API 成本用于计费：

```typescript
for await (const message of query({ prompt: "..." })) {
  if (message.type === "result" && message.subtype === "success") {
    console.log("总成本:", message.total_cost_usd);
    console.log("Token 使用量:", message.usage);
    
    // 按模型分解（对子代理有用）
    for (const [model, usage] of Object.entries(message.modelUsage)) {
      console.log(`${model}: $${usage.costUSD.toFixed(4)}`);
    }
  }
}
```

## 生产级代码审查代理

这里是一个整合所有功能的生产就绪代理：

```typescript
import { query, AgentDefinition } from "@anthropic-ai/claude-agent-sdk";

interface ReviewResult {
  issues: Array<{
    severity: "low" | "medium" | "high" | "critical";
    category: "bug" | "security" | "performance" | "style";
    file: string;
    line?: number;
    description: string;
    suggestion?: string;
  }>;
  summary: string;
  overallScore: number;
}

const reviewSchema = {
  type: "object",
  properties: {
    issues: {
      type: "array",
      items: {
        type: "object",
        properties: {
          severity: { type: "string", enum: ["low", "medium", "high", "critical"] },
          category: { type: "string", enum: ["bug", "security", "performance", "style"] },
          file: { type: "string" },
          line: { type: "number" },
          description: { type: "string" },
          suggestion: { type: "string" }
        },
        required: ["severity", "category", "file", "description"]
      }
    },
    summary: { type: "string" },
    overallScore: { type: "number" }
  },
  required: ["issues", "summary", "overallScore"]
};

async function runCodeReview(directory: string): Promise<ReviewResult | null> {
  console.log(`\n${"=".repeat(50)}`);
  console.log(`🔍 代码审查代理`);
  console.log(`📁 目录：${directory}`);
  console.log(`${"=".repeat(50)}\n`);

  let result: ReviewResult | null = null;

  for await (const message of query({
    prompt: `对 ${directory} 进行全面代码审查。

            分析所有源文件，重点关注：
            - Bug 和潜在运行时错误
            - 安全漏洞
            - 性能问题
            - 代码质量和可维护性

            请尽可能提供具体的文件路径和行号。`,
    options: {
      model: "opus",
      allowedTools: ["Read", "Glob", "Grep", "Task"],
      permissionMode: "bypassPermissions",
      maxTurns: 250,
      outputFormat: {
        type: "json_schema",
        schema: reviewSchema
      },
      agents: {
        "security-scanner": {
          description: "深度安全分析漏洞检测",
          prompt: `你是安全专家。扫描：
                  - 注入漏洞（SQL、XSS、命令注入）
                  - 认证和授权缺陷
                  - 敏感数据暴露
                  - 不安全的依赖`,
          tools: ["Read", "Grep", "Glob"],
          model: "sonnet"
        } as AgentDefinition
      }
    }
  })) {
    // 进度更新
    if (message.type === "assistant") {
      for (const block of message.message.content) {
        if ("name" in block) {
          if (block.name === "Task") {
            console.log(`🤖 委派给：${(block.input as any).subagent_type}`);
          } else {
            console.log(`📂 ${block.name}：${getToolSummary(block)}`);
          }
        }
      }
    }

    // 最终结果
    if (message.type === "result") {
      if (message.subtype === "success" && message.structured_output) {
        result = message.structured_output as ReviewResult;
        console.log(`\n✅ 审查完成！成本：$${message.total_cost_usd.toFixed(4)}`);
      } else {
        console.log(`\n❌ 审查失败：${message.subtype}`);
      }
    }
  }

  return result;
}

function getToolSummary(block: any): string {
  const input = block.input || {};
  switch (block.name) {
    case "Read": return input.file_path || "文件";
    case "Glob": return input.pattern || "模式";
    case "Grep": return `"${input.pattern}" 在 ${input.path || "."}`;
    default: return "";
  }
}

function printResults(result: ReviewResult) {
  console.log(`\n${"=".repeat(50)}`);
  console.log(`📊 审查结果`);
  console.log(`${"=".repeat(50)}\n`);
  
  console.log(`评分：${result.overallScore}/100`);
  console.log(`发现问题：${result.issues.length}\n`);
  console.log(`总结：${result.summary}\n`);
  
  const byCategory = {
    critical: result.issues.filter(i => i.severity === "critical"),
    high: result.issues.filter(i => i.severity === "high"),
    medium: result.issues.filter(i => i.severity === "medium"),
    low: result.issues.filter(i => i.severity === "low")
  };
  
  for (const [severity, issues] of Object.entries(byCategory)) {
    if (issues.length === 0) continue;
    
    const icon = severity === "critical" ? "🔴" :
                 severity === "high" ? "🟠" :
                 severity === "medium" ? "🟡" : "🟢";
    
    console.log(`\n${icon} ${severity.toUpperCase()} (${issues.length})`);
    console.log("-".repeat(30));
    
    for (const issue of issues) {
      const location = issue.line ? `${issue.file}:${issue.line}` : issue.file;
      console.log(`\n[${issue.category}] ${location}`);
      console.log(`  ${issue.description}`);
      if (issue.suggestion) {
        console.log(`  💡 ${issue.suggestion}`);
      }
    }
  }
}

// 运行审查
async function main() {
  const directory = process.argv[2] || ".";
  const result = await runCodeReview(directory);
  
  if (result) {
    printResults(result);
  }
}

main().catch(console.error);
```

运行：

```bash
npx tsx review-agent.ts ./src
```

---

**文章来源**：https://x.com/dabit3/status/2009131298250428923
