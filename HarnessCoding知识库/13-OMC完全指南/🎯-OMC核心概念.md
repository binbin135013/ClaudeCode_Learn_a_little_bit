---
type: omc-guide
tags: [oh-my-claudecode, omc, core-concepts, agent, skill, hook]
description: OMC 核心概念 - 三层架构、Skill、Hook、工作流编排
---

# 🎯 OMC 核心概念

## 1. 三层 Agent 架构 (Haiku/Sonnet/Opus)

### 1.1 架构概览

OMC 采用三层模型体系，根据任务复杂度自动选择合适的模型：

```
┌─────────────────────────────────────────────────────┐
│                    OMC Agent 层                      │
│                                                     │
│   ┌─────────┐    ┌─────────┐    ┌─────────┐      │
│   │  Haiku  │    │ Sonnet  │    │  Opus   │      │
│   │ (轻量)  │    │ (标准)  │    │ (深度)  │      │
│   └────┬────┘    └────┬────┘    └────┬────┘      │
│        │              │              │             │
│        ▼              ▼              ▼             │
│   快速查找      标准实现        架构设计           │
│   轻量检查      调试审查        深度分析           │
└─────────────────────────────────────────────────────┘
```

### 1.2 三层模型对比

| 模型 | 能力等级 | 速度 | 成本 | 适用场景 |
|-----|---------|------|------|---------|
| **Haiku** | 基础 | 最快 | 最低 | 快速查找、文件浏览、简单检查 |
| **Sonnet** | 标准 | 中等 | 中等 | 实现、调试、审查、重构 |
| **Opus** | 高级 | 最慢 | 最高 | 架构设计、复杂规划、高风险审查 |

### 1.3 Agent 模型分配

| Agent | 默认模型 | 说明 |
|-------|---------|------|
| `explore` | haiku | 快速代码库搜索和映射 |
| `writer` | haiku | 文档和简洁内容 |
| `debugger` | sonnet | 根因分析和故障诊断 |
| `executor` | sonnet | 实现和重构 |
| `verifier` | sonnet | 完成证据和验证 |
| `test-engineer` | sonnet | 测试策略和回归覆盖 |
| `planner` | opus | 排序和执行计划 |
| `architect` | opus | 系统设计和边界 |
| `code-reviewer` | opus | 综合代码审查 |

### 1.4 模型选择原则

```
任务复杂度
├── 查找/浏览        → Haiku
│   └── 示例: 查找文件、搜索代码、读文档
├── 实现/调试/审查   → Sonnet
│   └── 示例: 写功能、修复 bug、代码审查
└── 架构/规划/深度分析 → Opus
    └── 示例: 系统设计、复杂重构、高风险决策
```

---

## 2. Skill 系统详解

### 2.1 Skill 定义

Skill 是 OMC 的**工作流封装单元**，将复杂的多步骤任务封装为可复用的技能模块。

### 2.2 内置 Skill 一览

| Skill | 触发词 | 核心功能 |
|-------|--------|---------|
| `autopilot` | "autopilot" | 全自动执行，从想法到代码 |
| `ralph` | "ralph" | 持久循环直到任务完成 |
| `ultrawork` | "ulw", "ultrawork" | 高吞吐量并行执行 |
| `team` | "team" | 多 Agent 协调编排 |
| `ccg` | "ccg" | Claude+Codex+Gemini 三模型综合 |
| `ultraqa` | "ultraqa" | 测试-验证-修复-重复循环 |
| `plan` | "plan", "ralplan" | 战略规划工作流 |
| `sciomc` | "sciomc" | 并行科学研究工作流 |
| `deep-interview` | "deep interview" | 苏格拉底式需求澄清 |
| `ask` | "ask" | 智能模型路由 |
| `learner` | "learner" | 从对话提取可复用技能 |

### 2.3 Skill 调用方式

#### 方式一：斜杠命令

```
/skill autopilot 构建一个 REST API
/skill ralph 修复所有安全漏洞
/skill ultrawork 并行处理数据导入
```

#### 方式二：关键词触发

