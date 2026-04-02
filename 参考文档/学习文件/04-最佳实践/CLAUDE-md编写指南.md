---
tags: [最佳实践, CLAUDE-md, 配置]
aliases: [CLAUDE.md, CLAUDE-md, 项目配置]
date: 2026-04-02
related: "[[Claude Code]], [[Skills开发]], [[项目工作流]], [[权限与沙箱]]"
---

# CLAUDE.md 编写指南

> [!summary] 一句话定义
> CLAUDE.md 是 Claude Code 的**项目级配置文件**，告诉 AI 你的项目规范、技术栈和特殊要求。写好它 = 写好给 AI 的"员工手册"。

---

## 三层配置

```
~/.claude/CLAUDE.md          ← 全局（个人偏好，所有项目共享）
项目根目录/CLAUDE.md          ← 项目（团队共享，入库管理）
子目录/CLAUDE.md              ← 模块级（特定规则）
```

## 精简原则

> [!warning] Token 成本
> CLAUDE.md 每次对话都会加载到上下文中！
> - **目标**：< 5000 字符（约 1000-1500 tokens）
> - **策略**：精炼列表 > 冗长段落

```markdown
# ❌ 冗长写法
我们的团队使用以下代码规范。首先，所有代码必须使用 TypeScript 编写。
其次，我们要求所有函数都有 JSDoc 注释。另外...

# ✅ 精简写法
## 代码规范
- 语言：TypeScript（严格模式）
- 注释：JSDoc 必需
- 命名：变量 camelCase，类 PascalCase
```

## 推荐模板

```markdown
# [项目名] - Claude Code 配置

## 项目概览
- 描述：[一句话]
- 技术栈：React 18 + TypeScript + Vite + Prisma
- 阶段：开发/测试/生产

## 代码规范
- 函数 < 50 行，类 < 300 行
- 禁止 `any` 类型
- 文件名 kebab-case，类名 PascalCase
- 使用中文注释，代码用英文

## 安全规则
- 禁止硬编码敏感信息
- 禁止提交 .env 文件
- API 必须有速率限制

## 测试要求
- 新功能必须有单元测试
- 核心逻辑覆盖率 > 80%

## Git 规范
- 分支：feature/xxx, fix/xxx, refactor/xxx
- 提交：`<type>(<scope>): <description>`

## 特殊说明
[项目特有规则]
```

## 高级技巧

> [!tip] 进阶写法
> 1. **引用外部文件** — `详见 docs/api-reference.md`，不在 CLAUDE.md 内联
> 2. **分层配置** — 根目录写通用规则，子目录写特殊规则
> 3. **动态指令** — 通过 [[Hooks系统]] 的 InstructionsLoaded 注入
> 4. **结合 Skills** — 复杂流程封装为 [[Skills开发]]，CLAUDE.md 只写核心规则

---

**相关笔记**：[[Claude Code]] · [[Skills开发]] · [[项目工作流]] · [[权限与沙箱]] · [[性能与成本优化]]
