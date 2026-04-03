---
type: project-analysis
tags: [opencode, claw-code, python, reverse-engineering, claude-code]
description: claw-code 项目解析 - Python 实现的 Claude Code 架构复刻
---

# claw-code 项目解析

> ⏱ 难度: ★★☆ | 重要性: ⭐⭐⭐ | 推荐学习时间: 1-2天
> 🔗 源码: [claw-code](https://github.com/instructkr/claw-code) | Stars: 50K+
> 💡 **定位**: 2小时破 50K stars 纪录的 Python 架构重写

---

## 1. 项目概述

### 1.1 什么是 claw-code

claw-code 是一个**干净的 Python 重写**项目，基于 2026年3月泄露的 Claude Code 源码逆向工程而来。它将 TypeScript 的 Claude Code 架构用 Python 重新实现，适合：

- Python 开发者学习 Agent 架构
- 需要与 Python 技术栈集成的项目
- 想要深度定制 Agent 的开发者

### 1.2 惊人的成长记录

| 时间 | Stars |
|-----|-------|
| 发布 2 小时内 | 50K+ ⭐ |
| 一周内 | 80K+ ⭐ |

> [!note]- 为什么这么快？
> - Claude Code 源码泄露带来的热度
> - Python 的广泛受众
> - 开发者对 Agent 架构的强烈兴趣

---

## 2. 架构解析

### 2.1 核心模块

```
src/
├── commands.py        # CLI 命令定义
├── models.py          # 数据结构
├── tools.py           # Tool 定义（对应 Claude Code 的 Tool）
├── query_engine.py    # 查询引擎
├── main.py            # CLI 入口
├── task.py            # 任务系统
└── port_manifest.py   # 端口清单
```

### 2.2 与 Claude Code 的模块对照

| Claude Code (TS) | claw-code (Python) | 功能 |
|-----------------|-------------------|------|
| `src/commands/` | `commands.py` | CLI 命令 |
| `src/tools/` | `tools.py` | 工具定义 |
| `src/query/` | `query_engine.py` | 查询执行 |
| `src/agent/` | `task.py` | Agent 逻辑 |
| `src/models/` | `models.py` | 数据模型 |

---

## 3. 技术特点

### 3.1 Python 实现的优势

| 优势 | 说明 |
|-----|------|
| **生态丰富** | Python AI/数据科学库丰富 |
| **易于集成** | 可以直接 import 现有 Python 代码 |
| **学习曲线** | 对 Python 开发者更友好 |
| **部署简单** | Python 环境随处可见 |

### 3.2 核心功能对照

```python
# Claude Code TypeScript
const tools = [
  { name: "Read", description: "..." },
  { name: "Edit", description: "..." },
  { name: "Bash", description: "..." }
]

# claw-code Python (示意)
class Tool:
    name: str
    description: str
    execute: Callable

tools = [
    Tool(name="Read", description="...", execute=read_file),
    Tool(name="Edit", description="...", execute=edit_file),
    Tool(name="Bash", description="...", execute=run_bash)
]
```

---

## 4. Parity 审计

### 4.1 什么是 Parity

claw-code 提供了 `parity-audit` 命令来追踪与原始 Claude Code 的功能对齐程度：

```bash
python -m claw_code parity-audit
```

### 4.2 审计维度

| 维度 | 说明 |
|-----|------|
| **Command Parity** | CLI 命令覆盖 |
| **Tool Parity** | 工具函数覆盖 |
| **Behavior Parity** | 行为一致性 |
| **API Parity** | 编程接口 |

---

## 5. 学习价值

### 5.1 适合学习什么

1. **Claude Code 架构**: 通过 Python 代码理解原始设计
2. **Tool 系统**: Python 风格的 Tool 实现
3. **CLI 设计**: Python 命令行应用模式
4. **Agent 循环**: Python 异步编程实践

### 5.2 与 Claude Code 的差异

| 方面 | Claude Code | claw-code |
|-----|------------|-----------|
| **源码** | TypeScript | Python |
| **来源** | 官方 | 逆向工程 |
| **功能** | 完整 | 逐步对齐中 |
| **支持** | Anthropic | 社区维护 |

---

## 6. 使用建议

### 6.1 何时使用 claw-code

- ✅ 需要与 Python 项目深度集成
- ✅ 想学习 Agent 架构但不懂 TypeScript
- ✅ 需要私有化部署的 Agent

### 6.2 何时仍用 Claude Code

- ✅ 需要最新 Anthropic 模型能力
- ✅ 需要官方支持和稳定性
- ✅ 生产环境关键应用

---

## 相关章节

- [[16-01-🚀-OpenCode与开源Agent指南]] - 返回总览
- [[../01-架构总览/01-01-📐-架构概览]] - Claude Code 架构
- [[../02-Tool系统/02-01-🔧-Tool系统]] - Tool 系统

---

> [!cite]- 知识来源
>
> | 知识点 | 来源 |
> |-------|------|
> | **claw-code 源码** | [github.com/instructkr/claw-code](https://github.com/instructkr/claw-code) |
> | **项目介绍** | 项目 README |
> | **Stars 数据** | GitHub Trending 2026-03 |
