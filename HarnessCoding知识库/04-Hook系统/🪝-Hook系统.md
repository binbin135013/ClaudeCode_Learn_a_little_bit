---
type: system
tags: [claude-code, hook, lifecycle, advanced]
description: Claude Code Hook 系统五类26事件（进阶内容）
---

# 🪝 Hook 系统

> ⏱ 难度: ★★★ | 重要性: ★★☆ | 推荐学习时间: 2-3天
> ⚠️ **新手提示**: 这是进阶内容！建议先学完 Tool系统 和 权限系统 再来。

---

## 概述

Hook 是 Claude Code 的**生命周期扩展机制**，允许你在关键节点拦截和修改行为。

### 什么时候会用到 Hook？

| 场景 | 示例 |
|-----|------|
| 🔌 拦截敏感操作 | 删除文件前确认 |
| 📝 修改 AI 提示 | 自动注入项目上下文 |
| 📊 记录操作日志 | 审计谁做了什么 |
| 🔄 触发外部系统 | Slack 通知、CI/CD 触发 |

---

## 五类 Hook

| 类型 | 用途 | 触发时机 |
|-----|------|---------|
| **Command** | CLI 命令拦截 | 命令执行前后 |
| **Prompt** | Prompt 修改 | AI 请求前 |
| **Agent** | Agent 生命周期 | Agent 启动/结束 |
| **HTTP** | HTTP 钩子 | 请求/响应 |
| **Function** | 自定义函数 | 自定义事件 |

---

## 26 个生命周期事件

### 工具调用事件

```
PreToolUse        - 工具调用前
PostToolUse       - 工具调用后
PostToolUseFailure - 工具调用失败
```

### 用户交互事件

```
UserPromptSubmit  - 用户消息提交
Notification      - 通知
```

### 会话事件

```
SessionStart      - 会话开始
SessionEnd        - 会话结束
Stop              - 停止
StopFailure       - 停止失败
```

### 子Agent事件

```
SubagentStart     - 子代理启动
SubagentStop      - 子代理停止
```

### 压缩事件

```
PreCompact        - 压缩前
PostCompact       - 压缩后
```

---

## Hook 响应协议

```typescript
interface HookResponse {
  status: "approve" | "block" | "modify";
  updatedInput?: unknown;      // 修改后的输入
  additionalContext?: string; // 注入的上下文
  reason?: string;            // 决策原因
}
```

### 响应示例

```typescript
// 拦截危险命令
const blockDangerousBash: Hook = {
  event: "PreToolUse",
  filter: { tool: "Bash", command: /rm\s+-rf/i },
  async handle(context) {
    return {
      status: "block",
      reason: "危险命令已拦截"
    };
  }
};

// 修改 Prompt
const injectContext: Hook = {
  event: "UserPromptSubmit",
  async handle(context) {
    return {
      status: "modify",
      updatedInput: context.input + "\\n\\n[项目上下文已自动注入]",
      additionalContext: "用户当前在 src/feature 分支"
    };
  }
};
```

---

## 三层安全架构

```
┌─────────────────────────────────┐
│ 1. 全局开关 - 总启用/禁用        │
├─────────────────────────────────┤
│ 2. 企业控制 - 组织级别策略       │
├─────────────────────────────────┤
│ 3. 工作区信任 - 每个工作区配置   │
└─────────────────────────────────┘
```

---

## 六层优先级

Hook 执行顺序由优先级决定：

| 优先级 | 类型 | 说明 |
|-------|------|------|
| 1 | System Hooks | 内置系统钩子 |
| 2 | Enterprise Hooks | 企业级钩子 |
| 3 | Workspace Hooks | 工作区钩子 |
| 4 | Project Hooks | 项目级钩子 |
| 5 | User Hooks | 用户级钩子 |
| 6 | Session Hooks | 会话级钩子 |

---

## Hook vs Skill

| 维度 | Hook | Skill |
|-----|------|-----|
| 触发方式 | 生命周期事件自动 | 用户显式调用 `/skill` |
| 目的 | 修改/拦截系统行为 | 扩展用户能力 |
| 编写难度 | 需要理解内部机制 | 相对简单 |
| 执行时机 | 透明嵌入 | 显式调用 |
| 数量 | 5类, 26事件 | 11个内置 + 可自定义 |

---

## 实战：创建一个 Hook

### 场景：自动记录所有文件操作

```javascript
// ~/.claude/hooks/audit-log.js
export const auditLogHook = {
  event: "PostToolUse",
  filter: { tool: ["Write", "Edit", "Rm"] },

  async handle(context) {
    const { tool, input, result, timestamp } = context;

    await logToFile({
      action: tool,
      target: input.file_path || input.path,
      timestamp,
      user: getCurrentUser()
    });

    return { status: "approve" };
  }
};
```

### 注册 Hook

