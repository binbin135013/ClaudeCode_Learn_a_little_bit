---
tags: [Claude-Code进阶, Agent-SDK, 开发]
aliases: [Agent-SDK, SDK, Agent开发, Claude API]
date: 2026-04-02
related: "[[Agent与多代理]], [[Claude Code]], [[MCP协议]]"
---

# Agent SDK

> [!summary] 一句话定义
> Anthropic 提供的 **Agent 开发 SDK**，用编程方式创建自定义 AI Agent，支持工具调用、子代理编排、上下文管理。

---

## 核心能力

| 能力 | 说明 |
|------|------|
| **工具定义** | 自定义 Agent 可调用的工具 |
| **子代理编排** | Task 工具 + DAG 依赖管理 |
| **上下文注入** | 父代理向子代理传递信息 |
| **会话管理** | 多轮对话、历史管理 |
| **模型路由** | 不同任务用不同模型 |

## 子代理编排（Task 工具）

```typescript
// 定义子代理类型
const agents = {
  researcher: { model: "sonnet", tools: ["web_search", "read"] },
  coder: { model: "opus", tools: ["read", "write", "bash"] },
  reviewer: { model: "opus", tools: ["read", "grep"] }
};

// DAG 依赖
const tasks = [
  { id: "research", agent: "researcher", deps: [] },
  { id: "implement", agent: "coder", deps: ["research"] },
  { id: "review", agent: "reviewer", deps: ["implement"] }
];
```

## 与 CLI 内置 Subagent 的区别

| 维度 | CLI Subagent | Agent SDK |
|------|-------------|-----------|
| **使用方式** | 命令行配置 | 编程开发 |
| **自定义程度** | 有限 | 完全自定义 |
| **部署方式** | 本地 CLI | 应用/服务 |
| **适用场景** | 开发辅助 | 产品/自动化 |

> [!tip] 何时用 Agent SDK
> - 需要构建**可分发的 AI 应用**
> - 需要精确控制 Agent 的**每一步行为**
> - 需要将 Agent **部署为服务**
> - 需要集成到**现有产品**中

## 学习路径

```
1. 掌握 CLI Subagent 使用 → [[Subagent子代理]]
2. 理解 Agent 概念 → [[Agent与多代理]]
3. 学习 SDK API 文档
4. 开发自定义 Agent
5. 部署和集成
```

## 进阶：SDK 架构模式

### 常见 Agent 架构模式

根据任务复杂度选择合适的编排模式：

| 模式 | 适用场景 | 示例 | 优缺点 |
|------|---------|------|--------|
| **单代理** | 简单、独立的任务 | 代码格式化、单文件重构 | 简单可靠，但无法并行 |
| **流水线** | 步骤间有顺序依赖 | 分析代码 -> 修复问题 -> 运行测试 | 逻辑清晰，但总耗时等于各步之和 |
| **并行+合并** | 多个独立子任务 | 前端测试 + 后端测试同时跑 | 速度快，但合并结果可能冲突 |
| **编排者+工人** | 复杂、可分解的任务 | architect 分配 -> executor 执行 -> verifier 验证 | 灵活强大，但编排逻辑复杂 |

> [!tip] 选择建议
> 1-2 个步骤用单代理；3-5 个顺序步骤用流水线；独立子任务多于 2 个用并行；需要动态决策时用编排者模式。多数项目从流水线开始，遇到瓶颈再升级。

### DAG 依赖设计原则

Agent SDK 使用有向无环图（DAG）管理子代理间的依赖关系。以下是设计 DAG 时的核心原则：

- **永远不要设计循环依赖** -- A 依赖 B、B 又依赖 A 会导致死锁。SDK 会检测到循环并抛出错误。设计前先画出任务关系图，确认无环。
- **最长链决定总耗时，尽量并行化** -- DAG 的关键路径（最长依赖链）决定了整体完成时间。找出独立任务，去掉不必要的依赖边，让它们并行执行。
- **每个节点要有明确的输入/输出契约** -- 子代理 A 的输出格式必须与子代理 B 的输入要求匹配。在 DAG 定义中为每个节点写清 `input` 和 `output` 的数据结构。

```
# 好的 DAG 设计：关键路径短，并行度高
research_a ──┐
             ├── merge ─── report
research_b ──┘

# 差的 DAG 设计：不必要的串行
research_a ── research_b ─── report
（如果 a 和 b 互相独立，应该并行）
```

### Tool 定义最佳实践

Tool 是 Agent 与外部世界交互的唯一途径。好的 tool 定义能显著提升 Agent 的执行准确率：

```typescript
// 好的 tool 定义：输入输出明确
{
  name: "search_files",
  description: "在指定目录搜索匹配 glob 模式的文件，返回文件路径列表",
  input_schema: {
    type: "object",
    properties: {
      pattern: {
        type: "string",
        description: "glob 模式，如 '**/*.ts' 或 'src/**/*.test.ts'"
      },
      directory: {
        type: "string",
        description: "搜索根目录的绝对路径"
      }
    },
    required: ["pattern"]
  }
}

// 坏的 tool 定义：模糊不清，Agent 无法正确调用
{
  name: "do_stuff",
  description: "执行操作",
  input_schema: { type: "object" }
}
```

> [!note] Tool 定义的三个关键
> 1. **description 要写清"做什么"和"返回什么"** -- Agent 根据 description 决定是否调用和如何传参
> 2. **每个 property 都要有 description** -- 帮助 Agent 理解参数的含义和格式
> 3. **required 数组要准确** -- 漏标可选参数会导致 Agent 传递不必要的值，多标必填参数会导致调用失败

更多关于 tool 定义的细节，参考 [[MCP协议]] 中的 tool schema 规范。

---

**相关笔记**：[[Agent与多代理]] · [[Subagent子代理]] · [[MCP协议]] · [[Claude Code]]
