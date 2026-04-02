---
tags: [核心概念, OpenClaw]
aliases: [OC, openclaw, 开源AI助手, Clawdbot]
date: 2026-04-02
related: "[[Claude Code]], [[记忆系统]], [[架构设计]], [[Agent与多代理]]"
---

# OpenClaw

> [!summary] 一句话定义
> **开源 AI 私人助手框架** (227K+ Stars)，本地运行，连接 15+ 消息平台，支持 29+ AI 模型，隐私至上。

---

## 核心定位

| 维度 | 说明 |
|------|------|
| **产品类型** | 开源 AI 助手框架 |
| **前身** | Clawdbot → Moltbot → OpenClaw |
| **创始人** | Peter Steinberger |
| **Stars** | 227K+（史上增长最快的开源项目之一）|
| **许可证** | MIT |
| **技术栈** | TypeScript 5.9+ / Node.js 22+ |
| **配置格式** | JSON5（`~/.openclaw/openclaw.json`）|

## 与 Claude Code 的关系

> [!note] 互补而非竞争
> - [[Claude Code]] 管**编程工作流** — 终端里写代码、调 Bug、做架构
> - [[OpenClaw]] 管**日常工作流** — 消息平台管理、自动化、语音助手
> - 两者共享 Claude API，是 **AI Agent 时代的一体两面**

## 核心架构（三层）

```
消息平台层 (15+ Channels)
    WhatsApp · Telegram · Discord · Slack · 飞书 ...
        ↓
Gateway 网关 (ws://127.0.0.1:18789)
    路由 · 会话 · 安全 · 技能 · 记忆 · 工具
        ↓
AI 模型层 (29+ Providers)
    Claude · GPT · Gemini · Ollama · 通义千问 · DeepSeek ...
```

## 核心子系统

| 系统 | 说明 | 笔记 |
|------|------|------|
| **Gateway** | 核心守护进程，管理连接和调度 | [[架构设计]] |
| **Channel** | 消息平台抽象层，统一接口 | [[消息平台集成]]（未创建） |
| **Skills** | 50+ 内置技能 + 自定义开发 | [[技能系统]] · [[自定义技能开发]] |
| **Memory** | Markdown 文件记忆系统 | [[记忆系统]] |
| **Tools** | 浏览器控制、文件操作等 | [[架构设计]] |
| **Agent** | 多 Agent 路由和协作 | [[Agent与多代理]] |

## 关键 CLI 命令

```bash
npm install -g openclaw@latest          # 安装
openclaw onboard --install-daemon       # 初始化 + 注册守护进程
openclaw gateway --port 18789 --verbose # 调试模式启动
openclaw doctor                         # 健康检查
openclaw agent --message "你好"         # 命令行对话
openclaw update --channel stable        # 更新通道切换
```

## 版本策略

| 通道 | npm 标签 | 说明 |
|------|----------|------|
| stable | `latest` | 正式发布，推荐 |
| beta | `beta` | 预发布版本 |
| dev | `dev` | main 分支最新 |

> [!tip] 进阶要点（已有 2 次部署经验）
> 1. **多 Agent 路由** — 为不同场景创建专用 Agent → [[Agent与多代理]]
> 2. **自定义技能开发** — 超越 50+ 内置技能 → [[自定义技能开发]]
> 3. **Docker 生产部署** — 7x24 小时运行 → [[Docker部署]]
> 4. **安全加固** — 公网暴露必备 → [[安全最佳实践]]
> 5. **MCP 桥接** — 通过 mcporter 扩展 → [[MCP协议]]

---

**相关笔记**：[[Claude Code]] · [[架构设计]] · [[记忆系统]] · [[Agent与多代理]] · [[Docker部署]]
