---
type: system
tags: [claude-code, cicd, automation, github-actions]
description: Claude Code GitHub Actions集成与自动化
---

# CI/CD 与自动化

> 难度: ★★☆ | 重要性: ★★☆ | 推荐学习时间: 2-3天
> 💡 **适用场景**: 团队协作时需要自动化流程

---

## 概述

### GitHub Actions 集成

将 Claude Code 集成到 GitHub Actions 等 CI/CD 系统中，实现自动代码审查、安全扫描、文档生成。

### 官方 Action

`anthropics/claude-code-action@v1`

> 📌 **官方文档**: https://github.com/anthropics/claude-code-action
> - 完整参数列表见 [action.yml](https://github.com/anthropics/claude-code-action/blob/main/action.yml)
> - 最佳实践见 [Solutions Guide](https://github.com/anthropics/claude-code-action/blob/main/docs/solutions.md)

---

## 完整参数列表

`claude-code-action` 支持以下输入参数：

| 参数 | 必填 | 默认值 | 说明 |
|------|------|--------|------|
| `anthropic_api_key` | 条件 | - | Anthropic API 密钥（直接 API 方式） |
| `github_token` | 建议 | - | GitHub Token，建议显式传入 |
| `prompt` | 条件 | - | 传给 Claude 的指令 |
| `claude_args` | 否 | - | 传给 Claude CLI 的额外参数 |
| `settings` | 否 | - | Claude Code 设置（JSON） |
| `model` | 否 | - | 指定模型（如 `claude-sonnet-4-6`） |
| `trigger_phrase` | 否 | `@claude` | 触发短语 |
| `label_trigger` | 否 | `claude` | 触发标签 |
| `allowed_bots` | 否 | - | 允许的 Bot 用户名列表 |
| `allowed_non_write_users` | 否 | - | 无写权限但允许触发的用户 |
| `use_sticky_comment` | 否 | `false` | 是否使用单条评论聚合 |
| `track_progress` | 否 | `false` | 是否在 PR 中生成任务进度追踪 |
| `include_fix_links` | 否 | `true` | 是否在审查反馈中包含 "Fix this" 链接 |
| `plugins` | 否 | - | 插件列表（换行分隔） |
| `plugin_marketplaces` | 否 | - | 插件市场 Git URL 列表 |
| `use_commit_signing` | 否 | `false` | 启用提交签名 |
| `ssh_signing_key` | 否 | - | SSH 私钥用于提交签名 |
| `show_full_output` | 否 | `false` | 显示完整 JSON 输出（慎用，公开可见） |
| `display_report` | 否 | `false` | 在 Step Summary 中显示报告 |
| `path_to_claude_code_executable` | 否 | - | 自定义 Claude Code 可执行文件路径 |
| `additional_permissions` | 否 | - | 额外请求的 GitHub 权限 |
| `include_comments_by_actor` | 否 | - | 仅包含指定用户的评论 |
| `exclude_comments_by_actor` | 否 | - | 排除指定用户的评论 |
| `base_branch` | 否 | - | 用作 base/source 的分支 |
| `branch_prefix` | 否 | `claude/` | Claude 创建分支的前缀 |
| `use_bedrock` | 否 | `false` | 使用 AWS Bedrock（OIDC 认证） |
| `use_vertex` | 否 | `false` | 使用 Google Vertex AI（OIDC 认证） |
| `use_foundry` | 否 | `false` | 使用 Microsoft Foundry（OIDC 认证） |

---

## 完整 Workflow 示例

### 1. 基础代码审查工作流

```yaml
name: Claude Code Review

# 在 PR 打开、同步、重新打开时触发
on:
  pull_request:
    types: [opened, synchronize, reopened]
  # 也可以在 issue 评论时触发
  issue_comment:
    types: [created, edited]

# 最小权限原则：只申请用到的权限
permissions:
  contents: read          # 读取仓库内容（用于代码审查）
  pull-requests: write   # 写入 PR 评论
  issues: write           # 写入 Issue 评论
  # 如需创建分支/提交代码，取消注释：
  # contents: write

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      # 必须：检出代码，fetch-depth: 0 拉取完整历史以便 Diff 分析
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      # 必须：运行 Claude Code Action
      - uses: anthropics/claude-code-action@v1
        with:
          # 认证：优先使用 GitHub App Token（安全），其次 secrets
          github_token: ${{ secrets.GITHUB_TOKEN }}

          # 指定模型（可选，不指定则用默认）
          # claude_args: --model claude-sonnet-4-6

          # 审查提示词（核心配置）
          prompt: |
            You are a senior code reviewer. Analyze the pull request changes for:
            1. Code quality and best practices
            2. Potential bugs or logic errors
            3. Security vulnerabilities (injection, XSS, hardcoded secrets)
            4. Performance issues
            5. Test coverage

            Provide a structured review with:
            - Summary of changes
            - Issues found (severity: critical/high/medium/low)
            - Specific recommendations
            - Approval or request for changes

          # 额外 Claude CLI 参数
          claude_args: >
            --dangerously-skip-permissions
            --no-input

          # 进度追踪：开启任务列表更新
          track_progress: true

          # Fix this 链接：让审查结果可直接点击打开 Claude 修复
          include_fix_links: true
        env:
          # API Key（直接 API 方式需要）
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
```

### 2. 完整多阶段流水线

```yaml
name: CI Pipeline with Claude Review

on:
  push:
    branches: [main, develop]
  pull_request:
    types: [opened, synchronize, reopened]

permissions:
  contents: read
  pull-requests: write
  security-events: write   # 安全扫描结果写入
  actions: read            # 读取 workflow 运行状态

env:
  NODE_VERSION: "20"
  PYTHON_VERSION: "3.11"

jobs:
  # ============================================
  # Stage 1: 代码检查（Lint + Format）
  # ============================================
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}

      - name: Install dependencies
        run: npm ci

      - name: Run ESLint
        run: npm run lint

      - name: Check formatting
        run: npm run format:check

  # ============================================
  # Stage 2: 单元测试
  # ============================================
  test:
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}

      - name: Install dependencies
        run: npm ci

      - name: Run tests with coverage
        run: npm run test:coverage

      - name: Upload coverage reports
        uses: codecov/codecov-action@v4
        with:
          token: ${{ secrets.CODECOV_TOKEN }}

  # ============================================
  # Stage 3: Claude Code 审查（核心）
  # ============================================
  claude-review:
    runs-on: ubuntu-latest
    needs: [lint, test]
    # 只有 PR 类型才运行 Claude 审查
    if: github.event_name == 'pull_request'
    permissions:
      contents: read
      pull-requests: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Run Claude Code Review
        uses: anthropics/claude-code-action@v1
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          prompt: |
            You are conducting a thorough code review for this pull request.

            ## Review Scope
            - Analyze all changed files
            - Focus on logic errors, bugs, and security issues
            - Check test quality and coverage
            - Verify documentation accuracy

            ## Output Format
            Provide your review as a structured comment on the PR with:
            1. **Summary**: Brief overview of changes
            2. **Critical Issues**: Must-fix problems (if any)
            3. **Suggestions**: Improvements to consider
            4. **Approve/Request Changes**: Final recommendation

          track_progress: true
          include_fix_links: true
          # 审查模式：等待审查结果后再合并
          # claude_args: --await-exit
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}

  # ============================================
  # Stage 4: 安全扫描
  # ============================================
  security:
    runs-on: ubuntu-latest
    needs: [lint, test]
    permissions:
      contents: read
      security-events: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      # 运行 Trivy 安全扫描
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: "fs"
          scan-ref: "."
          format: "sarif"
          output: "trivy-results.sarif"

      # 上报结果到 GitHub Security
      - name: Upload Trivy scan results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: "trivy-results.sarif"

      # Claude 辅助安全审查
      - name: Claude Security Review
        uses: anthropics/claude-code-action@v1
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          prompt: |
            Analyze the code changes in this PR for security vulnerabilities:

            1. **Injection Attacks**: SQL injection, command injection, XSS
            2. **Authentication Issues**: Broken auth, missing auth checks
            3. **Data Exposure**: Hardcoded secrets, exposed credentials, insecure storage
            4. **Dependency Vulnerabilities**: Known CVEs in dependencies
            5. **Input Validation**: Missing or insufficient input sanitization

            Report findings with severity levels and specific remediation suggestions.
          claude_args: --no-input
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}

  # ============================================
  # Stage 5: 构建与部署（仅 main 分支）
  # ============================================
  build-and-deploy:
    runs-on: ubuntu-latest
    needs: [claude-review, security]
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Build application
        run: npm run build

      - name: Deploy
        run: npm run deploy
        env:
          DEPLOY_TOKEN: ${{ secrets.DEPLOY_TOKEN }}
```

### 3. 代码审查工作流（PR 触发 + 自动审查）

```yaml
name: Automated PR Review

on:
  pull_request:
    types: [opened, synchronize, reopened, ready_for_review]
  # 支持 Label 触发审查
  pull_request_target:
    types: [labeled]

permissions:
  contents: read
  pull-requests: write
  # 用于写入审查结果和状态
  statuses: write

jobs:
  # ============================================
  # 自动代码审查
  # ============================================
  auto-review:
    runs-on: ubuntu-latest
    # 只在 PR 准备好审查时运行
    if: github.event_name == 'pull_request'
    permissions:
      contents: read
      pull-requests: write
    steps:
      - uses: actions/checkout@v4
        with:
          # fetch-depth: 1 对于仅 Diff 分析足够
          fetch-depth: 0

      - name: Run Claude Code Review
        uses: anthropics/claude-code-action@v1
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}

          # 针对 PR 事件的自动审查提示词
          prompt: |
            You are an automated code reviewer for pull requests.

            ## Your Task
            Review the code changes in this PR thoroughly and provide a detailed analysis.

            ## Review Criteria
            1. **Correctness**: Does the code do what it's supposed to do?
            2. **Code Quality**: Are there code smells, anti-patterns, or style violations?
            3. **Test Coverage**: Are there adequate tests for the changes?
            4. **Security**: Any security concerns?
            5. **Performance**: Any obvious performance issues?

            ## Output Instructions
            - Post a review comment on the PR with your findings
            - Use Markdown formatting for readability
            - Mark critical issues clearly
            - Provide actionable suggestions

          # 开启进度追踪，生成任务列表
          track_progress: true

          # 聚合为单条评论（避免刷屏）
          use_sticky_comment: true

          # 内联审查（代码行级别评论）
          classify_inline_comments: true
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}

  # ============================================
  # 按 Label 触发的专项审查
  # ============================================
  labeled-review:
    runs-on: ubuntu-latest
    # 仅在添加特定 Label 时触发
    if: |
      github.event_name == 'pull_request_target' &&
      github.event.label.name == 'claude-review'
    permissions:
      contents: read
      pull-requests: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
          # 对于 pull_request_target，使用指定分支
          ref: ${{ github.event.pull_request.head.ref }}

      - name: Run Claude Deep Review
        uses: anthropics/claude-code-action@v1
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          prompt: |
            Perform a deep review of this PR with focus on:
            - Architecture and design patterns
            - API design and backward compatibility
            - Error handling and edge cases
            - Documentation completeness

          track_progress: true
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}

  # ============================================
  # 审查通过后自动合并
  # ============================================
  auto-merge:
    runs-on: ubuntu-latest
    needs: auto-review
    # 检查审查是否通过
    if: |
      github.event_name == 'pull_request' &&
      github.event.pull_request.merged != true
    permissions:
      contents: write
      pull-requests: write
    steps:
      - name: Check approval status
        run: |
          # 检查是否有足够的审查批准
          APPROVALS=$(gh pr view ${{ github.event.pull_request.number }} --repo ${{ github.repository }} --json reviewDecision -q ".reviewDecision")
          echo "Review decision: $APPROVALS"

          # 如果 Claude 审查通过且有足够审批，则合并
          if [ "$APPROVALS" == "APPROVED" ]; then
            echo "PR approved, proceeding with merge check..."
          fi

      - name: Merge PR
        if: success()
        run: |
          gh pr merge ${{ github.event.pull_request.number }} \
            --admin --squash \
            --repo ${{ github.repository }}
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### 4. 安全扫描集成工作流

```yaml
name: Security Scanning Pipeline

on:
  push:
    branches: [main]
  pull_request:
    types: [opened, synchronize]

permissions:
  contents: read
  security-events: write
  vulnerabilities: write

jobs:
  # ============================================
  # 第一层：基础依赖扫描
  # ============================================
  dependency-scan:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      security-events: write
    steps:
      - uses: actions/checkout@v4

      # GitHub 官方依赖快照
      - name: Run GitHub Dependency Snapshot
        uses: actions/dependency-review-action@v4
        with:
          # 允许特定许可证（酌情调整）
          allow-licenses: MIT, Apache-2.0, BSD-3-Clause, ISC

      #npm audit
      - name: Run npm audit
        run: npm audit --audit-level=high
        continue-on-error: true

  # ============================================
  # 第二层：SAST 静态分析
  # ============================================
  sast-scan:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      security-events: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      # CodeQL 静态分析
      - name: Initialize CodeQL
        uses: github/codeql-action/init@v2
        with:
          languages: javascript, typescript, python
          queries: security-extended

      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v2

  # ============================================
  # 第三层：Secrets 扫描
  # ============================================
  secrets-scan:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      # TruffleHog secrets 扫描
      - name: Run TruffleHog
        uses: trufflesecurity/trufflehog@main
        with:
          path: ./
          base: ${{ github.event.repository.default_branch }}
          head: HEAD
          # 仅检测，不阻止（避免误报阻塞）
          only_verified: false

  # ============================================
  # 第四层：Claude AI 安全审查
  # ============================================
  claude-security-review:
    runs-on: ubuntu-latest
    needs: [dependency-scan, sast-scan]
    permissions:
      contents: read
      pull-requests: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Claude Security Analysis
        uses: anthropics/claude-code-action@v1
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}

          prompt: |
            You are a security expert reviewing this code change.

            ## Security Review Focus
            Analyze for:

            ### 1. Injection Vulnerabilities
            - SQL injection (parameterized queries, ORM usage)
            - Command injection (avoid shell where possible)
            - XSS (output encoding, CSP headers)
            - Path traversal

            ### 2. Authentication & Authorization
            - Missing authentication checks
            - Broken authorization logic
            - Privilege escalation risks

            ### 3. Data Protection
            - Hardcoded secrets or credentials
            - Insecure storage of sensitive data
            - Missing encryption for data at rest/transit
            - Logging of sensitive information

            ### 4. Input Validation
            - Missing input sanitization
            - Type confusion vulnerabilities
            - ReDoS (Regular Expression DoS)

            ### 5. Dependency Security
            - Known vulnerabilities in new dependencies
            - Outdated dependencies with known CVEs

            ## Output Format
            Post findings as a PR review with:
            - **Critical**: Must fix before merge
            - **High**: Should fix soon
            - **Medium**: Consider fixing
            - **Low**: Best practice improvements

            For each issue, provide:
            1. Location (file:line)
            2. Description
            3. Impact assessment
            4. Remediation suggestion
          claude_args: --no-input
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}

  # ============================================
  # 第五层：容器安全扫描（如果有 Dockerfile）
  # ============================================
  container-scan:
    runs-on: ubuntu-latest
    # 仅在有 Dockerfile 时运行
    if: contains(files, 'Dockerfile')
    permissions:
      contents: read
      security-events: write
    steps:
      - uses: actions/checkout@v4

      - name: Build Docker image
        run: docker build -t myapp:${{ github.sha }} .

      - name: Run Trivy container scan
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: "myapp:${{ github.sha }}"
          format: "sarif"
          output: "container-results.sarif"
          severity: "CRITICAL,HIGH"

      - name: Upload container scan results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: "container-results.sarif"
