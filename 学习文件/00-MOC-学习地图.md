---
tags: [MOC, 导航]
aliases: [学习地图, Map of Content, 知识导航]
date: 2026-04-02
---

# 🗺️ 学习地图 (MOC)

> [!info] 使用方法
> 这是知识库的导航中心。每个节点都是一篇笔记，用 `[[双链]]` 互相连接。
> 在 Obsidian 的**关系图谱**视图中可以看到所有知识的关联。

---

## 🧱 基础层：核心概念

```dataview
LIST
FROM "学习文件/01-核心概念"
SORT file.name ASC
```

**关联图**：
- [[Claude Code]] ← 补充 → [[MCP协议]] · [[Hooks系统]] · [[技能系统]]
- [[OpenClaw]] ← 补充 → [[记忆系统]] · [[Agent与多代理]] · [[架构设计]]
- [[MCP协议]] ← 桥接 → [[Claude Code]] ↔ [[OpenClaw]]

---

## 🔧 进阶层：Claude Code 深度

```dataview
LIST
FROM "学习文件/02-Claude Code进阶"
SORT file.name ASC
```

**学习顺序建议**：
1. [[Commands自定义命令]] → 2. [[Subagent子代理]] → 3. [[Skills开发]]
4. [[权限与沙箱]] → 5. [[Agent-SDK]] → 6. [[CI-CD与自动化]]

---

## 🦞 实战层：OpenClaw 部署与定制

```dataview
LIST
FROM "学习文件/03-OpenClaw实战"
SORT file.name ASC
```

**实战顺序建议**：
1. [[架构设计]] → 2. [[模型配置策略]] → 3. [[自定义技能开发]]
4. [[Docker部署]] → 5. [[安全最佳实践]]

---

## 🔄 协同层：CC + OC 联动

> [!important] 核心洞察
> Claude Code 和 OpenClaw 是 **AI Agent 时代的一体两面**：
> - CC 管**编程工作流**（终端里写代码）
> - OC 管**日常工作流**（聊天软件自动化）
> - 两者通过 Claude API + MCP 形成 AI 全家桶

| 协同笔记 | 说明 |
|---------|------|
| [[CC-OC协同工作流]] | 编程 + 消息通知 + 日程驱动 + 知识闭环 |
| [[MCP协议]] | CC 和 OC 共享的工具连接协议 |
| [[记忆系统]] | 不同实现，同一目标（让 AI 记住上下文） |
| [[技能系统]] | CC Skills vs OC Skills — 各有特色 |

---

## 🏆 应用层：最佳实践

```dataview
LIST
FROM "学习文件/04-最佳实践"
SORT file.name ASC
```

---

## ⚡ 工具层：速查

```dataview
LIST
FROM "学习文件/05-速查"
SORT file.name ASC
```

---

## 🔗 跨领域知识连接

> [!tip] 知识双链网络
> 以下概念在多个领域中交叉出现，是知识库的关键枢纽节点：

| 枢纽概念 | 涉及领域 |
|---------|---------|
| [[MCP协议]] | Claude Code · OpenClaw · Agent-SDK |
| [[技能系统]] | Claude Code Skills · OpenClaw Skills |
| [[Agent与多代理]] | Claude Code Subagent · OpenClaw 多Agent · Agent-SDK |
| [[记忆系统]] | OpenClaw Memory · Claude Code Memory(oh-my-claudecode) |
| [[安全最佳实践]] | 权限与沙箱 · 安全加固 · CI-CD |
| [[消息平台集成]] | OpenClaw · 架构设计 · 安全 |
| [[Plugins生态]] | Claude Code · Skills · MCP |
| [[CC-OC协同工作流]] | Claude Code · OpenClaw · MCP · 记忆系统 |
| [[常见陷阱与反模式]] | Claude Code · OpenClaw · 安全 · 性能 |
| [[环境搭建指南]] | Claude Code · OpenClaw · MCP · Docker |
