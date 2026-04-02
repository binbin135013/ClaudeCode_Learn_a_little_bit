---
tags: [Claude-Code进阶, Plugins, 插件, 生态]
aliases: [Plugins, 插件生态, Marketplace, oh-my-claudecode, OMC]
date: 2026-04-02
related: "[[Claude Code]], [[技能系统]], [[Skills开发]], [[MCP协议]]"
---

# Plugins 生态

> [!summary] 一句话定义
> Claude Code 的**插件与扩展生态系统**，通过 Marketplace、oh-my-claudecode (OMC)、社区工具三层生态扩展能力。

---

## 三层生态

| 层级 | 说明 | 示例 |
|------|------|------|
| **官方内置** | Claude Code 自带功能 | Hooks, Skills, Subagent |
| **Marketplace** | 官方插件市场 | MCP 服务器、主题 |
| **社区增强** | 第三方工具框架 | oh-my-claudecode, Cursor 集成 |

## oh-my-claudecode (OMC)

> [!note] 最强社区框架
> oh-my-claudecode 是 Claude Code 的**多代理编排层**，提供：
> - **Tier-0 工作流**：autopilot（全自动）/ ralph（循环验证）/ ultrawork（并行执行）/ team（多代理团队）
> - **专家代理目录**：explore / planner / architect / executor / designer / writer / code-reviewer 等
> - **Skills 生态**：内置 30+ 技能（TDD、安全审查、前端模式等）
> - **增强记忆**：Priority Context / Working Memory / Project Memory
> - **Hook 自动化**：Magic Keyword 触发 / 会话状态管理

### OMC 模型路由

| 任务复杂度 | 模型 | 适用场景 |
|-----------|------|---------|
| 简单查询 | Haiku | 快速查找、定义 |
| 标准实现 | Sonnet | 编码、重构、测试 |
| 复杂分析 | Opus | 架构设计、深度分析 |

### OMC 工作流模式

| 模式 | 命令 | 说明 |
|------|------|------|
| **autopilot** | `/autopilot` | 完全自动：计划→执行→验证 |
| **ralph** | `/ralph` | 循环执行直到验证通过 |
| **ultrawork** | `/ulw` | 并行执行多个独立任务 |
| **team** | `/team` | 多代理团队协作 |
| **ralplan** | `/ralplan` | 共识计划 + 自动执行 |

## 与 Skills/Plugins 的关系

```
Plugins 生态
├── [[技能系统]] (Skills)
│   ├── 内置 Skills（Claude Code 自带）
│   ├── OMC Skills（oh-my-claudecode 提供）
│   └── 自定义 Skills → [[Skills开发]]
├── [[MCP协议]] (MCP Servers)
│   ├── 官方 MCP 服务器
│   └── 社区 MCP 服务器
└── Hooks → [[Hooks系统]]
    ├── 内置 Hook 事件
    └── OMC 增强钩子
```

> [!tip] 推荐安装
> 对于有 3-4 个项目经验的用户，强烈建议安装 oh-my-claudecode：
> ```bash
> # 安装 OMC
> omc setup
> # 或手动配置
> # 参考官方文档
> ```

---

**相关笔记**：[[Claude Code]] · [[技能系统]] · [[Skills开发]] · [[MCP协议]] · [[Hooks系统]]
