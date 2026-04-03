---
type: omc-guide
tags: [omc, notepad, memory, persistence, internal]
description: OMC Notepad 内部机制详解
---

# 📒 OMC Notepad 内部机制

## 1. Notepad 概述

### 1.1 什么是 OMC Notepad

OMC Notepad 是 OMC 框架中的**跨会话持久化笔记系统**，用于在多个 Claude Code 会话之间共享重要上下文和状态信息。它解决了 LLM 记忆窗口有限、跨会话上下文丢失的核心问题。

### 1.2 解决的问题

| 问题 | 传统方式 | Notepad 方式 |
|-----|---------|-------------|
| **会话丢失** | 每次会话结束上下文消失 | 持久化到文件系统 |
| **重复解释** | 新会话需要重新描述背景 | 读取历史上下文 |
| **状态丢失** | 长任务中断后难以恢复 | 暂存状态便于续接 |
| **团队协作** | Agent 间无法共享上下文 | 通过文件传递信息 |

---

## 2. 文件格式与结构

### 2.1 文件位置

Notepad 文件位于工作目录的 `.omc/notepad.md`：

```
.omc/
├── notepad.md           # 主笔记文件
├── state/               # 状态存储
│   └── sessions/         # 会话级状态
├── plans/               # 计划文件
└── research/            # 研究资料
```

### 2.2 格式规范

Notepad 使用标准 Markdown 格式，支持 Frontmatter 元数据：

```markdown
---
type: notepad
created: 2026-04-03T10:00:00Z
updated: 2026-04-03T15:30:00Z
priority: high
tags: [project-x, backend, important]
---

# 笔记标题

正文内容...
```

### 2.3 支持的元数据

| 字段 | 类型 | 说明 |
|-----|------|------|
| `type` | string | 固定值 `notepad` |
| `created` | ISO8601 | 创建时间戳 |
| `updated` | ISO8601 | 更新时间戳 |
| `priority` | string | 优先级：high / normal / low |
| `tags` | string[] | 分类标签 |
| `session_id` | string | 关联会话 ID |

### 2.4 与 MEMORY.md 的区别

| 维度 | Notepad | MEMORY.md |
|-----|---------|----------|
| **位置** | `.omc/notepad.md` | `.claude/MEMORY.md` |
| **用途** | 跨会话笔记、状态暂存 | 项目级持久记忆 |
| **格式** | Markdown + Frontmatter | 纯 Markdown |
| **写入频率** | 按需写入 | 定期更新 |
| **生命周期** | 手动清理 | 长期累积 |
| **访问方式** | `notepad_*` 工具 | 直接读取 |

---

## 3. 工作原理

### 3.1 会话状态追踪

OMC 通过 Notepad 实现会话状态的持久化：

```
会话 A (开始)
  └── 写入初始状态 → notepad.md
  └── 执行任务...
  └── 更新进度 → notepad.md
会话 A (结束)

会话 B (开始)
  └── 读取 notepad.md → 获取上下文
  └── 继续任务...
```

**状态暂存内容**：

- 当前任务进度和目标
- 遇到的问题和解决方案
- 重要决策记录
- 下一步行动计划

### 3.2 Agent 间上下文共享

Notepad 作为多 Agent 协作的**共享画板**：

```
Agent-1 (Planner)
  └── 写入计划 → notepad.md
  └── 通知 Agent-2

Agent-2 (Executor)
  └── 读取 notepad.md → 获取计划
  └── 执行...
  └── 写入结果 → notepad.md
  └── 通知 Agent-3

Agent-3 (Verifier)
  └── 读取 notepad.md → 获取上下文
  └── 验证结果
```

### 3.3 同步机制

#### 文件锁机制

OMC 使用文件系统锁防止并发写入冲突：

```typescript
// 写入前获取锁
async function writeWithLock(path: string, content: string) {
  const lockPath = `${path}.lock`;
  await acquireLock(lockPath);
  try {
    await writeFile(path, content);
  } finally {
    await releaseLock(lockPath);
  }
}
```

#### 冲突处理策略

