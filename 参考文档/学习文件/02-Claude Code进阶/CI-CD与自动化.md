---
tags: [Claude-Code进阶, CI-CD, 自动化]
aliases: [CI-CD, GitHub Actions, 自动化流水线]
date: 2026-04-02
related: "[[Claude Code]], [[Hooks系统]], [[权限与沙箱]], [[安全最佳实践]]"
---

# CI/CD 与自动化

> [!summary] 一句话定义
> 将 Claude Code 集成到 **GitHub Actions** 等 CI/CD 系统中，实现自动代码审查、安全扫描、文档生成。

---

## GitHub Actions 集成

### 官方 Action

`anthropics/claude-code-action@v1`

### 基础配置

```yaml
name: Claude Code Review
on:
  pull_request:
    types: [opened, synchronize, reopened]

permissions:
  contents: read
  pull-requests: write
  issues: write

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          model: "claude-sonnet-4-6"
```

## 多阶段流水线

```
代码检查 → 单元测试 → Claude 审查 → 安全扫描 → 构建 → 部署
                         ↓              ↓
                    PR 评论         安全报告
```

### 关键 Jobs

| 阶段 | 触发条件 | Claude 任务 |
|------|---------|------------|
| **代码审查** | PR | 质量评估、Bug 分析 |
| **安全扫描** | PR | 注入、XSS、硬编码密钥 |
| **文档生成** | Push to main | API 文档自动更新 |
| **交互命令** | @claude 评论 | 响应指令执行 |

## Secrets 配置

在 GitHub `Settings > Secrets` 中添加：
- `ANTHROPIC_API_KEY` — API 密钥
- `GITHUB_TOKEN` — 自动提供

## 与 Hooks 的关系

> [!note] 两层自动化
> - **[[Hooks系统]]** — 本地开发时的实时自动化
> - **CI/CD** — 团队协作时的流程自动化
>
> 两者互补，Hooks 管实时，CI/CD 管流程。

## 安全最佳实践

> [!warning] CI/CD 安全
> 1. API Key 只用 GitHub Secrets，不硬编码
> 2. 设置 `timeout` 防止无限运行
> 3. PR 审查结果仅供参考，人工复核必须的
> 4. 使用最小权限原则

---

**相关笔记**：[[Claude Code]] · [[Hooks系统]] · [[安全最佳实践]] · [[权限与沙箱]]
