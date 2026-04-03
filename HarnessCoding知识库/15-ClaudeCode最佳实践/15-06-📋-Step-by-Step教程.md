---
type: tutorial
tags: [claude-code, tutorial, setup, beginner]
description: Claude Code 开发环境 Step-by-Step 搭建教程
---

# Step-by-Step 搭建教程

> ⏱ 适用水平: 入门 | 重要性: 必读 | 预计时间: 2-4小时

---

## 概述

本教程将逐步指导你搭建完整的 Claude Code 开发环境。完成后，你将拥有：

- ✅ 可用的 Claude Code 环境
- ✅ OMC 多智能体框架
- ✅ 基础插件和 Skill
- ✅ 自定义 Hook 配置
- ✅ 个性化的项目结构

---

## 第一步：基础配置

### 1.1 安装 Claude Code

```bash
# macOS/Linux
curl -sL https:// claude.com/install.sh | sh

# Windows (通过 npm)
npm install -g @anthropic/claude-code

# 验证安装
claude --version
```

### 1.2 首次配置

```bash
# 启动配置向导
claude setup

# 交互式设置 API Key
# 输入你的 Anthropic API Key

# 选择默认模型
# 推荐: sonnet (日常使用), opus (复杂任务)
```

### 1.3 全局配置

```bash
# 创建全局配置目录
mkdir -p ~/.claude

# 创建 settings.json
cat > ~/.claude/settings.json << 'EOF'
{
  "model": "sonnet",
  "temperature": 0.7,
  "maxTokens": 4096,
  "垂类": "development"
}
EOF
```

### 1.4 验证安装

```bash
# 测试基本功能
claude "Hello, world!"

# 确认版本
claude --version
```

---

## 第二步：安装插件和 Skill

### 2.1 安装 OMC (oh-my-claudecode)

```bash
# 通过 npm 安装
npm install -g oh-my-claudecode

# 初始化 OMC
omc setup

# 查看可用技能
omc skill list
```

### 2.2 安装 Superpowers

```bash
# 克隆 Superpowers 仓库
git clone https://github.com/superpowers/superpowers ~/.claude/superpowers

# 安装依赖
cd ~/.claude/superpowers
npm install

# 激活 Superpowers
claude reload
```

### 2.3 配置 MCP Servers

```bash
# 安装常用 MCP Server
claude mcp install filesystem
claude mcp install git
claude mcp install github

# 查看已安装
claude mcp list
```

### 2.4 VS Code 插件安装

在 VS Code 中搜索并安装：

| 插件 | 说明 |
|------|------|
| **Claude Code** | 主插件 |
| **GitLens** | Git 可视化 |
| **Prettier** | 代码格式化 |
| **ESLint** | 代码检查 |
| **REST Client** | API 测试 |
| **Todo Tree** | TODO 追踪 |

---

## 第三步：配置 Hook

### 3.1 创建 Hooks 目录

```bash
mkdir -p ~/.claude/hooks
```

### 3.2 创建安全 Hook

```javascript
// ~/.claude/hooks/security.js

// 危险命令拦截
export const blockDangerousCommands = {
  event: "PreToolUse",
  filter: { tool: "Bash", pattern: /rm\s+-rf\s+\/|drop\s+database/i },
  async handle(context) {
    console.log(`[Security Hook] Blocked: ${context.input}`);
    return {
      status: "block",
      reason: "危险命令已拦截"
    };
  }
};

// 敏感文件保护
export const protectSensitiveFiles = {
  event: "PreToolUse",
  filter: { tool: ["Read", "Edit"], pattern: /\.env|password|secret/i },
  async handle(context) {
    return {
      status: "block",
      reason: "敏感文件访问已阻止"
    };
  }
};
```

### 3.3 创建上下文注入 Hook

```javascript
// ~/.claude/hooks/context.js

export const injectProjectContext = {
  event: "SessionStart",
  async handle(context) {
    const cwd = process.cwd();
    return {
      status: "modify",
      additionalContext: `当前目录: ${cwd}`
    };
  }
};
```

### 3.4 创建审计 Hook

```javascript
// ~/.claude/hooks/audit.js

export const operationAuditLog = {
  event: "PostToolUse",
  filter: { tool: ["Write", "Edit", "Rm"] },
  async handle(context) {
    console.log(`[Audit] ${context.tool}: ${JSON.stringify(context.input)}`);
    return { status: "approve" };
  }
};
```

### 3.5 注册 Hooks

```bash
# 更新 settings.json
cat > ~/.claude/settings.json << 'EOF'
{
  "model": "sonnet",
  "hooks": {
    "enabled": true,
    "items": [
      "./hooks/security.js",
      "./hooks/context.js",
      "./hooks/audit.js"
    ]
  }
}
EOF
```

### 3.6 验证 Hook 配置

```bash
# 测试 Hook 是否生效
claude "Delete all files"

# 应该看到拦截提示
```

---

## 第四步：自定义 Skill 开发

### 4.1 创建 Skills 目录

```bash
mkdir -p ~/.claude/skills
```

### 4.2 创建项目初始化 Skill

```markdown
<!-- ~/.claude/skills/project-init.md -->
---
name: project-init
description: 快速初始化新项目
type: skill
trigger:
  - "init-project"
  - "new-project"
version: "1.0.0"
tags: [project, setup]
agent:
  model: sonnet
  max_tokens: 4096
requires:
  tools: [Bash, Write, Edit]
---

# 项目初始化

## 功能
快速创建一个符合规范的新项目结构

## 使用方法
```
/skill project-init <项目名称> [--template ts]
```

## 支持的模板

| 模板 | 说明 |
|------|------|
| `ts` | TypeScript + Node.js |
| `react` | React + TypeScript |
| `python` | Python 项目 |

## 工作流程

1. 创建目录结构
2. 初始化配置文件
3. 安装依赖
4. 初始化 Git

## 示例

```bash
/skill project-init my-app --template ts
```
```

