---
type: best-practice
tags: [claude-code, hook, customization, configuration]
description: Claude Code 自定义 Hook 配置指南
---

# 自定义 Hook 配置指南

> ⏱ 难度: ★★☆ | 重要性: ★★★ | 推荐学习时间: 1-2天

---

## 1. Hook 概述

### 1.1 什么是 Hook

Hook 是 Claude Code 的**生命周期扩展机制**，允许在关键节点拦截和修改系统行为。

### 1.2 五类 Hook

| 类型 | 用途 | 触发时机 |
|-----|------|---------|
| **Command** | CLI 命令拦截 | 命令执行前后 |
| **Prompt** | Prompt 修改 | AI 请求前 |
| **Agent** | Agent 生命周期 | Agent 启动/结束 |
| **HTTP** | HTTP 钩子 | 请求/响应 |
| **Function** | 自定义函数 | 自定义事件 |

---

## 2. 推荐的 Hook 列表

### 2.1 安全类 Hook

```javascript
// ~/.claude/hooks/security.js

// 危险命令拦截
export const blockDangerousCommands = {
  event: "PreToolUse",
  filter: { tool: "Bash", pattern: /rm\s+-rf|drop\s+table|shutdown/i },
  async handle(context) {
    return {
      status: "block",
      reason: "危险命令已拦截，请确认操作"
    };
  }
};

// 敏感文件访问保护
export const protectSensitiveFiles = {
  event: "PreToolUse",
  filter: { tool: ["Read", "Edit"], pattern: /(\.env|password|secret|key)/i },
  async handle(context) {
    return {
      status: "block",
      reason: "敏感文件访问已阻止"
    };
  }
};
```

### 2.2 上下文注入 Hook

```javascript
// ~/.claude/hooks/context-inject.js

// 自动注入项目上下文
export const injectProjectContext = {
  event: "UserPromptSubmit",
  async handle(context) {
    const projectInfo = await getProjectInfo();
    return {
      status: "modify",
      additionalContext: `项目: ${projectInfo.name}\n版本: ${projectInfo.version}\n分支: ${projectInfo.branch}`
    };
  }
};

// 注入当前工作目录信息
export const injectCwdContext = {
  event: "SessionStart",
  async handle(context) {
    return {
      status: "modify",
      additionalContext: `当前目录: ${process.cwd()}`
    };
  }
};
```

### 2.3 日志审计 Hook

```javascript
// ~/.claude/hooks/audit.js

// 操作日志记录
export const operationAuditLog = {
  event: "PostToolUse",
  filter: { tool: ["Write", "Edit", "Rm", "Bash"] },
  async handle(context) {
    await logOperation({
      tool: context.tool,
      input: context.input,
      user: context.user,
      timestamp: new Date().toISOString()
    });
    return { status: "approve" };
  }
};

// Git 操作审计
export const gitOperationAudit = {
  event: "PostToolUse",
  filter: { tool: "Bash", pattern: /^git\s+(commit|push|merge)/i },
  async handle(context) {
    await notifySlack(`Git 操作: ${context.input}`);
    return { status: "approve" };
  }
};
```

### 2.4 质量检查 Hook

```javascript
// ~/.claude/hooks/quality.js

// 提交前检查
export const preCommitCheck = {
  event: "PreToolUse",
  filter: { tool: "Bash", pattern: /git\s+commit/i },
  async handle(context) {
    const hasLintErrors = await runLinter();
    if (hasLintErrors) {
      return {
        status: "block",
        reason: "代码检查未通过，请先修复 lint 错误"
      };
    }
    return { status: "approve" };
  }
};

// 测试通过检查
export const prePushTestCheck = {
  event: "PreToolUse",
  filter: { tool: "Bash", pattern: /git\s+push/i },
  async handle(context) {
    const testsPassed = await runTests();
    if (!testsPassed) {
      return {
        status: "block",
        reason: "测试未通过，请先修复测试"
      };
    }
    return { status: "approve" };
  }
};
```

---

## 3. 自定义 Hook 开发指南

### 3.1 Hook 文件结构

```
~/.claude/
├── hooks/
│   ├── index.js          # 主入口（导出所有 Hook）
│   ├── security.js       # 安全相关 Hook
│   ├── context.js        # 上下文注入 Hook
│   ├── audit.js          # 审计日志 Hook
│   └── quality.js        # 质量检查 Hook
└── settings.json        # Hook 配置
```

### 3.2 Hook 定义格式

```javascript
// hook 文件格式
export const hookName = {
  event: "EventName",           // 事件类型
  filter: {                      // 可选过滤器
    tool: "ToolName",           // 工具名称
    pattern: /regex/i            // 正则匹配
  },
  async handle(context) {
    // 处理逻辑
    return {
      status: "approve" | "block" | "modify",
      reason: "原因说明",
      updatedInput: context.input,
      additionalContext: "额外上下文"
    };
  }
};
```

### 3.3 事件类型完整列表

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
  - PostCompact     (压缩后)

权限层:
  - PermissionRequest (权限请求)
  - PermissionDenied  (权限拒绝)
