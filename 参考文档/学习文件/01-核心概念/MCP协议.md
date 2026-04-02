---
tags: [核心概念, MCP, 协议]
aliases: [MCP, Model Context Protocol, 模型上下文协议]
date: 2026-04-02
related: "[[Claude Code]], [[OpenClaw]], [[技能系统]], [[Agent-SDK]]"
---

# MCP 协议 (Model Context Protocol)

> [!summary] 一句话定义
> AI 模型与外部工具通信的**标准协议**，类似 USB 接口标准，让 Claude 能连接任意外部工具。

---

## 核心概念

| 概念 | 类比 | 说明 |
|------|------|------|
| **MCP Server** | USB 设备 | 提供具体功能的服务端 |
| **MCP Client** | USB 主机 | 调用功能的客户端（Claude Code） |
| **Tool** | 设备功能 | Server 暴露的原子操作 |
| **Resource** | 设备数据 | Server 提供的数据源 |
| **Prompt** | 设备预设 | Server 定义的提示模板 |

## 在 Claude Code 中的应用

### 配置文件：`.mcp.json`

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "./src"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_TOKEN": "${GITHUB_TOKEN}" }
    }
  }
}
```

### 核心能力

- **工具懒加载** (ToolSearch) — 上下文减少 95%
- **MCP Apps** — 交互式界面
- **MCP Elicitation** — 交互式信息请求
- **远程 MCP 服务器** — 连接远程工具

## 在 OpenClaw 中的应用

OpenClaw 通过 **mcporter** 桥接 MCP：

```
OpenClaw Gateway ←→ mcporter ←→ MCP Servers
```

> [!tip] 桥接优势
> - 添加/更换 MCP 服务器不需重启 Gateway
> - MCP 生态变动不影响核心稳定性
> - 核心工具保持精简

## 安全配置

> [!warning] MCP 安全要点
> - 通过 `permissions.deny` 禁止危险 MCP 操作
> - 使用环境变量传递凭证，不硬编码
> - 通过 [[Hooks系统]] 记录 MCP 调用日志

```json
{
  "permissions": {
    "allow": ["mcp__context7__get_library_docs"],
    "deny": ["mcp__*__delete_*", "mcp__*__drop_*"]
  }
}
```

## 常用 MCP 服务器

| 服务器 | 功能 | 配置难度 |
|--------|------|---------|
| filesystem | 文件系统操作 | ⭐ |
| github | GitHub API 操作 | ⭐⭐ |
| postgres | PostgreSQL 查询 | ⭐⭐ |
| brave-search | 网页搜索 | ⭐ |
| memory | 知识图谱记忆 | ⭐⭐ |

---

**相关笔记**：[[Claude Code]] · [[OpenClaw]] · [[技能系统]] · [[权限与沙箱]] · [[Agent-SDK]]
