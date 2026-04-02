---
tags: [最佳实践, 协同, CC-OC, 工作流]
aliases: [协同工作流, CC+OC, Claude Code与OpenClaw协作, AI全家桶]
date: 2026-04-02
related: "[[Claude Code]], [[OpenClaw]], [[项目工作流]], [[MCP协议]], [[记忆系统]]"
---

> [!caution] 内容性质说明
> 本文描述的 Claude Code + OpenClaw 协同工作流为**编辑推断和社区建议**，综合自两个工具的各自文档。CC 与 OC 的集成场景、配置示例和协同架构目前**没有官方教程覆盖**，实际可用性请以两个项目的最新文档为准。MCP 桥接（mcporter）功能请参考 OpenClaw 官方文档确认当前支持状态。

# CC + OC 协同工作流

> [!summary] 一句话定义
> Claude Code 管**编程工作流**，OpenClaw 管**日常工作流**，两者通过 Claude API 和 MCP 协议形成 **AI 全家桶**。

---

## 为什么需要协同

> [!note] 互补定位
> | 维度 | Claude Code | OpenClaw |
> |------|------------|----------|
> | **核心场景** | 编程开发 | 日常自动化 |
> | **交互方式** | 终端 CLI | 聊天软件 |
> | **最佳时机** | 写代码时 | 不在电脑前时 |
> | **模型使用** | 深度推理 | 对话式响应 |
> | **上下文** | 项目代码 | 消息 + 日程 |

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

## 日常使用时间线

```
08:00  OpenClaw 推送今日待办（Telegram）
09:00  打开 Claude Code 开始编程
10:30  CC 完成功能，自动运行测试
10:31  OpenClaw 收到测试结果通知
12:00  午休，通过 OC 简单对话回顾上午进度
14:00  CC 继续开发
17:00  CC 提交代码，CI 触发审查
17:05  OpenClaw 推送审查结果
18:00  下班，通过 OC 处理简单任务
```

## 进阶：协同架构实战

### 完整协同架构图

当项目规模增长到多个 CC 项目和 OC 部署时，需要一个清晰的架构来管理所有联动关系：

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

### 三个实战场景

#### 场景1：CI/CD 通知闭环

目标：代码从提交到部署的全流程通知，团队成员即使不在电脑前也能实时掌握进度。

```
CC 提交代码
  → GitHub Actions 触发构建
    → 构建成功/失败
      → Webhook 通知 OpenClaw
        → OC 推送到 Telegram 群组
```

GitHub Actions 配置关键步骤：

```yaml
# .github/workflows/notify.yml
- name: Notify OpenClaw
  if: always()
  run: |
    curl -X POST "${{ secrets.OC_WEBHOOK_URL }}" \
      -H "Content-Type: application/json" \
      -d '{
        "repo": "${{ github.repository }}",
        "branch": "${{ github.ref_name }}",
        "status": "${{ job.status }}",
        "commit": "${{ github.sha }}",
        "actor": "${{ github.actor }}"
      }'
```

> [!tip] Webhook 安全
> 务必在 OC 端验证 Webhook 来源（通过 `secret` 签名校验），防止伪造通知。参考 [[OpenClaw]] 中的 Webhook 配置章节。

#### 场景2：代码审查流水线

目标：通过 Telegram 远程触发代码审查，结果自动回传。

```
Telegram 消息 "审查 feature/login 分支"
  → OC 收到消息，解析意图
    → 调用 CC API 执行审查
      → CC 运行审查技能（[[Skills开发]]）
        → 审查结果回传 OC
          → OC 格式化后推送到 Telegram
```

关键配置点：
1. **OC 端**：配置一个 Telegram Skill，监听包含"审查"关键词的消息，提取分支名
2. **CC API 端**：暴露一个 HTTP 接口，接收分支名参数，调用 [[Agent-SDK]] 的审查 Agent
3. **结果格式化**：OC 将 CC 返回的 JSON 审查结果转换为 Telegram 友好的 Markdown 格式

> [!warning] 注意事项
> 远程调用 CC API 时要考虑认证和安全。建议使用 API Key + HTTPS，并限制可审查的分支范围（如只允许 `feature/*` 和 `fix/*`）。

#### 场景3：知识库自动更新

目标：代码变更自动生成文档更新，团队成员通过 OC 随时查询项目状态。

```
CC 完成代码变更
  → 自动生成变更说明（commit message + diff 摘要）
    → 通过 Webhook 发送给 OC
      → OC 将变更说明记录到记忆系统
        → 团队成员通过 OC 查询"最近改了什么"
```

这个场景的核心是**变更说明的结构化**。在 CC 的 [[Hooks系统]] 中配置 `PostCommit` hook，自动提取关键信息：

```json5
// .claude/settings.json
{
  "hooks": {
    "PostCommit": [{
      "matcher": "",
      "hooks": [{
        "type": "command",
        "command": "git diff HEAD~1 --stat | curl -X POST ${OC_WEBHOOK}/api/changelog -d @-"
      }]
    }]
  }
}
```

### 模型分配策略表

不同任务类型对模型能力的需求差异很大，合理分配模型能显著优化成本与质量：

| 任务类型 | CC 模型 | OC 模型 | 原因 |
|---------|--------|--------|------|
| 代码生成 | Opus / Sonnet | -- | 需要最强推理能力和代码理解 |
| 日常对话 | -- | Sonnet | 平衡响应速度和回答质量 |
| 通知推送 | -- | Haiku | 简单格式化即可，追求最低延迟和成本 |
| 代码审查 | Opus | -- | 需要深度分析安全性和架构问题 |
| 日程管理 | -- | Haiku | 结构化数据处理，不需要深度推理 |
| 文档生成 | Sonnet | -- | 格式化输出为主，Sonnet 性价比最高 |
| 复杂分析 | Opus | Opus | 两端都可能需要深度思考 |
| 简单问答 | Haiku | Haiku | 追求速度，降低成本 |

> [!tip] 成本控制经验
> 日常使用中，约 80% 的任务可以用 Haiku 或 Sonnet 完成。只在遇到复杂架构决策、深层 bug 排查、安全审查时升级到 Opus。参考 [[性能与成本优化]] 中的详细成本分析。

---

**相关笔记**：[[Claude Code]] · [[OpenClaw]] · [[项目工作流]] · [[MCP协议]] · [[性能与成本优化]] · [[记忆系统]]
