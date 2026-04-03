---
type: system
tags: [superpowers, claude-code, skill, workflow]
description: Superpowers 入门指南 - 掌握 Claude Code 的高级开发能力
---

# 🚀 Superpowers 入门

## 1. Superpowers 简介

### 1.1 什么是 Superpowers

Superpowers 是 Claude Code 的**高级 Skill 集合**，提供一套经过验证的开发方法论和最佳实践。它将复杂的开发流程封装为可复用的技能模块，帮助开发者快速完成高质量的软件开发。

Superpowers 不是单独的工具，而是 **7 个核心 Skill 的组合**：

| Skill | 用途 | 触发词 |
|-------|------|--------|
| `brainstorming` | 头脑风暴与设计 | "brainstorm" |
| `systematic-debugging` | 四阶段调试法 | "systematic-debugging" |
| `test-driven-development` | TDD 红绿重构 | "tdd" |
| `subagent-driven-development` | 子代理驱动开发 | "subagent" |
| `receiving-code-review` | 接收代码审查 | "receive-review" |
| `requesting-code-review` | 请求代码审查 | "request-review" |
| `writing-skills` | Skill 创作方法论 | "write-skills" |

### 1.2 Superpowers 与内置 Skill 的区别

| 维度 | 内置 Skill | Superpowers |
|-----|-----------|-------------|
| **定位** | 通用能力封装 | 专业开发方法论 |
| **复杂度** | 相对简单 | 需要理解流程 |
| **组合性** | 独立使用 | 可链式组合 |
| **适用场景** | 日常任务 | 复杂工程 |

### 1.3 核心理念

Superpowers 遵循两个核心原则：

1. **HARD-GATE 机制**：强制流程执行，确保每个阶段都完成后再进入下一阶段
2. **"Even 1% chance means invoke" 原则**：只要有可能出现问题，就应该调用对应的 Skill

---

## 2. 安装与配置

### 2.1 前置要求

- Claude Code 已安装并配置
- 了解基本命令和 Tool 使用方法
- 已阅读 [[../08-Skill系统/📚-Skill系统]] 章节

### 2.2 安装 Superpowers

Superpowers 作为 Claude Code 的 Skill 集合，无需额外安装。只要在支持的插件市场中获取，即可自动注册。

### 2.3 验证安装

```bash
/skill superpowers
```

或直接使用触发词：

```bash
brainstorm
tdd
systematic-debugging
```

---

## 3. 核心 Skill 一览

### 3.1 brainstorming - 头脑风暴与设计

**触发词**: `brainstorm`

**功能**: 在开始编码前，进行系统性的头脑风暴，明确需求和设计方案。

**使用场景**:
- 新项目启动
- 功能设计阶段
- 技术方案选型

**工作流程**:
1. 问题定义
2. 需求分析
3. 方案探索
4. 方案评估
5. 决策总结

### 3.2 systematic-debugging - 四阶段调试法

**触发词**: `systematic-debugging`

**功能**: 结构化的调试方法论，快速定位和修复问题。

**使用场景**:
- Bug 修复
- 错误分析
- 问题溯源

**工作流程**:
1. **复现** - 稳定复现问题
2. **定位** - 缩小范围找到根因
3. **修复** - 实施解决方案
4. **验证** - 确认问题已解决

### 3.3 test-driven-development - TDD 红绿重构

**触发词**: `tdd`

**功能**: 测试驱动的开发方法论，确保代码质量。

**使用场景**:
- 新功能开发
- 代码重构
- 关键模块实现

**工作流程**:
1. **红** - 编写一个会失败的测试
2. **绿** - 编写最少的代码使测试通过
3. **重构** - 改善代码结构

### 3.4 subagent-driven-development - 子代理驱动开发

**触发词**: `subagent`

**功能**: 利用子代理并行处理复杂任务，提高开发效率。

**使用场景**:
- 大型项目开发
- 并行任务处理
- 多模块同步开发

