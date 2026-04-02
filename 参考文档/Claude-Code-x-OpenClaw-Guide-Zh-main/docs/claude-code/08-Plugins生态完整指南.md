# Claude Code Plugins生态完整指南：从安装到自定义开发

> **课程信息**
>
> - **作者**：老金
> - **预计学时**：4-6小时
> - **难度等级**：⭐⭐ 入门级
> - **更新日期**：2026年3月
> - **适用版本**：Claude Code v2.1.69+（验证于2026-03-18）

---

## 📚 本课学习目标

完成本课学习后，你将能够：

1. **理解Plugin生态**：掌握Plugin与Commands/Skills/MCP的区别
2. **安装和使用Plugin**：通过 `--plugin-dir` 加载本地Plugin
3. **浏览Marketplace**：在官方网页Marketplace发现优质Plugin
4. **创建自定义Plugin**：从零开发一个完整的Plugin包
5. **发布Plugin**：将Plugin分享到GitHub和社区
6. **排查Plugin问题**：独立解决90%的常见故障

---

## 🗺️ 学习路径导航（先看这里！）

### 路径A：快速上手（⏱️ 30分钟）

**适合人群**：急着用Plugin，想快速体验

**只看这些章节**：

```
✅ 术语表（3分钟）
✅ 第1章：Plugins概览（10分钟）
✅ 第2章：5分钟快速开始（15分钟）
```

### 路径B：Plugin开发者（⏱️ 3小时）

**适合人群**：想创建自己的Plugin

**学习顺序**：

```
✅ 第1-2章：概念+快速上手（30分钟）
✅ 第3章：Marketplace深度指南（30分钟）
✅ 第4章：创建自定义Plugin（90分钟）
✅ 第5章：发布与分享（30分钟）
```

### 路径C：问题排查（⏱️ 5分钟）

**适合人群**：Plugin出问题了

**直接跳到**：

```
🔧 第6章：故障排查指南
🔧 第7章：FAQ
```

---

## 术语表（小白必读）

| 术语 | 英文 | 解释 |
|------|------|------|
| **Plugin** | Plugin | Claude Code的扩展包，可包含Commands+Skills+Hooks+MCP配置 |
| **Marketplace** | Marketplace | Plugin商店，浏览和发现Plugin的网页平台 |
| **.claude-plugin/plugin.json** | - | Plugin的元数据清单文件，位于 `.claude-plugin/` 子目录中 |
| **--plugin-dir** | - | Claude Code启动参数，指定加载Plugin的目录路径 |
| **Skill** | Skill | Plugin中的核心能力模块（SKILL.md定义） |
| **Hook** | Hook | Plugin中的自动化触发器（如代码提交前检查） |

---

## 第1章：Plugins生态概览

### 1.1 什么是Claude Code Plugin？

**定义**：

Plugin是Claude Code的扩展包，可以添加新的命令、专业能力和自动化流程，且可以跨项目和团队共享。

**类比理解**：

```
手机              |  Claude Code
------------------|------------------
操作系统(iOS/Android) | Claude Code核心
App Store        | Plugin Marketplace（网页）
安装的APP        | 已安装的Plugins
APP更新          | Plugin手动更新（git pull）
```

**核心价值**：

1. **可复用性**：一次开发，多个项目使用
2. **易分享性**：通过GitHub一键克隆
3. **模块化**：每个Plugin专注一个领域
4. **社区驱动**：社区持续贡献优质Plugin

### 1.2 Plugins vs Commands/Skills/MCP

| 维度 | Commands | Skills | MCP | **Plugins** |
|------|----------|--------|-----|-------------|
| **定义** | Markdown提示词 | 专业Agent能力 | 外部服务集成 | **打包的扩展** |
| **位置** | `.claude/commands/` | `.claude/skills/` | `.mcp.json` | **本地目录** |
| **可分享性** | ❌ 手动复制 | ❌ 手动复制 | ⚠️ 需配置 | **✅ git clone** |
| **包含内容** | 单个提示词 | 多个文件+配置 | 服务器配置 | **Commands+Skills+MCP** |
| **加载方式** | 自动（在项目目录中） | 自动（在项目目录中） | 自动（配置后） | **`--plugin-dir`** |

