---
type: best-practice
tags: [claude-code, claude-md, production, best-practices]
description: CLAUDE.md 高级模式 - 生产级项目配置指南
---

# CLAUDE.md 高级模式

> ⏱ 适用水平: 进阶 | 推荐程度: 必读

---

## 1. CLAUDE.md 基础回顾

### 1.1 什么是 CLAUDE.md

CLAUDE.md 是 Claude Code 的**项目级配置文件**，位于项目根目录，用于定义项目特有的指令、偏好设置和工作流程。与全局配置 `.claude/` 目录配合使用，实现项目级别的定制化。

### 1.2 核心作用

| 作用 | 说明 |
|-----|------|
| **项目上下文** | 定义项目结构、技术栈、业务领域 |
| **编码规范** | 统一代码风格、命名约定、提交规范 |
| **工作流程** | 定义开发流程、验证步骤、发布流程 |
| **团队协作** | 共享配置、角色定义、环境隔离 |

### 1.3 基本结构

```yaml
# 项目描述
# 项目技术栈
# 项目结构

## 开发规范
## 工作流程
## 验证标准
```

---

## 2. 项目级配置模式

### 2.1 permissions 块配置

```yaml
# 权限配置示例
permissions:
  # 允许的文件操作范围
  allow:
    - "**/*"           # 全部文件
    - "src/**/*"       # 源码目录
    - "*.{js,ts,py}"   # 指定扩展名
  # 禁止的操作
  deny:
    - "**/.env"        # 敏感文件
    - "**/secrets.*"   # 密钥文件
  # 只读模式（生产环境推荐）
  readOnly: false
```

### 2.2 commands 自定义命令

```yaml
commands:
  # 代码检查
  lint:
    description: "运行 ESLint 检查"
    command: "npm run lint"
    workingDirectory: "."

  # 测试
  test:
    description: "运行单元测试"
    command: "npm test -- --coverage"
    workingDirectory: "."

  # 构建
  build:
    description: "构建生产版本"
    command: "npm run build"
    workingDirectory: "."

  # 部署
  deploy:
    description: "部署到生产环境"
    command: "./scripts/deploy.sh prod"
    requireConfirmation: true
```

### 2.3 mcpServers 配置

```yaml
mcpServers:
  # 文件系统增强
  filesystem:
    command: "npx"
    args: ["-y", "@modelcontextprotocol/server-filesystem", "./src"]

  # Git 操作
  git:
    command: "npx"
    args: ["-y", "@modelcontextprotocol/server-git"]

  # GitHub 集成
  github:
    command: "npx"
    args: ["-y", "@modelcontextprotocol/server-github"]
    env:
      GITHUB_TOKEN: "${GITHUB_TOKEN}"

  # 数据库连接（示例）
  database:
    command: "npx"
    args: ["-y", "@modelcontextprotocol/server-database"]
    env:
      DATABASE_URL: "${DATABASE_URL}"
```

### 2.4 featureFlags 使用

```yaml
featureFlags:
  # 实验性功能
  experimental:
    multiAgent: true
    parallelExecution: false
    advancedContext: true

  # 稳定性功能
  stable:
    autoVerify: true
    incrementalBuild: true
    smartSuggestions: true

  # 调试功能（仅开发环境）
  debug:
    verboseLogging: false
    traceMode: false
    performanceMetrics: false
```

---

## 3. 团队协作模式

### 3.1 多环境配置（dev/staging/prod）

```yaml
# 环境特定配置
environments:
  development:
    permissions:
      allow: ["**/*"]
      deny: []
    mcpServers:
      filesystem:
        enabled: true
      debug:
        enabled: true

  staging:
    permissions:
      allow: ["src/**/*", "tests/**/*"]
      deny: ["**/.env", "**/credentials.json"]
    mcpServers:
      filesystem:
        enabled: true
      github:
        enabled: true

  production:
    permissions:
      allow: ["src/**/*"]
      deny: ["**/*"]
      readOnly: true
    mcpServers:
      filesystem:
        enabled: false
      github:
        enabled: true
        env:
          GITHUB_TOKEN: "${PROD_GITHUB_TOKEN}"
```

