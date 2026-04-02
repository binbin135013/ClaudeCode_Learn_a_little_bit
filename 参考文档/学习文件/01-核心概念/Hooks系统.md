---
tags: [核心概念, Hooks, 自动化]
aliases: [Hooks, 钩子系统, 事件触发]
date: 2026-04-02
related: "[[Claude Code]], [[MCP协议]], [[CI-CD与自动化]], [[技能系统]]"
---

# Hooks 系统

> [!summary] 一句话定义
> **事件驱动的自动化脚本**，在 Claude Code 执行操作的前后触发自定义命令，类似 Git Hooks。

---

## Hook 事件类型完整列表

### 工具相关（最常用）

| Hook 类型 | 触发时机 | 典型用途 |
|-----------|---------|---------|
| `PreToolUse` | 工具执行**前** | 拦截危险操作、注入上下文 |
| `PostToolUse` | 工具执行**后** | 日志记录、结果后处理 |
| `StopFailure` | 工具**失败**时 | 错误通知、重试逻辑 |

### 用户交互

| Hook 类型 | 触发时机 | 典型用途 |
|-----------|---------|---------|
| `UserPromptSubmit` | 用户输入**提交**时 | 输入验证、自动格式化提示词、注入上下文 |
| `Elicitation` | AI **需要用户输入**时 | 自定义输入处理、表单校验 |

### 会话相关

| Hook 类型 | 触发时机 | 典型用途 |
|-----------|---------|---------|
| `Notification` | AI 回复完成时 | 外部通知推送 |
| `PostCompact` | 上下文压缩后 | 恢复重要上下文 |
| `InstructionsLoaded` | 指令加载后 | 动态修改指令 |

### 生命周期

| Hook 类型 | 触发时机 | 典型用途 |
|-----------|---------|---------|
| `SessionStart` | 会话开始 | 环境检查、欢迎信息 |
| `Stop` | AI **停止响应**时 | 保存状态、清理资源 |
| `SubagentStart` | 子代理启动 | 注入上下文 |
| `SubagentStop` | 子代理**结束**时 | 结果收集、状态同步 |

### Worktree（工作树）

| Hook 类型 | 触发时机 | 典型用途 |
|-----------|---------|---------|
| `WorktreeCreate` | 工作树**创建**时 | 环境初始化、依赖安装 |
| `WorktreeRemove` | 工作树**删除**时 | 清理临时文件、释放资源 |

### HTTP Hooks（远程 Webhook）

| Hook 类型 | 触发时机 | 典型用途 |
|-----------|---------|---------|
| `PreToolUse` (HTTP) | 工具执行前 | 远程审批、审计 |
| `PostToolUse` (HTTP) | 工具执行后 | 远程日志、通知 |

## 配置方式

在 `.claude/settings.json` 中配置：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "echo '[拦截] $(date): $CLAUDE_TOOL_NAME' >> ./logs/hooks.log"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write $CLAUDE_FILE_PATHS"
          }
        ]
      }
    ],
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "curl -X POST https://hooks.slack.com/xxx -d '{\"text\":\"Claude Code 任务完成\"}'"
          }
        ]
      }
    ]
  }
}
```

## 实战场景

> [!example] 场景 1：自动格式化代码
> PostToolUse + Write → 自动运行 Prettier 格式化

> [!example] 场景 2：Git 提交检查
> PreToolUse + Bash(git commit) → 检查 commit message 格式

> [!example] 场景 3：Webhook 通知
> Notification + HTTP Hook → Slack/钉钉通知

> [!example] 场景 4：MCP 调用审计
> PreToolUse + mcp__* matcher → 记录所有 MCP 调用

## 数据传递机制

> [!important] Hooks 通过 stdin JSON 接收数据
> Hook 脚本通过 stdin 接收 JSON 格式的数据，而非环境变量。

| 字段 | 说明 |
|------|------|
| `tool_name` | 工具名称（如 "Edit", "Bash"）|
| `tool_input` | 工具输入参数（JSON 对象）|
| `file_paths` | 文件路径列表 |
| `session_id` | 会话 ID |
| `exit_code` | 退出码（仅 PostToolUse）|

环境变量：仅 `$CLAUDE_PROJECT_DIR` 可用。

> [!warning] 安全注意
> Hooks 执行任意命令，**只用信任的脚本**！避免在 Hooks 中处理敏感数据。

---

**相关笔记**：[[Claude Code]] · [[CI-CD与自动化]] · [[MCP协议]] · [[权限与沙箱]]
