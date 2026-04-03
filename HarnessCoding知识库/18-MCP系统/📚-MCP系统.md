---
type: system
tags: [claude-code, mcp, tool, protocol, harness, practice]
description: Claude Code MCP 系统核心机制 + 实战配置
source:
  - 官方文档: "参考文档/Claude-Code-x-OpenClaw-Guide-Zh-main/docs/claude-code/04-MCP集成完整指南.md"
  - MCP配置实现: "参考代码/claude-code/src/services/mcp/config.ts"
  - MCP类型定义: "参考代码/claude-code/src/services/mcp/types.ts"
  - MCP客户端: "参考代码/claude-code/src/services/mcp/client.ts"
  - MCP命令: "参考代码/claude-code/src/commands/mcp/mcp.tsx"
  - MCP工具: "参考代码/claude-code/src/tools/MCPTool/MCPTool.ts"
  - OMC-MCP实现: "参考代码/oh-my-claudecode/src/mcp/servers.ts"
---

# 📚 MCP 系统

> ⏱ 难度: ★★☆ | 重要性: ★★★ | 推荐学习时间: 2-3天

## 概述

MCP（Model Context Protocol）是 AI 工具的"USB接口标准"，让 Claude Code 能即插即用地连接各种外部服务。

### 核心价值

| 对比维度 | 传统集成方式 | MCP方式 |
|----------|-------------|---------|
| **开发成本** | 每个工具单独对接 | 一次开发，多处复用 |
| **兼容性** | 各平台接口不兼容 | 开放标准，跨平台通用 |
| **安全性** | 安全边界模糊 | 明确的权限控制和隔离 |

---

## 一、MCP协议工作原理

### 1.1 传输层机制

MCP 支持两种传输方式：

#### STDIO 传输（本地进程通信）

```
┌──────────────────┐         ┌──────────────────┐
│   MCP Client     │  stdin  │   MCP Server     │
│  (Claude Code)   │────────►│                  │
│                  │  stdout │                  │
│                  │◄────────│                  │
└──────────────────┘         └──────────────────┘
```

> 源码位置：`参考代码/claude-code/src/services/mcp/client.ts` - StdioClientTransport

- 最适合本地工具（filesystem、sqlite）
- 性能最优，进程级隔离
- 配置示例：
```json
{
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-filesystem", "./data"]
}
```

#### HTTP/Streamable 传输（远程通信）

```
┌──────────────────┐   HTTP   ┌──────────────────┐
│   MCP Client     │────────►│   MCP Server     │
│                  │   SSE   │   (Remote)       │
└──────────────────┘◄────────┘                  │
```

> 源码位置：`参考代码/claude-code/src/services/mcp/client.ts` - StreamableHTTPClientTransport

- 支持远程访问，多客户端共享
- 配置示例：
```json
{
  "type": "http",
  "url": "https://your-server.com/mcp",
  "headers": { "Authorization": "Bearer ${API_KEY}" }
}
```

### 1.2 JSON-RPC 通信机制

MCP 使用 JSON-RPC 2.0 协议进行通信，三种消息类型：

#### 请求（Request）
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "read_file",
    "arguments": { "path": "/path/to/file.txt" }
  }
}
```

#### 响应（Response）
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "content": [{ "type": "text", "text": "文件内容..." }]
  }
}
```

#### 通知（Notification）
```json
{
  "jsonrpc": "2.0",
  "method": "notifications/progress",
  "params": { "progressToken": "token-123", "progress": 50 }
}
```

### 1.3 三大组件

> 源码位置：`参考代码/claude-code/src/services/mcp/types.ts`

| 组件 | 作用 | 类比 |
|------|------|------|
| **Tools** | 可执行的函数 | 工具箱里的工具 |
| **Resources** | 只读数据资源 | 参考书架 |
| **Prompts** | 预设模板 | 填空表格 |

---

## 二、三作用域配置体系

> 源码位置：`参考代码/claude-code/src/services/mcp/config.ts` - ConfigScope 类型定义

### 2.1 三大作用域

| 作用域 | 存储位置 | 优先级 | 适用场景 |
|--------|----------|--------|----------|
| **Local** | `~/.claude.json` 项目条目 | 最高 | 个人私有配置、含API Key |
| **Project** | 项目根目录 `.mcp.json` | 中等 | 团队共享、版本控制 |
| **User** | `~/.claude.json` 全局部分 | 最低 | 个人常用工具 |

### 2.2 优先级合并规则

```
Local > Project > User
```

### 2.3 配置方法

#### Project 作用域（`.mcp.json`）
```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_PERSONAL_ACCESS_TOKEN}" }
    }
  }
}
```

#### Local 作用域（`~/.claude.json`）
```json
{
  "mcpServers": {
    "/Users/me/project1": {
      "github": {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-github"],
        "env": { "GITHUB_PERSONAL_ACCESS_TOKEN": "local-token" }
      }
    }
  }
}
```

#### CLI 命令方式
```bash
# 添加到 user 作用域
claude mcp add --scope user github npx -y @modelcontextprotocol/server-github

# 添加到 project 作用域
claude mcp add --scope project github npx -y @modelcontextprotocol/server-github
```

### 2.4 服务器去重机制

> 源码位置：`参考代码/claude-code/src/services/mcp/config.ts` - dedupPluginMcpServers