### 3.2 环境变量注入

```yaml
# 环境变量配置
env:
  # 开发环境变量（本地）
  local:
    DATABASE_URL: "postgresql://localhost:5432/dev"
    API_KEY: "${LOCAL_API_KEY}"

  # 测试环境变量
  test:
    DATABASE_URL: "${TEST_DATABASE_URL}"
    API_KEY: "${TEST_API_KEY}"

  # 生产环境变量（敏感）
  production:
    DATABASE_URL: "${PROD_DATABASE_URL}"
    API_KEY: "${PROD_API_KEY}"
    STRIPE_SECRET: "${STRIPE_SECRET}"
    JWT_SECRET: "${JWT_SECRET}"

# 敏感信息处理
security:
  maskInLogs: true
  envFiles:
    - ".env.local"
    - ".env.production"
  requiredVars:
    - "DATABASE_URL"
    - "API_KEY"
```

### 3.3 共享 vs 项目级配置

```yaml
# === 全局共享配置（~/.claude/CLAUDE.md）===
shared:
  # 所有项目通用的规范
  encoding: "UTF-8"
  lineEnding: "LF"
  indent: "spaces"
  indentSize: 2

  # 通用工具链
  tools:
    formatter: "prettier"
    linter: "eslint"
    testRunner: "jest"

# === 项目特定配置（项目内 CLAUDE.md）===
project:
  # 项目特有的配置会覆盖全局配置
  indentSize: 4
  testRunner: "vitest"

  # 项目特定规范
  architecture: "microservices"
  primaryLanguage: "typescript"
  frameworks:
    - "react"
    - "next.js"
```

---

## 4. 高级指令模式

### 4.1 System Prompt 优化

```yaml
# 角色定义
role: |
  你是一个专注于 [领域] 的 [职位]。

  核心职责：
  - [职责1]
  - [职责2]
  - [职责3]

  工作原则：
  1. 始终优先考虑代码可维护性
  2. 遵循 SOLID 原则
  3. 提供详细的代码注释

# 专业领域上下文
context: |
  本项目是 [项目类型]，主要解决 [业务问题]。

  技术栈：
  - 前端：[技术1] + [技术2]
  - 后端：[技术3] + [技术4]
  - 数据库：[数据库类型]
  - 部署：[部署方式]

  关键约束：
  - 性能要求：[具体指标]
  - 安全要求：[认证/授权要求]
  - 合规要求：[法规要求]
```

### 4.2 示例注入 (Examples)

```yaml
examples:
  # 代码示例
  code:
    - description: "正确的错误处理"
      snippet: |
        try {
          const result = await api.call();
          return result;
        } catch (error) {
          logger.error('API调用失败', { error, context });
          throw new AppError('SERVICE_UNAVAILABLE', error);
        }

    - description: "类型定义规范"
      snippet: |
        interface User {
          id: string;
          email: string;
          role: 'admin' | 'user' | 'guest';
          createdAt: Date;
        }

  # 工作流程示例
  workflow:
    - description: "功能开发流程"
      steps:
        - 理解需求并提出问题
        - 编写测试用例
        - 实现功能代码
        - 运行测试验证
        - 提交代码并描述变更

  # 最佳实践示例
  bestPractice:
    - description: "API 设计规范"
      rules:
        - 使用 RESTful 风格
        - 版本控制：/api/v1/resource
        - 统一响应格式：{ success, data, error }
```

### 4.3 条件指令 ([if], [unless])

```yaml
# 条件指令示例
conditions:
  # 语言特定的指令
  when:
    language: "typescript":
      rules:
        - "使用严格的类型检查"
        - "优先使用 interface 而不是 type"
        - "导出类型而非实现"

    language: "python":
      rules:
        - "遵循 PEP 8 规范"
        - "使用类型注解"
        - "优先使用 dataclass"

  # 环境特定的指令
  environment:
    production:
      constraints:
        - "禁止使用 console.log"
        - "所有 API 调用必须重试"
        - "启用详细错误日志"

    development:
      relaxations:
        - "允许 console.log 用于调试"
        - "API 重试次数减少"
        - "启用详细错误堆栈"

  # 文件类型的条件指令
  fileType:
    "*.test.ts":
      directives:
        - "测试文件放在 __tests__ 目录"
        - "使用 describe/it 结构"
        - "每个测试用例独立"

    "*.config.js":
      directives:
        - "配置文件需要注释"
        - "使用环境变量而非硬编码"
```