| 场景 | 处理方式 |
|-----|---------|
| **同时写入** | 文件锁保证串行写入 |
| **读取时写入** | 写入完成后通知读取方重试 |
| **锁超时** | 30 秒超时后强制释放 |

#### 原子性保证

写入操作采用**读取-修改-写入**模式：

```typescript
async function appendNote(content: string) {
  const current = await readFile(NOTEPAD_PATH);
  const updated = current + '\n' + content;
  await writeWithLock(NOTEPAD_PATH, updated);
}
```

---

## 4. 使用场景

### 4.1 跨会话记忆

**场景**：用户需要在多次会话中完成同一个大型任务。

**用法**：

```javascript
// 会话 1
notepad_write_manual({
  content: `# 大型重构任务

## 目标
重构 user-auth 模块

## 进度
- [x] 分析依赖关系
- [ ] 制定迁移计划
- [ ] 执行重构

## 注意事项
用户模块与支付模块高度耦合`
});

// 会话 2
notepad_read();  // 获取之前的状态
// 继续执行...
```

### 4.2 Agent 间通信

**场景**：Team 工作流中多个 Agent 需要协调。

**用法**：

```javascript
// Planner Agent
notepad_write_priority({
  content: `## 任务分配

- Agent-1: 负责 API 层重构
- Agent-2: 负责数据层迁移
- Agent-3: 负责测试覆盖`
});

// Executor Agent
const plan = notepad_read();
// 根据计划执行...
notepad_write_working({
  content: `## 执行状态

API 层重构完成，数据层进行中`
});
```

### 4.3 状态暂存

**场景**：长任务中断后需要恢复现场。

**用法**：

```javascript
// 任务中断前
notepad_write_manual({
  content: `## 中断状态

迁移进度: 70%
当前表: orders
已迁移记录: 15,000 / 21,000
错误记录: 23 条（见下方）

错误详情:
- ID: 1042, reason: foreign_key_constraint
- ID: 2103, reason: data_length_exceeded`
});

// 恢复后
const state = notepad_read();
// 从中断点继续...
```

---

## 5. 内部实现细节

### 5.1 写入时机

Notepad 在以下时机自动写入：

| 时机 | 触发条件 | 写入内容 |
|-----|---------|---------|
| **会话结束** | `SessionEnd` Hook | 会话摘要、状态快照 |
| **任务完成** | Agent 报告完成 | 执行结果、产物位置 |
| **重要决策** | 检测到关键决策点 | 决策理由、备选方案 |
| **错误发生** | 工具调用失败 | 错误上下文、尝试修复 |
| **手动触发** | 调用 `notepad_write_*` | 用户指定内容 |

### 5.2 读取策略

```
notepad_read()
│
├── 检查缓存 (session 级别)
│   └── 有 → 返回缓存
│
├── 检查文件
│   └── 有 → 解析 Frontmatter
│   └── 无 → 返回空
│
└── 返回结果
```

**读取优先级**：

1. Session 缓存（最新会话）
2. `.omc/notepad.md`（主文件）
3. `.omc/state/sessions/{sessionId}/notepad.md`（会话私有）

### 5.3 清理机制

Notepad 采用**手动清理 + 自动归档**策略：

#### 手动清理

```javascript
// 使用 notepad_prune 清理过期内容
notepad_prune({
  olderThan: "7d",  // 7 天前的内容
  tags: ["temporary"]  // 仅清理带有 temporary 标签的条目
});
```

#### 自动归档

| 条件 | 动作 |
|-----|------|
| 超过 30 天未访问 | 移动到 `.omc/archive/` |
| 文件超过 100KB | 提示清理 |
| Session 超过 90 天 | 归档到历史记录 |

#### 保留策略

以下内容**永不自动清理**：

- 带有 `pinned: true` 标记的笔记
- 带有 `important` 标签的条目
- 最新 10 条会话摘要

---

## 6. 相关章节

- [[../05-记忆系统/05-01-🧠-记忆系统]] - 记忆系统
- [[13-01-🏁-OMC入门]] - OMC 入门
- [[13-02-🎯-OMC核心概念]] - OMC 核心概念
- [[13-03-🔄-OMC工作流]] - OMC 工作流