```

### 5. 文档自动生成工作流

```yaml
name: Auto Documentation

on:
  push:
    branches: [main]
    paths:
      # 仅在文档相关文件变更时触发
      - '**.md'
      - 'docs/**'
      # 或在 API 文件变更时触发
      - 'src/api/**'
      - 'src/**/*.openapi.json'
  # 手动触发
  workflow_dispatch:

permissions:
  contents: write
  pull-requests: write

jobs:
  # ============================================
  # API 文档生成
  # ============================================
  api-docs:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      # 生成 OpenAPI/Swagger 文档
      - name: Generate API Documentation
        run: |
          # 根据项目类型执行对应的文档生成命令
          # 示例为 Node.js/TypeScript 项目
          npm run docs:api

      - name: Commit API docs
        run: |
          git config --local user.email "github-actions[bot]@users.noreply.github.com"
          git config --local user.name "GitHub Actions Bot"
          git add docs/api/
          git diff --staged --quiet || git commit -m "docs: auto-update API documentation"
          git push

  # ============================================
  # Claude 辅助文档审查与更新
  # ============================================
  claude-doc-review:
    runs-on: ubuntu-latest
    needs: api-docs
    permissions:
      contents: read
      pull-requests: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Claude Documentation Review
        uses: anthropics/claude-code-action@v1
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}

          prompt: |
            You are a technical documentation expert.

            ## Task
            Review and update documentation in this repository:

            1. **Consistency Check**: Ensure all public APIs are documented
            2. **Accuracy**: Verify documentation matches implementation
            3. **Completeness**: Check for missing examples or explanations
            4. **Formatting**: Ensure consistent Markdown style

            ## Specific Checks
            - README.md is up to date with current features
            - API documentation matches code changes
            - Migration guide is updated for breaking changes
            - Code examples are accurate and runnable

            ## Output
            - Update documentation files directly
            - Post a summary comment on the related PR
            - Create follow-up issues for major documentation gaps
          claude_args: --no-input
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}

  # ============================================
  # CHANGELOG 自动生成
  # ============================================
  changelog:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Generate CHANGELOG
        run: |
          # 使用 standard-version 或 release-please
          npx standard-version

      - name: Create PR for changelog
        if: github.event_name != 'push'
        run: |
          git push origin HEAD:changelog-update
          gh pr create \
            --title "chore: update changelog" \
            --body "Automated CHANGELOG update" \
            --base main

      - name: Commit changelog on main
        if: github.event_name == 'push'
        run: |
          git config --local user.email "github-actions[bot]@users.noreply.github.com"
          git config --local user.name "GitHub Actions Bot"
          git add CHANGELOG.md
          git diff --staged --quiet || git commit -m "chore(release): update changelog"
          git push
