---
type: system
tags: [claude-code, openclaw, collaboration, workflow]
description: Claude Code与OpenClaw协同工作流实战
---

# 🤝 Claude Code + OpenClaw 协同工作流

> ⏱ 难度: ★★☆ | 重要性: ★★★ | 推荐学习时间: 2-3天
> 💡 **特别推荐**: 想用 AI 全家桶？这是核心篇！

---

## 概述

### 为什么需要协同

> [!note] 互补定位
> | 维度 | Claude Code | OpenClaw |
> |------|------------|----------|
> | **核心场景** | 编程开发 | 日常自动化 |
> | **交互方式** | 终端 CLI | 聊天软件 |
> | **最佳时机** | 写代码时 | 不在电脑前时 |
> | **模型使用** | 深度推理 | 对话式响应 |
> | **上下文** | 项目代码 | 消息 + 日程 |

---

## 协同场景矩阵

### 场景 1：编程 + 消息通知

```
Claude Code（本地终端）
    ├── 编写代码...
    ├── CI 通过/失败 → Webhook
    ↓
OpenClaw（Telegram/WhatsApp）
    ├── 接收 CI 通知
    ├── "你的构建失败了，错误在 src/auth.ts:42"
    └── 在手机上直接回复 "帮我看看" → 触发代码审查
```

### 场景 2：日程驱动开发

```
OpenClaw（日程提醒）
    ├── "今天要完成用户认证模块"
    └── 推送提醒到 WhatsApp
                ↓
Claude Code（开始编码）
    ├── "实现用户认证模块"
    └── 参考项目 CLAUDE.md 规范
```

### 场景 3：知识管理闭环

```
OpenClaw（Obsidian 技能）
    ├── 搜索和管理笔记
    ├── AI 帮你整理知识
                ↓
Claude Code（项目开发）
    ├── 参考笔记中的架构决策
    └── 代码实现
```

---

## 技术集成点

### 1. 共享 Claude API

```json5
// 两者使用同一个 Anthropic API Key
// Claude Code: ANTHROPIC_API_KEY 环境变量
// OpenClaw: openclaw.json 中的 model 配置
{
  "model": "anthropic/claude-sonnet-4-6"
}
```

### 2. MCP 协议桥接

```
Claude Code ← MCP → [共享工具] ← mcporter → OpenClaw
                                ├── GitHub
                                ├── 文件系统
                                └── 数据库
```

### 3. Webhook 联动

```json5
// Claude Code Hook → OpenClaw 通知
// .claude/settings.json
{
  "hooks": {
    "Notification": [{
      "matcher": "",
      "hooks": [{
        "type": "command",
        "command": "curl -X POST http://localhost:18789/api/message -d '{\"msg\":\"$CLAUDE_OUTPUT\"}'"
      }]
    }]
  }
}
```

---

## 模型分配策略

> [!tip] 成本优化
> | 任务 | 工具 | 模型 | 原因 |
> |------|------|------|------|
> | 代码生成 | CC | Sonnet/Opus | 需要深度推理 |
> | 代码审查 | CC | Opus | 安全敏感 |
> | 快速修复 | CC | Haiku | 成本低 |
> | 日常对话 | OC | Sonnet | 平衡质量成本 |
> | 技能执行 | OC | Sonnet | 工具调用场景 |
> | 复杂分析 | OC | Opus | 需要深度思考 |

---

## 完整协同架构图

```
┌─────────────┐     Webhook      ┌──────────────┐
│  Claude Code │ ───────────────→ │   OpenClaw    │
│  (编程助手)   │ ←─── API ──────  │  (日常助手)   │
└──────┬──────┘                   └──────┬───────┘
       │                                 │
       │  MCP                            │  Skills
       ▼                                 ▼
┌─────────────┐                   ┌──────────────┐
│  开发工具链   │                   │  消息平台     │
│  Git/GH/CI   │                   │  TG/WA/DC    │
└─────────────┘                   └──────────────┘
```

核心数据流向：
- **CC -> OC**：代码提交、CI 结果、构建状态（通过 Webhook）
- **OC -> CC**：任务指令、审查请求、部署触发（通过 API）
- **CC <-> 工具链**：Git 操作、GitHub PR 管理、CI/CD 触发（通过 MCP 和 CLI）
- **OC <-> 消息平台**：Telegram/WhatsApp/Discord 收发消息（通过 Skills）

---

## 三个实战场景

### 场景1：CI/CD 通知闭环

目标：代码从提交到部署的全流程通知，团队成员即使不在电脑前也能实时掌握进度。

```
CC 提交代码
  → GitHub Actions 触发构建
    → 构建成功/失败
      → Webhook 通知 OpenClaw
        → OC 推送到 Telegram 群组
```

### 场景2：代码审查流水线

目标：通过 Telegram 远程触发代码审查，结果自动回传。

```
Telegram 消息 "审查 feature/login 分支"
  → OC 收到消息，解析意图
    → 调用 CC API 执行审查
      → CC 运行审查技能
        → 审查结果回传 OC
          → OC 格式化后推送到 Telegram
```

### 场景3：知识库自动更新

目标：代码变更自动生成文档更新，团队成员通过 OC 随时查询项目状态。

```
CC 完成代码变更
  → 自动生成变更说明（commit message + diff 摘要）
    → 通过 Webhook 发送给 OC
      → OC 将变更说明记录到记忆系统
        → 团队成员通过 OC 查询"最近改了什么"
```

---

## 相关章节

- [[../../01-架构总览/📐-架构概览]] - 架构概览
- [[../../04-Hook系统/🪝-Hook系统]] - Hook 系统
- [[../../05-记忆系统/🧠-记忆系统]] - 记忆系统
- [[../../08-Skill系统/📚-Skill系统]] - Skill 系统
- [[🏠-知识库首页]] - 返回首页