**关键区别**：Plugin是一个"超集"概念：

```
Plugin = Commands + Skills + Hooks + MCP配置 + 文档
```

### 1.3 Plugins生态现状（2026年2月）

**官方数据**：

- **当前版本**：Claude Code v2.1.52（2026年2月验证）
- **官方Marketplace**：✅ 已上线（code.claude.com/plugins）— 网页浏览
- **社区Plugin**：持续增长中

**主流Plugin来源**：

1. **Anthropic官方Marketplace**：
   - URL：`https://code.claude.com/plugins`
   - 特点：审核严格，质量保证，网页浏览

2. **Jeremy Longshore社区合集**：
   - URL：`https://github.com/jeremylongshore/claude-code-plugins-plus`
   - 特点：100%符合Anthropic Skills Schema

3. **GitHub搜索**：
   - 搜索关键词：`claude-code-plugin`
   - 特点：最丰富的来源，质量参差不齐

> ⚠️ **重要说明**：Claude Code 没有 `claude plugins` 这样的CLI子命令。Plugin的安装有两种方式：(1) 在交互模式中使用 `/plugin install` 斜杠命令；(2) 手动 git clone + `--plugin-dir` 指定目录。

---

## 第2章：5分钟快速开始

### 2.1 安装你的第一个Plugin

**目标**：从GitHub克隆一个Plugin并加载使用

**前置条件检查**：

```bash
# 确认Claude Code已安装
claude --version
# 预期输出：2.1.52 (Claude Code)

# 确认在项目目录中
cd /path/to/your/project
```

**步骤1：克隆Plugin到本地**

```bash
# 创建plugins目录（如果不存在）
mkdir -p .claude/plugins

# 克隆一个Plugin（以社区Plugin为例）
git clone https://github.com/jeremylongshore/claude-code-plugins-plus .claude/plugins/plugins-plus
```

**步骤2：使用 `--plugin-dir` 加载Plugin**

```bash
# 启动Claude Code时指定Plugin目录
claude --plugin-dir .claude/plugins/plugins-plus
```

Claude Code 会自动扫描该目录下的 `.claude-plugin/plugin.json`，加载其中定义的 Commands、Skills、Hooks。

> 💡 **开发小技巧**：开发Plugin时修改了文件，可以在交互模式中使用 `/reload-plugins` 重新加载，无需重启 Claude Code。

**步骤3：验证Plugin已加载**

在Claude Code交互模式中：

```bash
You: /help

# 如果Plugin包含自定义命令，你会在命令列表中看到它们
# 如果Plugin包含Skills，Claude会自动获得对应能力
```

**步骤4：多个Plugin目录**

```bash
# 可以同时加载多个Plugin目录
claude --plugin-dir ./plugin-a --plugin-dir ./plugin-b
```

### 2.2 卸载Plugin

Plugin就是本地目录，删除即可：

```bash
# 删除Plugin目录
rm -rf .claude/plugins/plugins-plus

# 下次启动不带 --plugin-dir 参数即可
```

---

## 第3章：使用Marketplace深度指南

### 3.1 浏览Marketplace

> ⚠️ **注意**：Marketplace 是**网页平台**，不是CLI命令。

**访问方式**：

打开浏览器访问 `https://code.claude.com/plugins`

**Marketplace功能**：

| 功能 | 说明 |
|------|------|
| **分类浏览** | 按用途分类：文档处理、代码质量、项目管理等 |
| **搜索** | 按关键词搜索Plugin |
| **详情页** | 查看Plugin说明、安装量、评分、源码链接 |
| **安装指引** | 每个Plugin页面提供安装命令（git clone） |

### 3.2 安装Plugin的方式

**方式1：从GitHub克隆（最常用）**

```bash
# 在Marketplace找到Plugin后，复制其GitHub地址
git clone https://github.com/author/my-plugin .claude/plugins/my-plugin

# 启动时加载
claude --plugin-dir .claude/plugins/my-plugin
```