```

---

## 多阶段流水线架构

```
代码检查 (Lint) → 单元测试 → Claude 审查 → 安全扫描 → 构建 → 部署
                      ↓              ↓              ↓
                 测试报告         PR 评论       安全报告
                                         ↓
                              Claude 辅助修复建议
```

### 关键 Jobs 触发条件

| 阶段 | 触发条件 | Claude 任务 |
|------|---------|------------|
| **代码审查** | PR | 质量评估、Bug 分析 |
| **安全扫描** | PR | 注入、XSS、硬编码密钥 |
| **文档生成** | Push to main | API 文档自动更新 |
| **交互命令** | @claude 评论 | 响应指令执行 |

---

## Secrets 配置

在 GitHub `Settings > Secrets` 中添加以下密钥：

| Secret 名称 | 必填 | 说明 |
|------------|------|------|
| `ANTHROPIC_API_KEY` | 直接 API 需要 | Anthropic API 密钥 |
| `GITHUB_TOKEN` | 推荐 | GitHub Token（自动提供，但建议显式使用） |
| `CODECOV_TOKEN` | 可选 | Codecov 上报令牌 |
| `DEPLOY_TOKEN` | 可选 | 部署用令牌 |

### 获取 ANTHROPIC_API_KEY

1. 访问 [ Anthropic Console ](https://console.anthropic.com/)
2. 创建 API Key
3. 在 GitHub Repository Settings > Secrets 中添加 `ANTHROPIC_API_KEY`

### 安全最佳实践

> [!warning] CI/CD 安全
> 1. API Key 只用 GitHub Secrets，不硬编码
> 2. 设置 `timeout-minutes` 防止无限运行
> 3. PR 审查结果仅供参考，人工复核必须的
> 4. 使用最小权限原则（permissions 按需分配）
> 5. `show_full_output: true` 会将完整输出暴露在公开日志中，慎用
> 6. 处理外部贡献者输入时，使用 `allowed_non_write_users` 进行隔离
> 7. `ssh_signing_key` 和 `use_commit_signing` 仅在需要签名提交时启用

---

## 与 Hooks 的关系

> [!note] 两层自动化
> - **[[../../04-Hook系统/🪝-Hook系统]]** — 本地开发时的实时自动化
> - **CI/CD** — 团队协作时的流程自动化
>
> 两者互补，Hooks 管实时，CI/CD 管流程。

---

## 常见问题

### Q: Claude 审查太慢怎么办？

A: 考虑以下优化：
- 使用 `fetch-depth: 1` 减少克隆深度
- 设置 `timeout-minutes` 限制运行时间
- 在 `prompt` 中限制审查范围（如只审查 `src/` 目录）

### Q: 如何避免评论刷屏？

A: 使用 `use_sticky_comment: true` 聚合为单条评论

### Q: 支持私有仓库吗？

A: 支持。确保正确配置 `github_token` 和 `ANTHROPIC_API_KEY`

---

## 相关章节

- [[../../01-架构总览/📐-架构概览]] - 架构概览
- [[../../04-Hook系统/🪝-Hook系统]] - Hook 系统
- [[../../03-权限系统/🔐-权限系统]] - 权限系统
- [[🏠-知识库首页]] - 返回首页
