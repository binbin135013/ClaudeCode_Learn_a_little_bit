---
type: best-practice
tags: [claude-code, skill, development, customization]
description: Claude Code 自定义 Skill 开发指南
---

# 自定义 Skill 开发指南

> ⏱ 难度: ★★★ | 重要性: ★★★ | 推荐学习时间: 2-3天

---

## 1. Skill 设计规范

### 1.1 Skill 核心概念

| 概念 | 说明 |
|------|------|
| **触发方式** | `/skill name` 或触发词 |
| **工作流** | 声明式的步骤定义 |
| **上下文** | 跨调用保持状态 |
| **组合能力** | Skill 可嵌套调用 |

### 1.2 设计原则

```
1. 单一职责 - 每个 Skill 只做一件事
2. 可组合性 - Skill 之间可相互调用
3. 声明式配置 - 通过 frontmatter 定义行为
4. 用户友好 - 清晰的输入输出
```

### 1.3 Skill 文件位置

```
~/.claude/skills/           # 全局 Skill
project/.claude/skills/     # 项目级 Skill
```

---

## 2. Skill 模板

### 2.1 完整模板

```markdown
---
name: my-custom-skill
description: 技能功能描述
type: skill
trigger:
  - "trigger-word-1"
  - "trigger-word-2"
version: "1.0.0"
author: "Your Name"
tags:
  - tag1
  - tag2
category: utility

# Agent 配置
agent:
  model: sonnet
  temperature: 0.7
  max_tokens: 4096

# 工作流配置
workflow:
  parallel: false
  retry: 3
  timeout: 300000

# 依赖配置
requires:
  tools: [Read, Edit, Bash]
  skills: [other-skill]
---

# 技能名称

## 功能概述
一句话描述技能的功能

## 触发方式
- 命令: `/skill skill-name [参数]`
- 触发词: `trigger-word`

## 参数说明

| 参数 | 必填 | 说明 | 默认值 |
|-----|-----|-----|-------|
| param1 | 是 | 参数1说明 | - |
| param2 | 否 | 参数2说明 | default |

## 工作流程

1. **步骤一**: 描述
2. **步骤二**: 描述
3. **步骤三**: 描述

## 示例

### 示例 1
```bash
skill-name --param1 value
```

### 示例 2
```bash
trigger-word param1
```

## 注意事项
- 注意事项1
- 注意事项2
```

### 2.2 Frontmatter 字段说明

| 字段 | 类型 | 必需 | 说明 |
|-----|-----|-----|-----|
| `name` | string | 是 | 技能唯一标识 |
| `type` | string | 是 | 固定为 "skill" |
| `description` | string | 是 | 功能描述 |
| `trigger` | array | 否 | 触发词列表 |
| `version` | string | 否 | 版本号 |
| `author` | string | 否 | 作者 |
| `tags` | array | 否 | 标签 |
| `category` | string | 否 | 分类 |
| `agent` | object | 否 | Agent 配置 |
| `workflow` | object | 否 | 工作流配置 |
| `requires` | object | 否 | 依赖配置 |

### 2.3 高级字段

```yaml
---
# 三级参数替换
arguments: arg1 arg2 arg3
argument-hint: "param1: 第一个参数, param2: 第二个参数"

# 工具权限
allowed-tools: [Read, Edit, Bash, Write]

# 模型配置
model: sonnet
effort: medium              # low | medium | high

# 调用控制
user_invocable: true        # 是否可用户调用
disable-model-invocation: false

# 上下文模式
context: fork              # fork | inline

# 条件触发
paths: ["*.py", "src/**"]  # 在特定路径下自动激活

# Hook 绑定
hooks:
  pre: hook-name           # 前置 Hook
  post: hook-name          # 后置 Hook

# Shell 类型
shell: bash               # bash | powershell
---
```

---

## 3. 实用 Skill 示例

### 3.1 项目初始化 Skill

```markdown
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

## 触发方式
```
/skill project-init <项目名称> [--template ts]
```

## 工作流程

1. **分析模板**: 根据 `--template` 选择项目模板
2. **创建目录**: 生成标准目录结构
3. **初始化配置**: 创建 package.json、tsconfig.json 等
4. **安装依赖**: 执行 npm install
5. **初始化 Git**: git init 和初始提交

## 支持的模板

| 模板 | 说明 |
|------|------|
| `ts` | TypeScript + Node.js |
| `react` | React + TypeScript |
| `python` | Python 项目 |
| `go` | Go 项目 |

## 示例

```
/skill project-init my-app --template ts
```

## 注意事项
- 需要 Node.js 环境
- 默认使用 npm，可指定 `--yarn` 或 `--pnpm`
```

### 3.2 代码审查 Skill

```markdown
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
对代码进行多维度审查，包括代码质量、安全性、性能

## 触发方式
```
/skill code-review <file-or-dir>
```

## 审查维度

| 维度 | 检查内容 |
|------|---------|
| **代码质量** | 命名规范、注释完整、复杂度 |
| **安全性** | 注入风险、敏感信息、依赖漏洞 |
| **性能** | 循环优化、内存泄漏、缓存策略 |
| **最佳实践** | 设计模式、错误处理、资源管理 |

## 工作流程

1. **读取代码**: 分析文件或目录
2. **静态分析**: 运行 linter
3. **依赖检查**: 检查依赖安全性
4. **生成报告**: 输出结构化审查结果
5. **修复建议**: 提供改进建议

## 输出格式

```markdown
## 审查报告