**方式2：指定本地路径（开发调试用）**

```bash
# 直接指向本地开发中的Plugin
claude --plugin-dir /path/to/my-local-plugin
```

**方式3：指定当前目录（Plugin项目本身）**

```bash
# 在Plugin项目根目录中
claude --plugin-dir .
```

### 3.3 管理已安装Plugins

由于Plugin就是本地目录，管理操作都是标准文件/git操作：

```bash
# 查看已安装的Plugins
ls .claude/plugins/

# 更新Plugin（git pull最新版本）
cd .claude/plugins/my-plugin && git pull

# 切换Plugin版本
cd .claude/plugins/my-plugin && git checkout v1.2.0

# 查看Plugin信息
cat .claude/plugins/my-plugin/.claude-plugin/plugin.json
```

### 3.4 Plugin配置

部分Plugin支持自定义配置。查看Plugin的 `.claude-plugin/plugin.json`：

```bash
# 查看Plugin元数据
cat .claude/plugins/my-plugin/.claude-plugin/plugin.json
```

配置方式取决于Plugin的实现——通常在Plugin目录下创建配置文件。具体请参考每个Plugin的README。

---

## 第4章：创建自定义Plugin

### 4.1 Plugin结构规范

**最小Plugin结构**：

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json      # 必需：Plugin元数据清单
├── .mcp.json            # 可选：MCP配置
├── README.md            # 推荐：使用文档
├── skills/              # 可选：Agent Skills
│   └── my-skill/
│       └── SKILL.md
├── commands/            # 可选：Slash Commands
│   └── my-command.md
├── agents/              # 可选：Agent定义
│   └── my-agent.md
└── hooks/               # 可选：Hooks
    └── pre-commit.py
```

**.claude-plugin/plugin.json 规范**：

```json
{
  "name": "my-awesome-plugin",
  "description": "A plugin that does awesome things",
  "version": "1.0.0",
  "author": "Your Name"
}
```

> 💡 **注意**：`plugin.json` 必须放在 `.claude-plugin/` 子目录中，不是Plugin根目录。官方规范只要求 `name`、`description`、`version`、`author` 四个核心字段。

### 4.2 创建第一个Plugin：Hello World

**步骤1：创建Plugin目录**

```bash
mkdir -p hello-world-plugin/.claude-plugin
mkdir -p hello-world-plugin/commands
cd hello-world-plugin
```

**步骤2：创建 .claude-plugin/plugin.json**

```json
{
  "name": "hello-world-plugin",
  "description": "A simple hello world plugin for learning",
  "version": "1.0.0",
  "author": "Claude Student"
}
```

**步骤3：创建自定义命令**

创建 `commands/hello.md`：

```markdown
# Hello World

Say hello to the user in a creative and fun way.
Include the current date and a random programming joke.
```

**步骤4：创建 README.md**

```markdown
# Hello World Plugin

A simple plugin that adds a `/hello` command to Claude Code.

## Installation

\`\`\`bash
git clone https://github.com/yourname/hello-world-plugin
claude --plugin-dir ./hello-world-plugin
\`\`\`

## Usage

In Claude Code interactive mode:
\`\`\`
You: /hello
\`\`\`

## Features

- Creative greetings
- Current date display
- Random programming jokes
```

**步骤5：测试Plugin**

```bash
# 在项目目录中启动Claude Code，加载Plugin
claude --plugin-dir /path/to/hello-world-plugin

# 在交互模式中使用
You: /hello
```

### 4.3 进阶Plugin：带Skills的Plugin

**目标**：创建一个包含Skill的Plugin，让Claude具备代码审查能力

**目录结构**：

```
code-review-plugin/
├── .claude-plugin/
│   └── plugin.json
├── README.md
├── commands/
│   └── review.md
└── skills/
    └── code-reviewer/
        └── SKILL.md
```

**skills/code-reviewer/SKILL.md**：