```
autopilot 创建一个博客系统
ralph 直到这个 bug 被修复
ulw 并行处理这 10 个文件
```

#### 方式三：OMC 前缀

```
/oh-my-claudecode:autopilot
/oh-my-claudecode:ralph
/oh-my-claudecode:team
```

### 2.4 Skill 元数据格式

```yaml
---
name: my-custom-skill          # 技能唯一标识
description: 我的自定义技能      # 功能描述
type: skill                     # 类型固定为 skill
trigger:
  - "my-skill"                  # 触发词
  - "ms"
version: "1.0.0"                # 版本号
author: "Your Name"            # 作者
tags: [custom, utility]         # 标签

# Agent 配置
agent:
  model: sonnet                # 使用模型
  temperature: 0.7             # 温度参数
  max_tokens: 4096             # 最大 token

# 工作流配置
workflow:
  parallel: false              # 是否并行
  retry: 3                     # 重试次数
  timeout: 300000              # 超时(ms)

# 依赖配置
requires:
  tools: [Read, Edit, Bash]    # 所需工具
  skills: []                   # 依赖的其他技能
---

# Skill 实现内容
```

### 2.5 参数替换规则

```bash
# 位置参数
$0, $1, $2 ... $N
$ARGUMENTS      # 完整参数

# 命名参数 (基于 frontmatter 的 arguments 字段)
# arguments: foo bar
$foo            # 第一个命名参数
$bar            # 第二个命名参数
```

---

## 3. Hook 机制

### 3.1 Hook 与 Skill 的关系

```
┌─────────────────────────────────────────┐
│            OMC 扩展体系                   │
│                                         │
│  ┌─────────────────────────────────┐    │
│  │         Skill (能力封装)         │    │
│  │  基于 Hook 构建，用户显式调用    │    │
│  └─────────────────────────────────┘    │
│                   │                     │
│                   ▼                     │
│  ┌─────────────────────────────────┐    │
│  │         Hook (生命周期钩子)      │    │
│  │  事件驱动，系统自动触发          │    │
│  └─────────────────────────────────┘    │
│                                         │
│  ┌─────────────────────────────────┐    │
│  │      Claude Code 底层系统        │    │
│  └─────────────────────────────────┘    │
└─────────────────────────────────────────┘
```

### 3.2 OMC 中的 Hook 类型

| 类型 | 用途 | 触发时机 |
|-----|------|---------|
| **Command Hook** | CLI 命令拦截 | 命令执行前后 |
| **Prompt Hook** | Prompt 修改 | AI 请求前 |
| **Agent Hook** | Agent 生命周期 | Agent 启动/结束 |
| **Function Hook** | 自定义函数 | 自定义事件 |

### 3.3 核心 Hook 事件

```
工具调用层:
  - PreToolUse          (工具调用前)
  - PostToolUse         (工具调用后)
  - PostToolUseFailure  (工具调用失败)

用户交互层:
  - UserPromptSubmit    (用户消息提交)
  - Notification        (通知)

会话管理层:
  - SessionStart        (会话开始)
  - SessionEnd          (会话结束)

子代理层:
  - SubagentStart       (子代理启动)
  - SubagentStop        (子代理停止)

压缩层:
  - PreCompact          (压缩前)
  - PostCompact         (压缩后)
```

### 3.4 Hook 响应协议

```typescript
interface HookResponse {
  status: "approve" | "block" | "modify";
  updatedInput?: unknown;      // 修改后的输入
  additionalContext?: string;  // 注入的上下文
  reason?: string;             // 决策原因
}
```

### 3.5 OMC Hook 配置

```javascript
// ~/.claude/settings.json
{
  "hooks": {
    "enabled": true,
    "items": [
      "./hooks/omc-audit.js",
      "./hooks/omc-context-inject.js"
    ]
  }
}
```

### 3.6 常见 OMC Hook 用例

