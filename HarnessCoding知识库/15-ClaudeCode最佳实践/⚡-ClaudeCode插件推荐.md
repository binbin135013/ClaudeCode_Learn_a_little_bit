---
type: best-practice
tags: [claude-code, plugin, extension, mcp, omc, superpowers]
description: Claude Code 插件和扩展推荐清单
---

# Claude Code 插件推荐

> ⏱ 适用水平: 入门到进阶 | 推荐程度: 必读

---

## 1. OMC (oh-my-claudecode) - 多智能体编排

### 1.1 简介

OMC 是 Claude Code 的**元框架**，提供多智能体编排、任务自动化和高级工作流管理能力。

### 1.2 核心功能

| 功能 | 说明 |
|-----|------|
| **多 Agent 编排** | `team`、`ralph`、`ultrawork` 等 Skill |
| **工作流自动化** | `autopilot`、`ultraqa` 循环 |
| **深度研究** | `sciomc`、`deep-interview` |
| **上下文管理** | 智能上下文压缩和路由 |

### 1.3 常用 Skill

```bash
# 自动化执行
/autopilot 创建一个完整的用户认证系统

# 持久化循环
/ralph 直到这个 bug 被修复

# 并行执行
/ultrawork 并行处理这 10 个文件的重构

# 团队协作
/team 3 个 Agent 完成这个项目

# 质量保证
/ultraqa 全面测试这个模块

# 深度规划
/plan 为这个功能制定详细计划
```

### 1.4 安装配置

```bash
# 通过 npm 安装
npm install -g oh-my-claudecode

# 初始化配置
omc setup

# 查看可用技能
omc skill list
```

---

## 2. Superpowers - 技能链扩展

### 2.1 简介

Superpowers 是**技能链扩展框架**，允许将多个 Skill 串联成复杂的工作流。

### 2.2 核心 Skill

| Skill | 功能 | 触发词 |
|-------|------|--------|
| `dispatching-parallel-agents` | 并行 Agent 分发 | "dispatch" |
| `brainstorming` | 头脑风暴工作流 | "brainstorm" |
| `test-driven-development` | TDD 工作流 | "tdd" |
| `systematic-debugging` | 系统调试 | "debug" |
| `verification-before-completion` | 完成前验证 | "verify" |
| `writing-skills` | 编写新 Skill | "skill-create" |

### 2.3 技能链示例

```bash
# 头脑风暴 → 规划 → 执行 → 验证
brainstorm 做一个电商推荐系统
  → plan 制定实现计划
  → team 3 个 Agent 开发
  → ultraqa 全面测试
  → verify 最终验证
```

---

## 3. MCP Servers - 模型上下文协议

### 3.1 什么是 MCP

MCP (Model Context Protocol) 是 Anthropic 推出的**标准化上下文协议**，允许 Claude 与外部数据源和工具集成。

### 3.2 热门 MCP Servers

| Server | 用途 | Stars |
|--------|------|-------|
| **filesystem** | 文件系统操作 | 高 |
| **git** | Git 操作 | 高 |
| **github** | GitHub API | 高 |
| **memory** | 持久化记忆 | 中 |
| **slack** | Slack 集成 | 中 |
| **database** | 数据库连接 | 中 |

### 3.3 安装 MCP Server

```bash
# 安装官方 MCP Server
claude mcp install filesystem
claude mcp install git
claude mcp install github

# 查看已安装
claude mcp list
```

### 3.4 MCP 配置示例

```json
// ~/.claude/settings.json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "~/projects"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "your-token"
      }
    }
  }
}
```

---

## 4. VS Code 插件推荐

### 4.1 必装插件

| 插件 | 功能 | 重要性 |
|------|------|--------|
| **Claude Code** | 主插件 | ⭐⭐⭐ |
| **GitLens** | Git 可视化 | ⭐⭐⭐ |
| **GitHub Copilot** | 代码补全备选 | ⭐⭐ |
| **Error Lens** | 错误即时显示 | ⭐⭐ |
| **Todo Tree** | TODO 管理 | ⭐⭐ |
| **REST Client** | API 测试 | ⭐⭐ |

### 4.2 效率插件

| 插件 | 功能 | 重要性 |
|------|------|--------|
| **Prettier** | 代码格式化 | ⭐⭐⭐ |
| **ESLint** | 代码检查 | ⭐⭐⭐ |
| **Auto Rename Tag** | 自动重命名 | ⭐⭐ |
| **Bracket Pair Colorizer** | 括号高亮 | ⭐⭐ |
| **Path Intellisense** | 路径补全 | ⭐⭐ |
| **Thunder Client** | API 客户端 | ⭐⭐ |

### 4.3 知识管理插件 (Obsidian 联动)

| 插件 | 功能 | 重要性 |
|------|------|--------|
| **Obsidian** | 笔记管理 | ⭐⭐⭐ |
| **Dataview** | 数据查询 | ⭐⭐⭐ |
| **Reminder** | 学习提醒 | ⭐⭐ |
| **Breadcrumbs** | 导航增强 | ⭐⭐ |

---

## 5. 其他实用插件和工具

### 5.1 CLI 工具

| 工具 | 用途 | 安装 |
|------|------|------|
| **oh-my-claudecode** | OMC 框架 | `npm i -g oh-my-claudecode` |
| **claude-code-cli** | CLI 增强 | `npm i -g claude-code-cli` |
| **tree-sitter** | 代码解析 | 预装 |

### 5.2 浏览器扩展

| 扩展 | 功能 | 平台 |
|------|------|------|
| **Claude for Search** | 搜索增强 | Chrome |
| **Claude Clipboard** | 剪贴板同步 | Chrome/Firefox |

### 5.3 开发工具链

| 工具 | 用途 |
|------|------|
| **Docker** | 容器化开发环境 |
| **kubectl** | Kubernetes 管理 |
| **terraform** | 基础设施即代码 |
| **ansible** | 配置管理 |

---

## 6. 插件组合推荐

### 6.1 入门组合

```
必需: Claude Code + OMC + GitLens + Prettier
推荐: + Dataview + Todo Tree + REST Client
```

### 6.2 进阶组合

```
入门 + Superpowers + MCP Servers + Error Lens
+ Thunder Client + Docker
```

### 6.3 专业组合

```
进阶 + 自定义 Hook + 自定义 Skill
+ 全套 MCP Servers + CI/CD 集成
```

---

## 7. 安装优先级建议

### 第一阶段 (必装)
1. Claude Code 主插件
2. oh-my-claudecode
3. GitLens + Prettier

### 第二阶段 (推荐)
4. Superpowers
5. Dataview (Obsidian)
6. REST Client / Thunder Client

### 第三阶段 (可选)
7. MCP Servers
8. 自定义 Skill/Hook
9. CI/CD 集成

---

## 相关章节

- [[../04-Hook系统/🪝-Hook系统]] - Hook 配置
- [[../08-Skill系统/📚-Skill系统]] - Skill 开发
- [[🏗️-整体架构建议]] - 项目结构
- [[📋-Step-by-Step教程]] - 逐步搭建

---

> [!cite]- 知识来源
>
> | 知识点 | 来源 |
> |-------|------|
> | **OMC 技能列表** | [oh-my-claudecode](https://github.com/oh-my-claudecode/oh-my-claudecode) |
> | **Superpowers** | [superpowers](https://github.com/superpowers/superpowers) |
> | **MCP Servers** | [modelcontextprotocol](https://github.com/modelcontextprotocol) |
> | **everything-claude-code** | [everything-claude-code](https://github.com/everything-claude-code/everything-claude-code)（高 Star 项目集合） |