```markdown
---
name: code-reviewer
description: Expert code review skill
---

You are an expert code reviewer. When reviewing code:

1. Check for security vulnerabilities (SQL injection, XSS, etc.)
2. Identify performance bottlenecks
3. Suggest improvements for readability
4. Verify error handling completeness
5. Check naming conventions consistency

Output format:
- 🔴 CRITICAL: Must fix before merge
- 🟡 WARNING: Should fix
- 🟢 INFO: Nice to have
```

**commands/review.md**：

```markdown
Review the current git diff and provide a detailed code review.
Use the code-reviewer skill for analysis.
Focus on security, performance, and maintainability.
```

**测试**：

```bash
claude --plugin-dir ./code-review-plugin

You: /review
# Claude会使用code-reviewer Skill分析你的代码变更
```

### 4.4 Plugin最佳实践

| 原则 | 说明 |
|------|------|
| **单一职责** | 每个Plugin专注一个领域（代码审查、文档生成等） |
| **清晰文档** | README必须包含安装步骤、使用示例、配置说明 |
| **版本管理** | 使用语义化版本号（SemVer），打git tag |
| **最小依赖** | 尽量减少外部依赖，保持Plugin轻量 |
| **安全第一** | 不在Plugin中硬编码密钥，使用环境变量 |

---

## 第5章：发布与分享Plugin

### 5.1 发布前检查清单

```
✅ .claude-plugin/plugin.json 字段完整（name, version, description, author）
✅ README.md 包含安装和使用说明
✅ 所有命令和Skills已测试通过
✅ 无硬编码密钥或敏感信息
✅ .gitignore 排除了不必要的文件
✅ LICENSE 文件存在
```

### 5.2 发布到GitHub

```bash
# 初始化git仓库
cd my-plugin
git init
git add -A
git commit -m "feat: initial release v1.0.0"

# 创建GitHub仓库并推送
gh repo create my-plugin --public --source=. --push

# 打版本标签
git tag v1.0.0
git push --tags
```

**推荐的GitHub仓库设置**：

- 添加 Topics：`claude-code-plugin`、`claude-code`、`ai-plugin`
- 写清楚 Description
- 添加 `claude-code-plugin` topic 方便社区搜索发现

### 5.3 分享到社区

1. **提交到 claude-code-plugins-plus**：
   - Fork `jeremylongshore/claude-code-plugins-plus`
   - 添加你的Plugin信息
   - 提交PR

2. **在GitHub Discussions分享**：
   - 到 `anthropics/claude-code` 的 Discussions 板块分享

3. **提交到官方Marketplace**：
   - 访问 `code.claude.com/plugins` 查看提交指南
   - 需要通过官方审核

---

## 第6章：故障排查指南

### 6.1 Plugin加载失败

**症状**：`--plugin-dir` 指定后，Plugin的命令/Skills没有生效

**排查步骤**：

```bash
# 1. 确认路径正确
ls /path/to/your/plugin/.claude-plugin/plugin.json

# 2. 验证plugin.json格式
cat /path/to/your/plugin/.claude-plugin/plugin.json | python3 -m json.tool

# 3. 使用debug模式启动
claude --plugin-dir /path/to/your/plugin --debug
```

### 6.2 命令不显示

**可能原因**：

| 原因 | 解决方案 |
|------|----------|
| commands目录路径错误 | 确认在Plugin根目录下有 `commands/` 目录 |
| 命令文件不是.md格式 | 命令文件必须是 `.md` 后缀 |
| .claude-plugin/plugin.json 缺失 | 确认Plugin根目录下有 `.claude-plugin/plugin.json` |
| 文件权限问题 | 确认文件可读：`chmod 644 commands/*.md` |

### 6.3 Skills不生效

**排查**：

```bash
# 确认SKILL.md存在且格式正确
cat /path/to/plugin/skills/my-skill/SKILL.md

# 确认frontmatter格式
# 必须以 --- 开头和结尾，包含name和description
```

### 6.4 多Plugin冲突

如果多个Plugin定义了同名命令：

```bash
# 后加载的Plugin会覆盖先加载的
# 调整 --plugin-dir 的顺序来控制优先级
claude --plugin-dir ./plugin-low-priority --plugin-dir ./plugin-high-priority
```

---

## 第7章：FAQ（常见问题）