### 文件: src/index.ts

#### 代码质量
- [x] 命名规范
- [ ] 缺少注释 (第 15 行)
- [ ] 函数复杂度高 (calc(), 圈复杂度 12)

#### 安全性
- [ ] 潜在注入风险 (第 23 行)
- [x] 无敏感信息泄露

#### 改进建议
1. 为 `calc()` 函数添加注释
2. 重构高复杂度函数
3. 添加输入验证
```

## 注意事项
- 使用 `--severity` 调整严重级别
- 使用 `--auto-fix` 自动修复简单问题
```

### 3.3 测试生成 Skill

```markdown
---
name: generate-tests
description: 自动生成测试用例
type: skill
trigger:
  - "gen-test"
  - "generate-test"
version: "1.0.0"
tags: [testing, automation]
agent:
  model: sonnet
  max_tokens: 8192
requires:
  tools: [Read, Write, Glob]
---

# 测试生成

## 功能
根据源代码自动生成单元测试和集成测试

## 触发方式
```
/skill generate-tests <file> [--framework jest] [--coverage 80]
```

## 支持框架

| 框架 | 测试文件模式 |
|------|-------------|
| `jest` | *.test.ts |
| `vitest` | *.spec.ts |
| `pytest` | test_*.py |
| `go test` | *_test.go |

## 工作流程

1. **分析源码**: 解析函数签名和逻辑
2. **识别边界**: 找出边界条件和异常情况
3. **生成用例**: 创建测试用例
4. **执行验证**: 运行测试确保通过
5. **报告覆盖率**: 输出测试覆盖率

## 示例

```
/skill generate-tests src/utils.ts --framework jest --coverage 80
```

## 注意事项
- 复杂函数可能需要手动补充边界用例
- 覆盖率目标建议 70-80%
```

---

## 4. Skill 与 Hook 的配合

### 4.1 配合模式

```
Hook → 拦截/准备 → Skill → 执行 → Hook → 验证/记录
```

### 4.2 实际场景

```markdown
---
name: safe-deploy
description: 安全部署工作流
type: skill
trigger:
  - "safe-deploy"
  - "deploy"
version: "1.0.0"
tags: [deployment, safety]

# Hook 前置检查
hooks:
  pre: deploy-check        # 部署前检查
  post: deploy-notify      # 部署后通知

# 工作流
workflow:
  parallel: false
  retry: 2
  timeout: 600000
---

# 安全部署

## 功能
包含完整检查的部署工作流

## 工作流程

1. **Hook: 部署前检查**
   - 运行测试
   - 检查 lint
   - 验证环境变量

2. **Skill: 执行部署**
   - 构建镜像
   - 推送到仓库
   - 滚动更新

3. **Hook: 部署后通知**
   - 发送 Slack 通知
   - 记录部署日志
   - 监控健康状态

## 示例

```
/skill safe-deploy production
```

## 注意事项
- 部署到生产需要确认
- 自动回滚如果健康检查失败
```

### 4.3 组合示例

```yaml
# Skill 组合配置
workflow:
  - skill: plan              # 先规划
  - skill: team              # 再分发任务
    agents: 3
  - skill: ultraqa           # 最后质量验证
```

---

## 5. Skill 调试与优化

### 5.1 调试技巧

```markdown
---
name: debug-skill
description: 调试用 Skill
debug: true                  # 开启调试模式
verbose: true                # 详细输出
---

# 添加调试信息
## 工作流程
1. 打印输入参数: $ARGUMENTS
2. 打印中间状态
3. 打印最终结果
```

### 5.2 性能优化

| 优化点 | 方法 |
|--------|------|
| 减少 Token | 使用 `context: inline` 减少上下文 |
| 加快响应 | 设置合理的 `max_tokens` |
| 避免重复 | 利用记忆系统存储状态 |

### 5.3 常见错误

| 错误 | 解决方案 |
|------|----------|
| Skill 不触发 | 检查 `trigger` 拼写和格式 |
| 参数解析失败 | 确认 `arguments` 字段定义 |
| 工具权限不足 | 添加 `requires.tools` 配置 |

---

## 6. 发布与分享

### 6.1 发布格式

```
skill-name/
├── skill.md           # 主文件
├── README.md          # 使用文档
├── examples/          # 示例
└── tests/             # 测试用例
```

### 6.2 分享平台

| 平台 | 说明 |
|------|------|
| GitHub Gist | 快速分享单个 Skill |
| npm | 发布为包 |
| OMC Skill Registry | OMC 官方注册表 |

---

## 相关章节

- [[15-01-⚡-ClaudeCode插件推荐]] - 插件生态
- [[15-03-🪝-自定义Hook配置]] - Hook 配置
- [[15-05-🏗️-整体架构建议]] - 项目结构
- [[15-06-📋-Step-by-Step教程]] - 逐步搭建
- [[../08-Skill系统/08-01-📚-Skill系统]] - Skill 系统详解

---

> [!cite]- 知识来源
>
> | 知识点 | 来源 |
> |-------|------|
> | **Skill 开发** | [everything-claude-code](https://github.com/everything-claude-code/everything-claude-code) |
> | **OMC Skills** | [oh-my-claudecode](https://github.com/oh-my-claudecode/oh-my-claudecode) |
> | **Superpowers** | [superpowers](https://github.com/superpowers/superpowers) |
