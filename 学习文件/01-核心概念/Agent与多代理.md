---
tags: [核心概念, Agent, 多代理]
aliases: [Agent, 子代理, Subagent, 多代理协作, Agent Teams]
date: 2026-04-02
related: "[[Claude Code]], [[OpenClaw]], [[Agent-SDK]], [[架构设计]]"
---

# Agent 与多代理协作

> [!summary] 一句话定义
> 让多个 AI 代理分工协作的模式。Claude Code 有 Subagent/Agent Teams，OpenClaw 有多 Agent 路由。

---

## Claude Code 的 Agent 体系

### Subagent 子代理

| 类型 | 说明 | 示例 |
|------|------|------|
| **内置专家** | 100+ 预定义专家代理 | `code-reviewer`, `architect`, `debugger` |
| **Agent Teams** | 多代理团队协作 | 并行执行、任务分工 |
| **自定义** | 通过 Skills 创建 | 项目特定代理 |

### Agent Teams 工作流

```
用户指令
    ↓
主代理（协调者）
    ├── 子代理 A（前端）  ← 并行执行
    ├── 子代理 B（后端）  ← 并行执行
    └── 子代理 C（测试）  ← 等待 A+B 完成
    ↓
结果汇总 → 主代理 → 用户
```

> [!tip] DAG 依赖
> Agent Teams 支持 DAG（有向无环图）依赖关系，子代理可以并行或串行执行。

## OpenClaw 的多 Agent 路由

### 架构

```
消息 → Gateway 路由引擎
           ├── Agent "工作助手"（Telegram）
           ├── Agent "生活管家"（WhatsApp）
           └── Agent "代码审查"（Discord）
```

每个 Agent 有独立的：
- 人格设定（System Prompt）
- 技能集（Skills）
- 记忆（Memory）
- 模型配置（Model）

### 路由规则

| 规则 | 说明 |
|------|------|
| 平台路由 | 不同平台 → 不同 Agent |
| 联系人路由 | 不同联系人 → 不同 Agent |
| 关键词路由 | 特定关键词 → 特定 Agent |
| 群聊路由 | 每个群独立会话 |

## 统一抽象：Agent SDK

用 [[Agent-SDK]] 编程开发自定义 Agent，统一 Claude Code 和 OpenClaw 的 Agent 开发范式。

---

**相关笔记**：[[Claude Code]] · [[OpenClaw]] · [[Agent-SDK]] · [[架构设计]] · [[Subagent子代理]]
