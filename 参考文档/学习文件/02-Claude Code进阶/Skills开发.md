---
tags: [Claude-Code进阶, Skills, 开发]
aliases: [Skills开发, 技能包开发, SKILL-md]
date: 2026-04-02
related: "[[技能系统]], [[Commands自定义命令]], [[Claude Code]], [[自定义技能开发]]"
---

# Skills 开发

> [!summary] 一句话定义
> 创建 Claude Code 的**可复用技能包**，通过 `SKILL.md` 定义，支持独立上下文和热重载。

---

## SKILL.md 完整结构

```markdown
---
name: my-skill
description: 一句话描述技能功能
context: fork           # fork=独立子代理 | shared=主会话内
---

# 技能指令

这里是技能的核心指令内容...
```

### Frontmatter 字段

| 字段 | 必填 | 说明 |
|------|------|------|
| `name` | ✅ | 技能名称（小写+连字符）|
| `description` | ✅ | 一句话描述 |
| `context` | ❌ | `fork`（推荐）或 `shared` |

### context 模式

> [!note] 两种上下文模式
> - **fork**（推荐）— 在子代理中运行，不影响主会话上下文窗口
> - **shared** — 在主会话中运行，共享上下文

## 开发流程

```
1. 创建目录结构
   .claude/skills/my-skill/
   └── SKILL.md

2. 编写技能指令
   定义输入、处理逻辑、输出格式

3. 测试
   修改后 Hot Reload 自动重新加载

4. 使用 ${CLAUDE_SKILL_DIR}
   引用技能目录下的辅助文件
```

## 实战示例：代码审查技能

```markdown
---
name: code-review
description: 代码审查技能
context: fork
---

# 代码审查

## 审查维度
1. 代码质量：可读性、命名、重复代码
2. 安全性：输入验证、注入风险
3. 性能：N+1 查询、不必要计算
4. 测试：覆盖度、边界情况

## 输出格式
### 必须修改 (Blocking)
- [问题] 位置：[文件:行号] 建议：[修改方案]

### 建议修改 (Suggestion)
- [问题] 建议：[优化方案]
```

## 技能目录引用

```markdown
# 在 SKILL.md 中引用同目录文件
请参考 ${CLAUDE_SKILL_DIR}/checklist.md 中的审查清单
```

> [!tip] 进阶技巧
> 1. **组合技能** — 一个技能可以调用其他技能
> 2. **条件触发** — 通过 [[Hooks系统]] 在特定事件时自动激活技能
> 3. **技能市场** — 发布到 oh-my-claudecode 等社区共享

## 进阶：高级技能架构

### 多文件技能目录结构

简单的单文件技能适合快速原型，但生产级技能通常需要多个辅助文件。以下是一个代码审查技能的完整目录结构：

```
.claude/skills/advanced-review/
├── SKILL.md           # 主入口（每次调用加载）
├── checklists/
│   ├── security.md    # 安全审查清单
│   └── performance.md # 性能审查清单
├── templates/
│   └── review-output.md  # 输出模板
└── scripts/
    └── analyze.sh     # 辅助脚本
```

在 SKILL.md 中通过 `${CLAUDE_SKILL_DIR}` 引用这些文件：

```markdown
# 代码审查指令

请先阅读 ${CLAUDE_SKILL_DIR}/checklists/security.md 和
${CLAUDE_SKILL_DIR}/checklists/performance.md 中的审查清单。

审查完成后，按照 ${CLAUDE_SKILL_DIR}/templates/review-output.md
的格式输出报告。

如需静态分析，可执行 ${CLAUDE_SKILL_DIR}/scripts/analyze.sh。
```

> [!note] 设计原则
> SKILL.md 只放**指令逻辑**，具体数据（清单、模板、脚本）放在子目录中按需加载。这样 SKILL.md 保持精简，加载更快，维护也更方便。

### 技能组合模式

技能并非孤立运行，有三种常见的组合模式：

- **链式调用** -- skill A 的输出作为 skill B 的输入。例如 `code-review` 完成审查后，自动调用 `fix-issues` 修复发现的问题。在 SKILL.md 中直接写明"审查完成后，调用 /fix-issues 传入上述问题列表"即可。
- **条件激活** -- 通过 [[Hooks系统]] 在特定事件自动触发 skill。例如配置 `PostToolUse` hook，当 `Edit` 工具修改 `.ts` 文件后自动运行类型检查技能。参考 [[Hooks系统]] 中的 hook matcher 语法。
- **技能市场** -- oh-my-claudecode 社区共享机制。通过 `/oh-my-claudecode:skill` 管理本地技能的安装、搜索和更新。社区技能通常遵循标准目录结构，可以直接克隆到 `.claude/skills/` 下使用。

### context 模式选择决策树

选择 `fork` 还是 `shared` 是技能设计中的关键决策。用以下决策树判断：

```
你的技能需要？
├── 访问主会话上下文 → shared
│   ├── 例：基于当前对话历史做总结
│   ├── 例：根据前面的代码讨论继续生成
│   └── 注意：会占用主会话上下文窗口
│
└── 不需要主会话上下文 → fork（推荐）
    ├── 例：独立的代码审查
    ├── 例：文档生成
    ├── 例：测试运行与分析
    └── 优势：不影响主会话，失败也不会污染上下文
```

> [!warning] 慎用 shared
> `shared` 模式下技能的输出会直接进入主会话上下文窗口，大量输出会加速上下文消耗。除非明确需要访问对话历史，否则一律使用 `fork`。详见 [[Subagent子代理]] 中的上下文隔离机制。

### 性能优化

技能的性能直接影响日常使用体验，以下是三个关键优化点：

- **建议 SKILL.md 控制在合理长度内** -- SKILL.md 在每次技能调用时都会完整加载到上下文中，过长的指令会导致不必要的 token 消耗和响应延迟。源教程建议不超过 1000 行，将详细内容拆分到辅助文件中。
- **辅助文件用 `${CLAUDE_SKILL_DIR}` 按需引用** -- 不要把清单、模板、长示例内嵌到 SKILL.md 中。代理只在需要时才读取辅助文件，避免一次性加载过多无关内容。
- **避免在 SKILL.md 中内嵌大量示例数据** -- 如果需要示例（如审查报告样例），放在 `templates/` 或 `examples/` 子目录中，在指令中写"参考 `${CLAUDE_SKILL_DIR}/examples/sample-output.md`"。

> [!note] 更多 Frontmatter 字段
> SKILL.md 还支持以下 frontmatter 字段（非完整列表）：`model`、`agent`、`allowed-tools`、`user-invocable`、`disable-model-invocation`、`argument-hint`。完整字段请参考 Claude Code 官方文档。

---

**相关笔记**：[[技能系统]] · [[Commands自定义命令]] · [[Subagent子代理]] · [[自定义技能开发]]
