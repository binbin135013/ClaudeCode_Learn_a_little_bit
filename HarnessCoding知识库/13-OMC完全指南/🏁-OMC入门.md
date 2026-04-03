---
type: omc-guide
tags: [oh-my-claudecode, omc, getting-started, setup]
description: OMC (oh-my-claudecode) 完全指南 - 入门篇
---

# 🏁 OMC 入门

## 1. OMC 简介与核心价值

### 1.1 什么是 OMC

OMC (oh-my-claudecode) 是 Claude Code 的**多智能体编排框架**，通过协调专业化 Agent 完成复杂任务。它不是 Claude Code 的替代品，而是运行在 Claude Code 之上的**元层 orchestration 系统**。

```
┌─────────────────────────────────────────┐
│           用户 (You)                     │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│     OMC (多Agent编排层)                  │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐   │
│  │ Explorer│ │ Planner │ │ Executor│   │
│  └─────────┘ └─────────┘ └─────────┘   │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│      Claude Code (底层执行层)            │
│  QueryEngine + Tool + Permission + ...  │
└─────────────────────────────────────────┘
```

### 1.2 OMC 的核心价值

| 价值维度 | 说明 |
|---------|------|
| **专业化分工** | Haiku/Sonnet/Opus 三层模型各司其职 |
| **工作流自动化** | autopilot/ralph/ultrawork/team 四大工作流 |
| **记忆持久化** | Notepad + Project Memory 跨会话记忆 |
| **团队协作** | 多 Agent 并行/串行协作完成任务 |
| **自我修正** | Ralph 循环实现持续改进直到达标 |

### 1.3 OMC vs 原生 Claude Code

| 维度 | 原生 Claude Code | OMC |
|-----|----------------|-----|
| **执行模式** | 单 Agent 对话 | 多 Agent 编排 |
| **任务复杂度** | 简单到中等 | 复杂、多阶段任务 |
| **自我修正** | 依赖人工反馈 | Ralph 循环自动修正 |
| **并行能力** | 单线执行 | ultrawork 并行分发 |
| **团队协作** | 无 | team 多 Agent 协作 |

---

## 2. 安装与配置步骤

### 2.1 环境要求

| 要求 | 最低版本 | 推荐版本 |
|-----|---------|---------|
| Node.js | 18.0+ | 20.x LTS |
| Claude Code | 最新版 | 始终保持最新 |
| 操作系统 | macOS/Windows/Linux | macOS 或 Ubuntu |
| 内存 | 8GB | 16GB+ |

### 2.2 安装方式

#### 方式一：Claude Code 原生集成（推荐）

```
# 在 Claude Code 中直接调用
/claude omc-setup
```

#### 方式二：独立安装

```bash
# 克隆 OMC 仓库
git clone https://github.com/caoyuan/oh-my-claudecode.git

# 进入目录
cd oh-my-claudecode

# 安装依赖
npm install

# 全局链接
npm link
```

### 2.3 目录结构

```
oh-my-claudecode/
├── src/
│   ├── agents/           # Agent 定义
│   │   ├── executor.ts
│   │   ├── planner.ts
│   │   ├── explorer.ts
│   │   └── ...
│   ├── skills/           # Skill 工作流
│   │   ├── autopilot.ts
│   │   ├── ralph.ts
│   │   ├── ultrawork.ts
│   │   └── team.ts
│   ├── hooks/            # OMC Hook
│   ├── memory/           # 记忆系统
│   └── index.ts
├── skills/               # 用户自定义 Skill
├── state/               # 状态存储
├── plans/               # 计划文件
└── README.md
```

### 2.4 配置文件

OMC 配置位于 `~/.claude/omc/` 目录：

```json
// ~/.claude/omc/config.json
{
  "model": {
    "default": "sonnet",
    "haiku": "haiku",
    "sonnet": "sonnet-4",
    "opus": "opus-3"
  },
  "workspace": {
    "root": "~/omc-workspace",
    "maxConcurrent": 5
  },
  "memory": {
    "enabled": true,
    "path": "~/.claude/omc/memory"
  },
  "hooks": {
    "enabled": true,
    "items": []
  }
}
```

---

## 3. 快速上手指南

### 3.1 第一个 OMC 命令

```bash
# 在 Claude Code 中启动 omc
omc

# 或者使用 skill 方式
/oh-my-claudecode:omc-setup
```

### 3.2 基础命令

| 命令 | 功能 | 示例 |
|-----|------|------|
| `/omc-setup` | 初始化 OMC 环境 | `/oh-my-claudecode:omc-setup` |
| `/omc-doctor` | 诊断 OMC 状态 | `/oh-my-claudecode:omc-doctor` |
| `/cancel` | 取消当前任务 | `/oh-my-claudecode:cancel` |
| `/team N` | 启动 N 个 Agent 团队 | `/team 3` |
| `/skill <name>` | 调用 Skill 工作流 | `/skill autopilot` |

