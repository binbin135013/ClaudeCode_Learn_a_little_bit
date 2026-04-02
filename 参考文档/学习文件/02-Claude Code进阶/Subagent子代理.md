---
tags: [Claude-Code进阶, Subagent, 代理]
aliases: [Subagent, 子代理, 专家代理, Agent Teams]
date: 2026-04-02
related: "[[Claude Code]], [[Agent与多代理]], [[Agent-SDK]], [[技能系统]]"
---

# Subagent 子代理

> [!summary] 一句话定义
> Claude Code 内置的**专家代理系统**，100+ 预定义专家可按需调用，支持多代理团队并行协作。

---

## 代理类型

### 内置专家代理（100+）

| 类别 | 代理 | 功能 |
|------|------|------|
| **代码** | `code-reviewer` | 代码审查 |
| | `architect` | 架构设计 |
| | `debugger` | 调试排错 |
| **安全** | `security-reviewer` | 安全审计 |
| **数据库** | `database-reviewer` | 数据库优化 |
| **测试** | `tdd-guide` | TDD 引导 |
| | `e2e-runner` | E2E 测试 |
| **文档** | `doc-updater` | 文档更新 |

> [!note] 关于代理名称
> 上表中的名称为功能类别描述，实际可用的代理名称因版本而异。可通过自然语言方式调用，例如 `claude "请用代码审查专家帮我检查 src/auth.ts"`。

### Agent Teams（v2.1.78+）

多代理团队协作模式：
- **并行执行** — 独立任务同时运行
- **DAG 依赖** — 有向无环图管理执行顺序
- **上下文注入** — 父代理向子代理传递信息

```
Team Lead（协调者）
    ├── Frontend Agent（并行）
    ├── Backend Agent（并行）
    └── Test Agent（等待 Frontend + Backend）
```

## 调用方式

### 单代理调用

```bash
# 通过 Agent 工具调用
claude "请使用 code-reviewer 审查 src/auth.ts"
```

### Agent Teams 配置

Agent Teams 是 Claude Code 内置功能（v2.1.78+），可通过原生 API 直接使用。oh-my-claudecode 等 [[Plugins生态]] 扩展在此基础上提供了额外的编排模式（如 autopilot、team workflow 等）。

> [!tip] 最佳实践
> 1. **合理分工** — 每个子代理专注一个领域
> 2. **控制并行度** — 不要同时启动超过 5 个子代理
> 3. **明确依赖** — 用 DAG 清晰定义执行顺序
> 4. **结果验证** — 子代理结果需要主代理审核

## 子代理 vs Skills

| 维度 | Subagent | Skill |
|------|----------|-------|
| **执行方式** | 独立上下文，后台运行 | 主会话内联执行 |
| **并行能力** | 支持并行 | 串行 |
| **适用场景** | 耗时、独立任务 | 快速、上下文相关 |
| **资源消耗** | 较高（独立上下文） | 较低 |

---

## 进阶：多代理架构实战

### Agent Teams 配置实战

```
场景：全栈功能开发
├── architect (Opus) — 设计方案
│   ↓ 输出：实施方案文档
├── [并行] frontend (Sonnet) + backend (Sonnet) — 实现
│   ↓ 输出：代码变更
├── [并行] code-reviewer (Opus) + test-engineer (Sonnet) — 验证
│   ↓ 输出：审查报告 + 测试结果
└── executor (Sonnet) — 修复审查发现的问题
```

### 模型路由最佳实践

| 代理角色 | 推荐模型 | 理由 |
|---------|---------|------|
| architect | Opus | 需要全局视野和深度推理 |
| executor | Sonnet | 标准实现，性价比高 |
| code-reviewer | Opus | 需要发现隐含问题 |
| tdd-guide | Sonnet | 规则明确，执行为主 |
| doc-updater | Haiku | 文档格式化，简单任务 |
| security-reviewer | Opus | 安全审计需要深度分析 |
| e2e-runner | Sonnet | 测试执行，中等复杂度 |

### 常见陷阱

> [!warning] 多代理开发常见错误
> 1. **过度并行** — 同时启动 10+ 子代理，API 限流 + 上下文爆炸。正确：控制在 3-5 个
> 2. **循环依赖** — agent A 等 B，B 等 A。正确：设计 DAG 确保无环
> 3. **忽略结果验证** — 子代理结果直接用。正确：主代理必须审核
> 4. **共享文件冲突** — 多个子代理同时写同一文件。正确：每个子代理负责不同文件
> 5. **上下文重复传递** — 把完整历史传给每个子代理。正确：只传必要信息

### 子代理 vs Skills 选择决策

```
你的任务需要？
├── 独立上下文（不需要主会话历史）→ Subagent (fork)
│   └── 例：代码审查、文档生成、安全扫描
├── 需要主会话上下文 → Skill (shared)
│   └── 例：基于当前对话的代码修改、上下文感知格式化
└── 需要并行执行多个任务 → Agent Teams
    └── 例：同时测试前端和后端
```

---

**相关笔记**：[[Claude Code]] · [[Agent与多代理]] · [[Agent-SDK]] · [[技能系统]] · [[Skills开发]] · [[常见陷阱与反模式]]