### Q1：Plugin和Skill有什么区别？

**Skill** 是单个能力定义（一个SKILL.md文件），**Plugin** 是一个完整的扩展包，可以包含多个Skills + Commands + Hooks + MCP配置。Plugin是Skill的超集。

### Q2：有没有 `claude plugins install` 命令？

Claude Code 没有 `claude plugins` 这样的CLI子命令，但在交互模式中有 `/plugin install` 斜杠命令。安装Plugin有两种方式：
1. 在交互模式中使用 `/plugin install {plugin-name}` 安装
2. `git clone` 到本地，启动时用 `--plugin-dir` 指定路径

### Q3：Plugin会访问我的代码吗？

Plugin中的Skills和Commands在Claude Code的沙箱中运行，遵循与内置工具相同的权限模型。Plugin不能绕过权限系统。

### Q4：如何更新Plugin？

```bash
cd .claude/plugins/my-plugin
git pull origin main
```

下次启动Claude Code时会自动加载最新版本。

### Q5：如何卸载Plugin？

```bash
# 删除Plugin目录
rm -rf .claude/plugins/my-plugin

# 启动时不再指定该 --plugin-dir 即可
```

### Q6：可以同时加载多少个Plugin？

没有硬性限制，但每个Plugin都会增加上下文占用。建议同时加载不超过5个Plugin，按需加载。

### Q7：Plugin开发需要懂编程吗？

不一定。最简单的Plugin只需要写Markdown文件（Commands和Skills都是Markdown）。只有需要Hooks（自动化脚本）时才需要编程能力。

### Q8：Plugin可以离线使用吗？

可以。Plugin是本地文件，加载后不需要网络。但如果Plugin包含MCP配置连接外部服务，那部分功能需要网络。

### Q9：如何让Plugin在所有项目中生效？

```bash
# 方法1：每次启动时指定
claude --plugin-dir ~/.claude/global-plugins/my-plugin

# 方法2：设置shell别名
alias claude='claude --plugin-dir ~/.claude/global-plugins/my-plugin'
```

### Q10：Plugin报错如何获取帮助？

1. 查看Plugin的GitHub Issues
2. 使用 `claude --debug` 查看详细日志
3. 检查Plugin的README中的Troubleshooting部分

---

## 🔗 相关链接

| 资源 | 链接 | 说明 |
|------|------|------|
| **上一节** | [07-Skills定制完整指南](./07-Skills定制完整指南.md) | 创建可复用功能包 |
| **下一节** | [09-Agent-SDK完整指南](./09-Agent-SDK完整指南.md) | 编程开发AI Agent |

---

## 📋 附录：速查表

### Plugin操作速查

| 操作 | 命令 |
|------|------|
| **安装Plugin** | `/plugin install <name>` 或 `git clone <url>` |
| **加载Plugin** | `claude --plugin-dir .claude/plugins/<name>` |
| **加载多个** | `claude --plugin-dir ./a --plugin-dir ./b` |
| **更新Plugin** | `cd .claude/plugins/<name> && git pull` |
| **卸载Plugin** | `rm -rf .claude/plugins/<name>` |
| **查看Plugin信息** | `cat .claude/plugins/<name>/.claude-plugin/plugin.json` |
| **开发时重载** | 交互模式中使用 `/reload-plugins` |
| **调试Plugin** | `claude --plugin-dir <path> --debug` |
| **浏览Marketplace** | 浏览器访问 `code.claude.com/plugins` |

### Plugin目录结构速查

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json      # 必需：元数据清单
├── .mcp.json            # 可选：MCP配置
├── README.md            # 推荐：文档
├── commands/*.md        # 可选：Slash命令
├── skills/*/SKILL.md    # 可选：Agent能力
├── agents/*.md          # 可选：Agent定义
└── hooks/*.py           # 可选：自动化脚本
```

> 💡 **命名空间**：Plugin中的Skills会自动添加命名空间前缀，格式为 `/plugin-name:skill-name`，避免与其他Plugin冲突。

---

> **最后更新**：2026年2月25日 | **适用版本**：Claude Code v2.1.52+