### 3.3 四种基础工作流

#### 3.3.1 Autopilot - 全自动执行

```
autopilot 创建一个用户认证系统
```

**适用场景**：需求明确，从想法到实现一步到位。

#### 3.3.2 Ralph - 持久修正循环

```
ralph 持续优化性能直到达到 100ms 以下
```

**适用场景**：需要多轮迭代直到满足条件。

#### 3.3.3 Ultrawork - 并行执行

```
ultrawork 并行处理这 10 个文件的重构
```

**适用场景**：多个独立任务可同时执行。

#### 3.3.4 Team - 团队协作

```
team 3 个 Agent 完成这个重构任务
```

**适用场景**：复杂任务需要分工协作。

### 3.4 Agent 模型路由

OMC 使用三层模型体系：

| 模型 | 用途 | 触发条件 |
|-----|------|---------|
| **Haiku** | 快速查找、轻量检查 | `/omc:explore` |
| **Sonnet** | 标准实现、调试、审查 | 默认模型 |
| **Opus** | 架构设计、深度分析 | `/omc:architect` |

---

## 4. 常用命令速查

### 4.1 Skill 调用速查

| Skill | 触发词 | 功能 |
|-------|--------|------|
| `autopilot` | "autopilot" | 全自主执行 |
| `ralph` | "ralph" | 自我修正循环 |
| `ultrawork` | "ulw", "ultrawork" | 并行执行 |
| `team` | "team" | 团队协作 |
| `ccg` | "ccg" | 三模型综合 |
| `ultraqa` | "ultraqa" | QA 循环 |
| `plan` | "plan" | 战略规划 |
| `sciomc` | "sciomc" | 研究工作流 |
| `deep-interview` | "deep interview" | 需求澄清 |
| `ask` | "ask" | 模型路由 |

### 4.2 Agent 目录

| Agent | 模型 | 用途 |
|-------|------|------|
| `explore` | haiku | 快速代码库搜索 |
| `analyst` | opus | 需求分析 |
| `planner` | opus | 计划编排 |
| `architect` | opus | 架构设计 |
| `debugger` | sonnet | 调试诊断 |
| `executor` | sonnet | 实现重构 |
| `verifier` | sonnet | 验证确认 |
| `tracer` | sonnet | 追踪取证 |
| `code-reviewer` | opus | 代码审查 |

### 4.3 状态管理命令

| 命令 | 功能 |
|-----|------|
| `state_read` | 读取 OMC 状态 |
| `state_write` | 写入状态 |
| `state_clear` | 清除状态 |
| `state_list_active` | 列出活跃状态 |

### 4.4 Notepad 命令

| 命令 | 功能 |
|-----|------|
| `notepad_read` | 读取笔记 |
| `notepad_write_priority` | 写优先级笔记 |
| `notepad_write_working` | 写工作中笔记 |
| `notepad_write_manual` | 手动写笔记 |

### 4.5 Team 运行时命令

| 命令 | 功能 |
|-----|------|
| `TeamCreate` | 创建团队 |
| `TeamDelete` | 删除团队 |
| `SendMessage` | 发送消息 |
| `TaskCreate` | 创建任务 |
| `TaskList` | 列出任务 |
| `TaskUpdate` | 更新任务 |

---

## 5. 第一个实战任务

### 5.1 任务：使用 Team 协作重构模块

**目标**：使用 3 个 Agent 协作完成模块重构

**步骤**：

```
# 1. 启动团队协作
/team 3

# 2. Agent 1 (explorer) 分析代码结构
/explorer 分析 src/ 目录的依赖关系

# 3. Agent 2 (planner) 制定重构计划
/planner 制定重构计划

# 4. Agent 3 (executor) 执行重构
/executor 按照计划执行重构

# 5. 验证结果
/verifier 验证重构后的代码质量
```

### 5.2 使用 Ralph 循环修正问题

```
# 启动 Ralph 循环
/ralph 直到所有测试通过

# Ralph 会自动：
# 1. 运行测试
# 2. 发现失败
# 3. 分析原因
# 4. 修复问题
# 5. 重复直到全部通过
```

---

## 6. 相关资源

- [[../01-架构总览/📐-架构概览]] - Claude Code 底层架构
- [[../08-Skill系统/📚-Skill系统]] - Skill 系统详解
- [[../04-Hook系统/🪝-Hook系统]] - Hook 扩展机制
- [[🎯-OMC核心概念]] - 三层 Agent 架构
- [[🔄-OMC工作流]] - 四大工作流详解