**工作流程**:
1. 任务分解
2. 子代理分配
3. 并行执行
4. 结果聚合

### 3.5 receiving-code-review - 接收代码审查

**触发词**: `receive-review`

**功能**: 系统化地接收和处理代码审查反馈。

**使用场景**:
- 审查反馈处理
- 代码质量改进
- 团队协作

**工作流程**:
1. 接受反馈
2. 分类处理
3. 逐项修复
4. 确认完成

### 3.6 requesting-code-review - 请求代码审查

**触发词**: `request-review`

**功能**: 规范化地向他人请求代码审查。

**使用场景**:
- 代码提交前
- Pull Request 创建
- 团队协作

**工作流程**:
1. 自我检查
2. 准备审查材料
3. 发起审查请求
4. 处理反馈

### 3.7 writing-skills - Skill 创作方法论

**触发词**: `write-skills`

**功能**: 创建自定义 Skill 的方法论指导。

**使用场景**:
- 创建新 Skill
- 封装工作流
- 团队 Skill 共享

---

## 4. 快速上手

### 4.1 第一个任务：使用 TDD 开发功能

**目标**: 用 TDD 方法实现一个字符串工具函数

**步骤**:

1. **启动 TDD**
   ```bash
   tdd 实现一个字符串工具库
   ```

2. **编写测试（红）**
   - 先写测试用例
   - 定义预期行为

3. **实现代码（绿）**
   - 编写最小代码
   - 让测试通过

4. **重构**
   - 改善代码质量
   - 保持测试通过

### 4.2 调试任务：使用 systematic-debugging

**目标**: 修复一个 Bug

**步骤**:

1. **启动调试**
   ```bash
   systematic-debugging
   ```

2. **复现问题**
   - 确定复现步骤
   - 验证问题存在

3. **定位根因**
   - 收集信息
   - 缩小范围
   - 找到根本原因

4. **修复问题**
   - 实施解决方案

5. **验证修复**
   - 确认问题解决
   - 确保无回归

### 4.3 头脑风暴：使用 brainstorming

**目标**: 设计一个新功能

**步骤**:

1. **启动头脑风暴**
   ```bash
   brainstorm 设计用户认证模块
   ```

2. **探索方案**
   - 列出所有可能的方案
   - 分析每个方案的优缺点

3. **做出决策**
   - 选择最佳方案
   - 明确实施计划

---

## 5. Superpowers 方法论核心

### 5.1 HARD-GATE 机制

HARD-GATE 是 Superpowers 的核心执行机制：

- **H**alt - 在每个阶段门口停止
- **A**ssert - 断言前置条件满足
- **R**eview - 审查本阶段成果
- **D**ecide - 决定是否可以进入下一阶段
- **G**uarantee - 保证流程完整性
- **A**utomate - 自动化执行检查
- **T**race - 追踪每个决策
- **E**nsure - 确保结果质量

### 5.2 "Even 1% chance means invoke" 原则

只要有任何可能出现问题的可能性，就应该调用对应的 Skill：

- 有 Bug 风险 → `systematic-debugging`
- 新功能开发 → `tdd`
- 复杂设计 → `brainstorming`
- 大型任务 → `subagent-driven-development`

---

## 6. Skill 组合使用

Superpowers 的强大之处在于 Skill 可以组合使用：

```
brainstorm → tdd → systematic-debugging → receive-review
```

完整开发流程：
1. `brainstorming` - 明确需求和设计
2. `test-driven-development` - TDD 方式实现
3. `systematic-debugging` - 如遇问题则调试
4. `receiving-code-review` - 接受审查反馈

---

## 相关章节

- [[../08-Skill系统/📚-Skill系统]] - Skill 系统基础
- [[../09-子Agent与协作/🤝-子Agent与协作]] - 子代理协作
- [[../12-质量保证/🧪-质量保证]] - QA 方法论

---

*最后更新：2026-04-03*
