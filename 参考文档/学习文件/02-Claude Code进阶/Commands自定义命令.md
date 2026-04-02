---
tags: [Claude-Code进阶, Commands, 命令]
aliases: [Commands, 自定义命令, Slash命令, 斜杠命令]
date: 2026-04-02
related: "[[Claude Code]], [[技能系统]], [[Skills开发]]"
---

# Commands 自定义命令

> [!summary] 一句话定义
> 通过 `.claude/commands/` 目录创建 `/slash` 命令，一键触发预定义的复杂工作流。

---

## 命令类型

| 类型 | 位置 | 说明 |
|------|------|------|
| **内置命令** | Claude Code 自带 | `/help`, `/clear`, `/exit`, `/voice` 等 |
| **项目命令** | `.claude/commands/` | 团队共享，入库管理 |
| **个人命令** | `~/.claude/commands/` | 个人偏好，跨项目可用 |

## 内置命令速查（v2.1.78）

| 命令 | 功能 | 版本 |
|------|------|------|
| `/help` | 帮助信息 | 初始 |
| `/clear` | 清除对话 | 初始 |
| `/exit` | 退出 | 初始 |
| `/voice` | 语音模式 | v2.1.69+ |
| `/effort` | 推理深度控制 | v2.1.69+ |
| `/loop` | 定时循环任务 | v2.1.69+ |
| `/sandbox` | 沙箱隔离 | v2.1.69+ |
| `/fast` | 快速模式切换 | v2.1.52+ |
| `/tp` | 跨设备传送会话 | v2.1+ |

## 创建自定义命令

### 基本格式

在 `.claude/commands/` 下创建 Markdown 文件：

```markdown
name: review
description: 代码审查命令

# 代码审查

请对以下代码变更进行审查：
1. 代码质量和可维护性
2. 潜在安全问题
3. 性能考量
4. 测试覆盖建议
```

使用：`/review` 即可触发

### 带参数的命令

```markdown
name: fix-issue
description: 修复 GitHub Issue

# 修复 Issue #$ARGUMENTS

请分析并修复 GitHub Issue #$ARGUMENTS 中的问题。
```

使用：`/fix-issue 123` → 自动填充 Issue 编号

## 命名规范

> [!tip] 推荐命名方案
> ```
> 00-09：基础设施（help, setup）
> 10-19：开发（dev, build）
> 20-29：测试（test, lint）
> 30-39：部署（deploy, release）
> 40-49：数据（migrate, seed）
> 90-99：工具（debug, monitor）
> ```

## 实战：团队共享命令集

```
.claude/commands/
├── 10-dev.md          # 开发相关
├── 20-test.md         # 测试相关
├── 30-deploy.md       # 部署相关
├── review.md          # 代码审查
├── security-review.md # 安全审查
└── fix-issue.md       # Issue 修复
```

> [!warning] 注意
> 命令文件必须入库（`.gitignore` 不要忽略 `.claude/commands/`），团队才能共享使用。

---

## 进阶：命令设计模式

### 命令 vs Skills 选择

> [!note] 什么时候用 Command，什么时候用 Skill？
> - **Command** — 用户主动触发的快捷操作，轻量级
> - **Skill** — 可复用能力包，支持独立上下文和热重载
> - **Hook** — 自动触发的后台操作，不需要用户干预

| 维度 | Command | Skill | Hook |
|------|---------|-------|------|
| 触发方式 | 用户 `/command` | 用户调用或自动加载 | 事件自动触发 |
| 复杂度 | 简单指令 | 可包含多文件 | 单一脚本 |
| 上下文 | 主会话内 | 可独立(fork) | 独立进程 |
| 适用场景 | 快捷操作 | 领域专家能力 | 自动化 |

### 实用命令模板

**快速审查命令** — `.claude/commands/review.md`

```markdown
请审查以下文件的代码质量，重点关注：
1. 安全漏洞
2. 性能问题
3. 代码规范

文件：$ARGUMENTS
```

**批量重构命令** — `.claude/commands/refactor.md`

```markdown
对 $ARGUMENTS 执行重构，遵循以下原则：
1. 保持行为不变
2. 提高可读性
3. 减少重复代码
先输出重构计划，等确认后再执行。
```

**测试生成命令** — `.claude/commands/test.md`

```markdown
为 $ARGUMENTS 生成测试用例，要求：
1. 覆盖正常路径和边界情况
2. Mock 外部依赖
3. 测试命名清晰
使用项目现有的测试框架。
```

### 命令命名规范

| 规范 | 示例 | 说明 |
|------|------|------|
| 小写连字符 | `code-review` | 不要用 camelCase |
| 动词开头 | `fix-lint` | 明确行为 |
| 简短有意义 | `test` | 不超过 3 个词 |
| 避免缩写 | `refactor` 非 `rfc` | 可读性优先 |

---

**相关笔记**：[[Claude Code]] · [[技能系统]] · [[Skills开发]] · [[CI-CD与自动化]] · [[常见陷阱与反模式]]