```javascript
// 1. 自动记录任务执行
const taskAuditHook = {
  event: "PostToolUse",
  filter: { tool: ["Write", "Edit", "Bash"] },
  async handle(context) {
    await notepad_write_manual({
      content: `执行: ${context.tool}\n输入: ${JSON.stringify(context.input)}`
    });
    return { status: "approve" };
  }
};

// 2. 敏感操作拦截
const dangerousBashHook = {
  event: "PreToolUse",
  filter: { tool: "Bash", pattern: /rm\s+-rf|drop\s+table/i },
  async handle(context) {
    return {
      status: "block",
      reason: "危险命令已拦截"
    };
  }
};

// 3. 上下文自动注入
const contextInjectHook = {
  event: "UserPromptSubmit",
  async handle(context) {
    return {
      status: "modify",
      additionalContext: await loadProjectContext()
    };
  }
};
```

---

## 4. 工作流编排

### 4.1 工作流体系

```
OMC 工作流
├── 🚀 Autopilot    - 全自主执行模式
├── 🔄 Ralph        - 持久修正循环
├── ⚡ Ultrawork    - 并行执行引擎
├── 👥 Team         - 多 Agent 协作
├── 🔬 CCG          - 三模型综合
└── 🧪 UltraQA      - QA 循环
```

### 4.2 工作流选择决策树

```
任务类型
├── 端到端实现  → autopilot
│   └── 需求明确，一步到位
│
├── 多轮迭代  → ralph
│   └── 需要持续修正直到达标
│
├── 独立并行  → ultrawork
│   └── 多个任务可同时执行
│
├── 分工协作  → team
│   └── 复杂任务需要多角色分工
│
├── 多模型综合 → ccg
│   └── 需要 Claude+Codex+Gemini 各自优势
│
└── 质量驱动  → ultraqa
    └── 测试-验证-修复-重复
```

### 4.3 工作流组合

```
autopilot
  └── plan (内部规划)
      └── team (多 Agent 执行)
          ├── Agent 1: 架构设计
          ├── Agent 2: 实现
          └── Agent 3: 测试
      └── ultraqa (质量验证)
          ├── 测试编写
          ├── 验证执行
          └── 修复循环
```

### 4.4 Team Pipeline

Team 工作流遵循标准管道：

```
team-plan → team-prd → team-exec → team-verify → team-fix (循环)
```

| 阶段 | 说明 |
|-----|------|
| `team-plan` | 任务分解和计划 |
| `team-prd` | 需求评审 |
| `team-exec` | 执行实施 |
| `team-verify` | 验证结果 |
| `team-fix` | 修复问题 (可循环) |

### 4.5 状态管理

OMC 使用状态存储来跟踪任务进度：

```javascript
// 状态文件位置
.omc/state/
├── sessions/{sessionId}/
│   ├── ultrawork-state.json    // ultrawork 并行状态
│   └── mission-state.json       // 任务状态
├── agent-replay-{sessionId}.jsonl  // Agent 操作记录
└── notepad.md                   // 笔记
```

---

## 5. 记忆系统

### 5.1 记忆类型

| 记忆类型 | 持久化 | 用途 |
|---------|-------|------|
| **Notepad** | 永久 | 跨会话重要笔记 |
| **Project Memory** | 永久 | 项目级上下文 |
| **Session Memory** | 会话级 | 当前会话上下文 |
| **State** | 临时 | 任务状态跟踪 |

### 5.2 Notepad 操作

| 命令 | 功能 |
|-----|------|
| `notepad_read` | 读取笔记 |
| `notepad_write_priority` | 写优先级笔记 |
| `notepad_write_working` | 写工作中笔记 |
| `notepad_write_manual` | 手动写笔记 |

### 5.3 Project Memory

```javascript
// 添加项目记忆
project_memory_add_note({
  content: "这个模块使用领域驱动设计",
  tags: ["ddd", "architecture"]
});

// 读取项目记忆
project_memory_read();
```

---

## 6. 相关资源

- [[🏁-OMC入门]] - 安装与快速上手
- [[🔄-OMC工作流]] - 四大工作流详解
- [[🛠️-OMC自定义开发]] - 自定义 Skill 和 Hook
- [[../08-Skill系统/📚-Skill系统]] - Claude Code Skill 系统
- [[../04-Hook系统/🪝-Hook系统]] - Claude Code Hook 系统