```javascript
// ~/.claude/settings.json
{
  "hooks": {
    "enabled": true,
    "items": [
      "./hooks/audit-log.js"
    ]
  }
}
```

---

## 常见 Hook 用例

### 1. 自动注入项目上下文

```javascript
const projectContextHook = {
  event: "UserPromptSubmit",
  async handle(context) {
    const packageJson = await readFile("package.json");
    const projectInfo = parsePackageJson(packageJson);

    return {
      status: "modify",
      additionalContext: `当前项目: ${projectInfo.name}\\n版本: ${projectInfo.version}`
    };
  }
};
```

### 2. 危险命令二次确认

```javascript
const dangerousCommandHook = {
  event: "PreToolUse",
  filter: { tool: "Bash", pattern: /rm\s+-rf|drop\s+table/i },
  async handle(context) {
    const confirmed = await askUser(
      `⚠️ 危险命令: ${context.command}\\n确定要执行吗？`
    );
    return { status: confirmed ? "approve" : "block" };
  }
};
```

### 3. Slack 通知

```javascript
const slackNotifyHook = {
  event: "PostToolUse",
  filter: { tool: "GitCommit" },
  async handle(context) {
    await fetch(SLACK_WEBHOOK, {
      method: "POST",
      body: JSON.stringify({
        text: `新提交: ${context.result.commitHash}\\n${context.result.message}`
      })
    });
  }
};
```

---

## 8. 五种钩子类型详解

| 类型 | 执行引擎 | 可持久化 | 延迟 | 典型场景 |
|-----|---------|---------|------|---------|
| **Command** | Shell | 是 | 毫秒级 | 脚本检查 |
| **Prompt** | LLM | 是 | 秒级 | 内容审核 |
| **Agent** | LLM多步 | 是 | 秒~分钟 | 测试验证 |
| **HTTP** | HTTP请求 | 是 | 网络依赖 | CI集成 |
| **Function** | TS回调 | 否 | 毫秒级 | 运行时拦截 |

## 9. 26个生命周期事件完整列表

```
工具调用层:
  - PreToolUse        (工具调用前)
  - PostToolUse       (工具调用后)
  - PostToolUseFailure (工具调用失败)

用户交互层:
  - UserPromptSubmit  (用户消息提交)
  - Notification      (通知)

会话管理层:
  - SessionStart     (会话开始)
  - SessionEnd       (会话结束)
  - Stop             (停止)
  - StopFailure      (停止失败)

子代理层:
  - SubagentStart    (子代理启动)
  - SubagentStop     (子代理停止)

压缩层:
  - PreCompact       (压缩前)
  - PostCompact      (压缩后)

权限层:
  - PermissionRequest (权限请求)
  - PermissionDenied  (权限拒绝)

其他:
  - ConfigChange     (配置变更)
  - CwdChanged      (目录变更)
  - FileChanged     (文件变更)
  - Setup          (初始化)
  - Elicitation    (请求澄清)
  - InstructionsLoaded (指令加载)
```

## 10. 结构化JSON响应协议

```typescript
interface HookResponse {
  decision: "approve" | "block";
  reason?: string;              // 阻止原因
  updatedInput?: unknown;       // 修改工具输入
  additionalContext?: string;   // 注入额外上下文
  continue?: boolean;           // 控制助手是否继续生成
}

// 退出码协作
// 0 = approve, 2 = block, 其他 = error
```

## 11. 钩子优先级（6层）

```
userSettings > projectSettings > localSettings > pluginHook > builtinHook > sessionHook
```

## 相关章节

- [[../08-Skill系统/📚-Skill系统]] - Skill 基于 Hook 构建
- [[../11-QueryEngine/⚙️-QueryEngine]] - Hook 嵌入引擎
- [[../10-设计模式/♻️-核心设计模式]] - 拦截器模式
- [[../02-Tool系统/🔧-Tool系统]] - 被 Hook 拦截的工具
- [[🏠-知识库首页]] - 返回首页

---

> [!cite]- 知识来源
>
> 本文档核心内容来源：
>
> | 知识点 | 来源 |
> |-------|------|
> | **五类 Hook**（Command/Prompt/Agent/HTTP/Function） | [Claude Code 官方文档](https://docs.anthropic.com/zh-CN/docs/claude-code/hooks) |
> | **26 个生命周期事件** | [lintsinghua/claude-code-book](https://github.com/lintsinghua/claude-code-book)（主教材，1,190 stars） |
> | **三层安全架构** | [Claude Code 官方企业文档](https://docs.anthropic.com/zh-CN/docs/claude-code/hooks#hook-安全) |
> | **HookResponse 协议** | [liuup/claude-code-analysis](https://github.com/liuup/claude-code-analysis)（537 stars，源码分析） |
> | **六层优先级** | [claude-code-best/claude-code](https://github.com/claude-code-best/claude-code)（8,712 stars，企业级参考） |