### 4.3 创建代码审查 Skill

```markdown
<!-- ~/.claude/skills/code-review.md -->
---
name: code-review
description: 自动化代码审查
type: skill
trigger:
  - "review"
  - "code-review"
version: "1.0.0"
tags: [quality, review]
agent:
  model: opus
  temperature: 0.3
requires:
  tools: [Read, Bash, Grep]
---

# 代码审查

## 功能
对代码进行多维度审查

## 使用方法
```
/skill code-review <文件或目录>
```

## 审查维度

- 代码质量
- 安全性
- 性能
- 最佳实践

## 示例

```bash
/skill code-review src/
```
```

### 4.4 验证 Skill

```bash
# 列出所有 Skill
omc skill list

# 测试 Skill
claude "init-project test-app --template ts"

# 应该看到项目初始化流程
```

---

## 第五步：高级配置

### 5.1 配置 AGENTS.md

```markdown
<!-- 项目根目录/AGENTS.md -->
# Agent 配置文件

## 可用 Agent

| Agent | 模型 | 用途 |
|-------|------|------|
| architect | opus | 架构设计 |
| executor | sonnet | 代码实现 |
| reviewer | opus | 代码审查 |

## 默认配置

```yaml
default:
  model: sonnet
  temperature: 0.7

architect:
  model: opus
  max_tokens: 8192
```
```

### 5.2 配置记忆系统

```json
<!-- ~/.claude/memories/project-memory.json -->
{
  "projectName": "我的项目",
  "techStack": ["React", "TypeScript", "Node.js"],
  "conventions": {
    "commitFormat": "type(scope): description"
  },
  "team": {
    "size": 3,
    "members": ["Alice", "Bob", "Charlie"]
  }
}
```

### 5.3 配置项目 CLAUDE.md

```markdown
<!-- 项目根目录/CLAUDE.md -->
# 我的项目

## 项目概述
这是一个使用 React + TypeScript 构建的 Web 应用。

## 技术栈
- 前端: React 18 + TypeScript
- 后端: Node.js + Express
- 样式: Tailwind CSS

## 开发规范

### 代码风格
- 使用 ESLint + Prettier
- TypeScript 严格模式
- 函数式组件

### Git 规范
- 分支: feature/xxx, bugfix/xxx
- 提交: type(scope): description

## 常用命令

```bash
npm install      # 安装依赖
npm run dev     # 开发
npm test        # 测试
npm run build   # 构建
```
```

### 5.4 配置 .claudeignore

```gitignore
# .claudeignore
node_modules/
dist/
build/
.env
*.log
coverage/
```

### 5.5 配置 CI/CD 集成

```yaml
# .github/workflows/claude.yml
name: Claude Code CI
on: [push, pull_request]
jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Run lint
        run: npm run lint
      - name: Run tests
        run: npm test
```

---

## 验证清单

完成以上步骤后，检查以下项目：

### 基础功能
- [ ] Claude Code 可正常启动
- [ ] 可以执行基本命令
- [ ] API Key 配置正确

### 插件和 Skill
- [ ] OMC 已安装并可用
- [ ] MCP Servers 已配置
- [ ] VS Code 插件已安装

### Hook 配置
- [ ] 危险命令被正确拦截
- [ ] 敏感文件被保护
- [ ] 审计日志正常工作

### Skill 开发
- [ ] 自定义 Skill 可被触发
- [ ] Skill 按预期工作

### 高级配置
- [ ] AGENTS.md 配置正确
- [ ] 记忆系统工作正常
- [ ] CLAUDE.md 包含项目说明

---

## 常见问题

### Q1: Hook 不生效？

```bash
# 检查 settings.json 配置
cat ~/.claude/settings.json

# 确保 hooks.enabled = true
# 确保 items 路径正确

# 重新加载配置
claude reload
```

### Q2: Skill 不显示？

```bash
# 确保 Skill 文件在正确位置
ls ~/.claude/skills/

# 检查 frontmatter 格式
# 确保 type: skill

# 重新加载
claude reload
```

### Q3: MCP Server 连接失败？

```bash
# 检查 MCP 配置
cat ~/.claude/settings.json | jq '.mcpServers'

# 查看 MCP 日志
claude mcp debug

# 重新安装
claude mcp remove <server>
claude mcp install <server>
```

---

## 下一步

完成基础配置后，推荐继续学习：

- [[15-01-⚡-ClaudeCode插件推荐]] - 深入了解插件生态
- [[15-03-🪝-自定义Hook配置]] - 高级 Hook 配置
- [[15-04-🎯-自定义Skill开发]] - Skill 开发进阶
- [[15-05-🏗️-整体架构建议]] - 项目结构最佳实践

---

## 相关章节

- [[../00-入口/00-02-🚀-快速上手指南]] - 快速上手
- [[../01-架构总览/01-01-📐-架构概览]] - 架构概览
- [[../04-Hook系统/04-01-🪝-Hook系统]] - Hook 系统
- [[../08-Skill系统/08-01-📚-Skill系统]] - Skill 系统

---

> [!cite]- 知识来源
>
> | 知识点 | 来源 |
> |-------|------|
> | **官方文档** | [Claude Code Docs](https://docs.anthropic.com/zh-CN/docs/claude-code) |
> | **OMC 教程** | [oh-my-claudecode](https://github.com/oh-my-claudecode/oh-my-claudecode) |
> | **Superpowers** | [superpowers](https://github.com/superpowers/superpowers) |
> | **everything-claude-code** | [everything-claude-code](https://github.com/everything-claude-code/everything-claude-code) |