```

---

## 4. Hook 配置示例

### 4.1 settings.json 配置

```json
// ~/.claude/settings.json
{
  "hooks": {
    "enabled": true,
    "items": [
      "./hooks/security.js",
      "./hooks/context-inject.js",
      "./hooks/audit.js",
      "./hooks/quality.js"
    ]
  }
}
```

### 4.2 完整 Hook 示例

```javascript
// ~/.claude/hooks/complete-example.js

// 多工具拦截
export const multiToolInterceptor = {
  event: "PreToolUse",
  filter: {
    tool: ["Bash", "Write", "Rm"],
    pattern: /production|dangerous|critical/i
  },
  async handle(context) {
    const isConfirmed = await confirmUser(
      `⚠️ 危险操作: ${context.tool}\n命令: ${context.input}\n确定继续吗？`
    );
    return { status: isConfirmed ? "approve" : "block" };
  }
};

// Prompt 修改示例
export const promptEnhancer = {
  event: "UserPromptSubmit",
  async handle(context) {
    // 添加代码规范上下文
    const styleGuide = await getStyleGuide();
    return {
      status: "modify",
      updatedInput: context.input,
      additionalContext: `\n\n代码规范提示: ${styleGuide}`
    };
  }
};

// 响应拦截示例
export const responseLogger = {
  event: "PostToolUse",
  async handle(context) {
    console.log(`[Hook Log] ${context.tool} - ${JSON.stringify(context.result)}`);
    return { status: "approve" };
  }
};
```

---

## 5. 常见 Hook 场景

### 5.1 开发环境场景

| 场景 | 推荐 Hook |
|------|----------|
| 防止误删文件 | `PreToolUse` 拦截 `rm` 命令 |
| 提交前检查 | `PreToolUse` 拦截 `git commit` |
| 敏感信息检查 | `PreToolUse` 拦截敏感文件访问 |
| 自动生成文档 | `PostToolUse` 触发文档生成 |

### 5.2 团队协作场景

| 场景 | 推荐 Hook |
|------|----------|
| 代码审查通知 | `PostToolUse` 触发 PR 评论 |
| 部署前验证 | `PreToolUse` 拦截生产部署 |
| 操作审计日志 | `PostToolUse` 记录所有操作 |
| 变更通知 | `PostToolUse` 发送团队通知 |

### 5.3 个人效率场景

| 场景 | 推荐 Hook |
|------|----------|
| 自动上下文注入 | `SessionStart` 注入项目信息 |
| 常用命令快捷方式 | `UserPromptSubmit` 触发替换 |
| 学习笔记自动生成 | `PostToolUse` 记录学习内容 |
| 定时提醒 | `Notification` 定时触发 |

---

## 6. Hook 调试技巧

### 6.1 本地调试

```javascript
// 添加详细日志
export const debugHook = {
  event: "PreToolUse",
  async handle(context) {
    console.log("=== Hook Debug ===");
    console.log("Tool:", context.tool);
    console.log("Input:", JSON.stringify(context.input));
    console.log("==================");
    return { status: "approve" };
  }
};
```

### 6.2 逐步启用

```javascript
// 先只启用日志
export const safeHook = {
  event: "PostToolUse",
  async handle(context) {
    console.log(`[SAFE] ${context.tool} called`);
    return { status: "approve" };  // 先只记录
  }
};

// 确认无误后改为 block
export const productionHook = {
  event: "PreToolUse",
  filter: { tool: "Bash", pattern: /rm\s+-rf\s+\//i },
  async handle(context) {
    return { status: "block", reason: "危险命令" };
  }
};
```

---

## 7. 最佳实践

### 7.1 安全建议

- 生产环境使用 `block` 需谨慎
- 重要操作前先 `confirmUser`
- 保持 Hook 逻辑简洁
- 定期审查 Hook 日志

### 7.2 性能建议

- 避免在 Hook 中执行耗时操作
- 使用 `filter` 减少触发频率
- 异步处理非关键逻辑

### 7.3 组织建议

```
hooks/
├── index.js         # 汇总导出
├── security/        # 分类目录
│   ├── dangerous.js
│   └── sensitive.js
├── context/         # 上下文相关
├── audit/           # 审计相关
└── quality/         # 质量相关
```

---

## 相关章节

- [[⚡-ClaudeCode插件推荐]] - 插件生态
- [[🎯-自定义Skill开发]] - Skill 开发
- [[../04-Hook系统/🪝-Hook系统]] - Hook 系统详解
- [[🏗️-整体架构建议]] - 项目结构

---

> [!cite]- 知识来源
>
> | 知识点 | 来源 |
> |-------|------|
> | **Hook 系统** | [Claude Code 官方文档](https://docs.anthropic.com/zh-CN/docs/claude-code/hooks) |
> | **Hook 事件列表** | [lintsinghua/claude-code-book](https://github.com/lintsinghua/claude-code-book) |
> | **企业级 Hook** | [claude-code-best/claude-code](https://github.com/claude-code-best/claude-code) |