---

## 5. 最佳实践

### 5.1 安全注意事项

```yaml
security:
  # 敏感信息处理
  sensitiveData:
    patterns:
      - "**/.env*"
      - "**/credentials.json"
      - "**/secrets.yaml"
      - "**/*.pem"
    action: "deny"

  # 权限最小化
  principle: "least-privilege"
  defaultPermissions:
    read: false
    write: false
    execute: false

  # 审计日志
  audit:
    enabled: true
    logFile: ".claude/audit.log"
    includeCommands: true
    includeFileAccess: true
```

### 5.2 性能优化建议

```yaml
performance:
  # 上下文优化
  context:
    maxTokens: 100000
    compressionEnabled: true
    priorityFiles:
      - "src/**/*.ts"
      - "package.json"
      - "CLAUDE.md"

  # 缓存策略
  cache:
    enabled: true
    ttl: 3600
    exclude:
      - "*.log"
      - "*.tmp"

  # 并行处理
  parallel:
    enabled: true
    maxWorkers: 4
    strategies:
      - "文件并行处理"
      - "测试并行运行"
      - "构建分布执行"
```

### 5.3 调试技巧

```yaml
debugging:
  # 日志级别
  logging:
    level: "info"        # trace, debug, info, warn, error
    categories:
      - "tool_calls"
      - "file_operations"
      - "command_execution"

  # 追踪配置
  tracing:
    enabled: false
    output: ".claude/trace.log"
    includeContext: true

  # 诊断工具
  diagnostics:
    showTokenCount: true
    showFileChanges: true
    showExecutionTime: true
```

---

## 6. 完整配置示例

```yaml
# ========================================
# 生产项目 CLAUDE.md 完整配置
# ========================================

---
type: production-config
tags: [production, enterprise, typescript]
description: 企业级 TypeScript 项目配置
---

# 项目描述
name: "企业级后台管理系统"
version: "2.0.0"
environment: "production"

# 权限配置
permissions:
  allow:
    - "src/**/*"
    - "tests/**/*"
    - "scripts/**/*"
  deny:
    - "**/.env*"
    - "**/secrets.*"
    - "**/credentials.json"
  readOnly: false

# 自定义命令
commands:
  dev: "npm run dev"
  build: "npm run build"
  test: "npm test -- --coverage"
  lint: "npm run lint"
  deploy: "./scripts/deploy.sh"

# MCP Servers
mcpServers:
  github:
    enabled: true
    env:
      GITHUB_TOKEN: "${GITHUB_TOKEN}"

# 工作流程
workflow:
  commit:
    convention: "Conventional Commits"
    scopes: ["feat", "fix", "docs", "style", "refactor", "test", "chore"]
  review:
    requiredApprovals: 2
    autoMerge: false

# 验证标准
verification:
  testCoverage: 80
  lintPass: true
  buildPass: true
```

---

## 相关章节

- [[../03-权限系统/03-01-🔐-权限系统]] - 权限配置
- [[../15-ClaudeCode最佳实践/15-01-⚡-ClaudeCode插件推荐]] - 插件推荐
- [[../08-Skill系统/08-01-📚-Skill系统]] - 技能开发
- [[../15-ClaudeCode最佳实践/15-03-🪝-自定义Hook配置]] - Hook 配置

---

> [!cite]- 知识来源
>
> | 知识点 | 来源 |
> |-------|------|
> | **CLAUDE.md 官方文档** | [Anthropic 官方](https://docs.anthropic.com/) |
> | **项目配置最佳实践** | [Claude Code Best Practices](https://github.com/claude-code-best/claude-code) |
> | **团队协作模式** | [oh-my-claudecode](https://github.com/oh-my-claudecode/oh-my-claudecode) |
