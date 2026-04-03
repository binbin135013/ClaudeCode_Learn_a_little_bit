---
type: project-analysis
tags: [opencode, oh-my-openagent, multi-agent, orchestration, openagent]
description: oh-my-openagent 项目深度解析 - 多模型编排的 OpenCode 插件框架
---

# oh-my-openagent 深度解析

> ⏱ 难度: ★★★ | 重要性: ⭐⭐⭐⭐ | 推荐学习时间: 2-3天
> 🔗 源码: [oh-my-openagent](https://github.com/oh-my-openagent/oh-my-openagent)
> 💡 **定位**: 将基础 Agent 转化为多模型编排的"全能瑞士军刀"

---

## 1. 项目概述

### 1.1 什么是 oh-my-openagent

oh-my-openagent 是一个专为 OpenCode 设计的**高级插件框架**，它将简单的单模型 Agent 转化为具备以下能力的复杂系统：

- **多模型编排**: 5+ 专业 Agent 同时工作
- **自主执行**: `ultrawork` 一键驱动直到任务完成
- **智能规划**: Interview 模式的战略规划
- **精确编辑**: Hash 锚定的文件修改（从 93% 失败率降至 32%）

### 1.2 核心设计理念

| 理念 | 说明 |
|-----|------|
| **插件优先** | 不替换 OpenCode，而是扩展它 — 完全兼容 Hook/Command/Skill/MCP |
| **模型即服务** | 不同任务分配给最适合的模型 |
| **Hash 锚定编辑** | 内容哈希确保编辑精确性 |
| **按需加载** | Skill 自带 MCP server，用时才激活 |

---

## 2. 架构解析

### 2.1 Agent 体系

oh-my-openagent 定义了 5 个专业 Agent：

| Agent | 角色 | 模型倾向 |
|-------|------|---------|
| **Sisyphus** | 命令中枢 — 理解用户意图，分发任务 | Sonnet |
| **Hephaestus** | 执行者 — 实际运行命令和代码 | Opus |
| **Prometheus** | 规划师 — 战略规划，面试式追问 | Opus |
| **Oracle** | 代码搜索 — 语义搜索和代码库理解 | Sonnet |
| **Librarian** | 知识管理 — RAG 和知识检索 | Haiku |

### 2.2 Agent 分类框架

```
visual-engineering  → 视觉/UI 相关任务
deep                → 复杂分析/架构设计
quick               → 简单快速的修改
ultrabrain          → 深度学习和复杂推理
```

系统根据任务复杂度自动选择最优 Agent 类型。

### 2.3 核心模块结构

```
src/
├── commands/          # CLI 命令定义
├── agents/            # Agent 实现
├── skills/            # Skill 系统
├── hooks/             # Hook 扩展点
├── mcp/               # MCP 协议集成
└── utils/             # 工具函数
```

---

## 3. 核心功能详解

### 3.1 ultrawork / ulw

**功能**: 一键自主执行，驱动所有 Agent 直到任务完成。

```bash
/ulw 完成这个用户认证模块的重构
```

**执行流程**:
1. Sisyphus 分析任务意图
2. Prometheus 制定执行计划
3. Hephaestus 执行具体步骤
4. Oracle 提供代码上下文
5. Librarian 管理知识记忆
6. 循环直到任务完成

### 3.2 IntentGate

**功能**: 意图分析门禁 — 在行动前分析用户真实意图，避免误解。

```bash
# 用户说"修复它"
IntentGate 分析:
- 用户可能指: 编译错误？测试失败？功能异常？
- 需要追问确认还是直接尝试？
```

**解决的问题**: Agent 错误理解用户意图导致做无用功或破坏性操作。

### 3.3 Hash 锚定编辑

**功能**: 基于内容哈希的精确编辑。

**原理**:
```
传统方式: 找到"第5行"然后修改
Hash锚定: 计算内容哈希，确认位置后再修改
```

**效果**:
- Grok Code Fast 编辑失败率: 93%
- 使用 Hash 锚定后: 32%
- 减少了 61% 的无效编辑

### 3.4 Prometheus 规划模式

**功能**: Interview 模式的战略规划。

```bash
/prompts-plan
# AI 会追问:
# - "你希望通过重构解决什么问题？"
# - "有哪些边界情况需要考虑？"
# - "成功的标准是什么？"
```

### 3.5 Deep Context Init

**功能**: 自动生成树形 AGENTS.md 文件。

```bash
/init-deep
# 分析项目结构
# 生成符合规范的 AGENTS.md
# 包含所有子目录和关键文件
```

---

## 4. Skill 系统

### 4.1 Skill-Injected MCP

每个 Skill 可以自带 MCP server，按需激活：

```javascript
// skill 示例
{
  name: "code-review",
  mcp: {
    command: "npx",
    args: ["mcp-server-code-review"],
    triggers: ["/review", "code review"]
  }
}
```

### 4.2 内置 Skill 列表

| Skill | 功能 | 触发词 |
|-------|------|--------|
| `ultrawork` | 自主执行 | `/ulw`, `/ultrawork` |
| `ralph` | 持久化循环 | `/ralph` |
| `deep-interview` | 深度访谈 | `/deep-interview` |
| `autopilot` | 自动驾驶 | `/autopilot` |
| `sciomc` | 科学推理 | `/sciomc` |

---

## 5. 技术栈

| 组件 | 技术 |
|-----|------|
| **运行时** | Bun + TypeScript |
| **AI SDK** | @opencode-ai/sdk |
| **协议** | @modelcontextprotocol/sdk |
| **代码分析** | @ast-grep/napi |
| **验证** | Zod |
| **CLI** | commander |

---

## 6. 与 Claude Code 的对比

| 维度 | Claude Code | oh-my-openagent |
|-----|------------|-----------------|
| **模型** | Anthropic 独占 | 多模型混用 |
| **交互** | 单 Agent CLI | 多 Agent 协作 |
| **编辑** | 传统方式 | Hash 锚定 |
| **扩展** | Hook/Skill | 完整插件系统 |
| **透明度** | 闭源 | 开源可改 |

---

## 7. 学习要点

### 7.1 值得学习的模式

1. **多 Agent 编排**: 如何让多个专业 Agent 协作
2. **意图识别**: IntentGate 的设计思路
3. **Hash 锚定编辑**: 精确编辑的解决方案
4. **Skill 自带 MCP**: 按需加载的架构

### 7.2 可以借鉴到 Claude Code 的想法

- 多模型混用来提升效率
- Intent 分析增强指令理解
- 更精确的文件编辑策略

---

## 相关章节

- [[16-01-🚀-OpenCode与开源Agent指南]] - 返回总览
- [[../09-子Agent与协作/09-01-🤝-子Agent与协作]] - 多 Agent 协作
- [[../08-Skill系统/08-01-📚-Skill系统]] - Skill 系统
- [[../02-Tool系统/02-01-🔧-Tool系统]] - Tool 系统

---

> [!cite]- 知识来源
>
> | 知识点 | 来源 |
> |-------|------|
> | **oh-my-openagent 源码** | [github.com/oh-my-openagent](https://github.com/oh-my-openagent/oh-my-openagent) |
> | **架构设计** | 项目 AGENTS.md 和 README |