- 手动配置优先于插件服务器
- 插件服务器命名空间：`plugin:pluginName:serverName`
- 通过命令数组/URL签名进行内容去重

---

## 三、常用服务器配置

### 3.1 配置字段说明

> 源码位置：`参考代码/claude-code/src/services/mcp/types.ts` - McpServerConfigSchema

| 字段 | 必需 | 说明 |
|------|------|------|
| `command` | ✅ | 启动命令（npx/node/python） |
| `args` | ✅ | 命令参数数组 |
| `env` | ❌ | 环境变量对象 |
| `type` | ❌ | 传输类型（stdio/http/sse/ws） |

### 3.2 核心服务器速查

| 服务器 | 包名 | 用途 | API Key |
|--------|------|------|---------|
| **filesystem** | `@modelcontextprotocol/server-filesystem` | 文件读写 | ❌ |
| **github** | `@modelcontextprotocol/server-github` | GitHub管理 | ✅ |
| **sqlite** | `mcp-server-sqlite` (uvx) | SQLite数据库 | ❌ |
| **postgres** | `@modelcontextprotocol/server-postgres` | PostgreSQL | ❌ |
| **context7** | `@upstash/context7-mcp` | 技术文档 | ❌ |
| **brave-search** | `@modelcontextprotocol/server-brave-search` | 网页搜索 | ✅ |
| **memory** | `@modelcontextprotocol/server-memory` | 持久化记忆 | ❌ |
| **fetch** | `mcp-server-fetch` (uvx) | 网页获取 | ❌ |

### 3.3 配置示例

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "./src", "./docs"],
      "env": {}
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_PERSONAL_ACCESS_TOKEN}" }
    },
    "sqlite": {
      "command": "uvx",
      "args": ["mcp-server-sqlite", "--db-path", "./data/app.db"],
      "env": {}
    },
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp"],
      "env": {}
    }
  }
}
```

---

## 四、连接生命周期

> 源码位置：`参考代码/claude-code/src/services/mcp/client.ts` - MCP客户端连接管理

### 4.1 服务器状态类型

> 源码位置：`参考代码/claude-code/src/services/mcp/types.ts` - MCPServerConnection

```typescript
type MCPServerConnection =
  | ConnectedMCPServer   // 已连接
  | FailedMCPServer      // 连接失败
  | NeedsAuthMCPServer   // 需要认证
  | PendingMCPServer     // 连接中
  | DisabledMCPServer    // 已禁用
```

### 4.2 连接流程

```
1. 初始化阶段
   Client → Server: initialize请求（发送客户端能力）
   Server → Client: initialize响应（发送服务器能力）
   Client → Server: initialized通知（确认初始化完成）

2. 操作阶段
   Client → Server: 发送请求（tools/call, resources/read等）
   Server → Client: 返回响应或错误

3. 关闭阶段
   Client/Server: 关闭连接，清理资源
```

### 4.3 工具懒加载（v2.1.52+）

> 源码位置：`参考代码/claude-code/src/tools/MCPTool/MCPTool.ts` - ToolSearch机制

当配置大量MCP服务器时，懒加载机制延迟加载工具定义，减少上下文占用高达95%。

---

## 五、故障排查

### 5.1 常见问题排查流程

```
问题出现
    │
    ▼
1. 检查配置文件 ─── JSON格式错误？→ 修复JSON
    │
    ▼
2. 检查服务器状态 ─── 服务器启动失败？→ 查看日志
    │
    ▼
3. 检查环境变量 ─── API Key缺失？→ 配置环境变量
    │
    ▼
4. 检查网络 ─── 无法下载包？→ 配置镜像/代理
```

### 5.2 常见错误

| 错误 | 原因 | 解决方案 |
|------|------|----------|
| `ETIMEDOUT` | 网络超时 | 配置npm镜像 |
| `ENOENT` | 文件不存在 | 检查路径是否正确 |
| `Invalid JSON` | JSON格式错误 | 使用jq验证 |
| `Missing env var` | 环境变量未设置 | 导出环境变量 |

### 5.3 日志查看

```bash
# macOS/Linux
tail -f ~/.local/share/claude/logs/mcp*.log

# Windows
Get-Content "$env:LOCALAPPDATA\Claude\logs\mcp*.log" -Wait

# 调试模式启动
claude --mcp-debug
```

---

## 六、相关模块

- [[📚-MCP系统]]（当前模块）
- [[🔧-Tool系统]] - MCP工具最终通过Tool系统暴露
- [[🔐-权限系统]] - MCP操作需要权限检查
- [[🪝-Hook系统]] - Elicitation可在Hook中拦截处理
- [[📚-Skill系统]] - OMC Skill系统使用MCP作为扩展机制

---

## 七、知识来源

> 📌 **信息来源**：
> - [MCP官方文档](https://modelcontextprotocol.io/) | 验证日期：2026-02-25
> - [GitHub MCP Server仓库](https://github.com/modelcontextprotocol/servers) | 验证日期：2026-02-25
> - [Claude Code文档](https://code.claude.com/docs/en/mcp) | 验证日期：2026-02-25

**适用版本**：MCP规范 2025-11-25 / Claude Code v2.1.69+

---

> 📝 **学习建议**：
> 1. 先配置一个简单的 filesystem MCP 感受效果
> 2. 理解三作用域配置是正确使用MCP的关键
> 3. 阅读源码加深对MCP协议工作原理的理解
> 4. 尝试开发一个自定义MCP服务器
