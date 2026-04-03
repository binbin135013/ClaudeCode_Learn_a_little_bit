---
type: method
tags: [phase-gate, quality-gate, review, workflow, omc]
description: 阶段门禁审查系统 - 软件开发关键节点的质量把关机制
---

# ⏸️ Phase-Gate 门禁审查

> ⏱ 难度: ★★★ | 重要性: ★★★ | 推荐学习时间: 45分钟
> 💡 **适用场景**: 复杂项目质量控制、团队协作审查、OMC工作流集成。我自己做的一个SKILL

---

## 概述

Phase-Gate（阶段门禁）是软件开发中的**关键节点质量审查机制**。在每个阶段结束时，必须通过预设标准的审查，才能进入下一阶段。

### 核心问题

```
为什么需要门禁审查？
├── 防止问题向后传递（越早发现，成本越低）
├── 确保每个阶段交付物质量达标
├── 提供明确的阶段边界和决策点
└── 强制团队停下来审视进度和质量
```

### Phase-Gate vs Continuous Delivery

| 维度 | Phase-Gate | Continuous Delivery |
|------|------------|---------------------|
| **审查时机** | 阶段结束时 | 持续进行 |
| **通过标准** | 明确门禁条件 | 自动化测试通过 |
| **适用场景** | 复杂项目、合规要求 | 敏捷项目、快速迭代 |
| **反馈周期** | 长（阶段间隔） | 短（每次提交） |

---

## 一、门禁机制原理

### 1.1 五阶段模型

```
┌─────────┐    Gate 1    ┌─────────┐    Gate 2    ┌─────────┐
│  需求   │─────────────►│   设计   │─────────────►│  开发   │
│  Phase  │   需求评审    │  Phase  │   设计评审    │  Phase  │
└─────────┘              └─────────┘              └─────────┘
                                                       │
                           Gate 3                       │
                          测试评审 ◄────────────────────┘
                               │
                           Gate 4
                          部署评审 ◄────────────────────┘
                               │
                           Gate 5
                         上线评审 ◄────────────────────┘
```

### 1.2 门禁检查点

| 门禁 | 名称 | 检查内容 | 通过标准 |
|------|------|---------|---------|
| **Gate 1** | 需求门禁 | 需求完整性 | 需求文档评审通过 |
| **Gate 2** | 设计门禁 | 架构合理性 | 设计文档评审通过 |
| **Gate 3** | 开发门禁 | 代码质量达标 | 代码审查通过 |
| **Gate 4** | 测试门禁 | 测试覆盖充分 | 测试通过率 100% |
| **Gate 5** | 部署门禁 | 部署就绪 | 部署检查清单完成 |

### 1.3 门禁状态

```
┌─────────────────────────────────────────┐
│              Gate 状态机                 │
├─────────────────────────────────────────┤
│                                         │
│  [Pending] ──检查开始──► [InReview]      │
│      │                      │           │
│      │                 审查通过          │
│      │                      │           │
│      │                      ▼           │
│      │               [Approved]         │
│      │                      │           │
│      │     审查不通过        │           │
│      │                      ▼           │
│      └──────────► [Rejected]            │
│                          │              │
│                   重新提交审查           │
│                          │              │
│                          ▼              │
│                    [Pending]            │
└─────────────────────────────────────────┘
```

---

## 二、触发条件

### 2.1 自动触发

当满足以下条件时，自动触发门禁审查：

```
自动触发条件
├── 阶段完成标记（commit message 规范）
├── 关键文件变更（设计文档、测试报告）
├── 分支合并请求（Pull Request 创建）
└── 定时检查（每日/每周）
```

#### Git Hook 触发示例

```bash
# .git/hooks/prepare-commit-msg
case "$2" in
  merge|multisite|scrub)
    echo "merge commit - triggering Gate check"
    omc phase-gate --auto
    ;;
esac
```

### 2.2 手动触发

```bash
# 触发当前阶段门禁审查
/omc phase-gate

# 触发指定阶段门禁
/omc phase-gate --phase design

# 强制重新审查
/omc phase-gate --force
```

### 2.3 OMC 工作流触发

在 OMC Skill 执行完成后自动触发门禁：

```
Skill 执行完成
    │
    ▼
OMC 检测阶段标记
    │
    ▼
自动触发对应 Gate
    │
    ├── 通过 → 进入下一阶段
    └── 不通过 → 停留 + 通知
```

---

## 三、配置方法

### 3.1 项目级配置

在项目根目录创建 `.phase-gates.json`：

```json
{
  "version": "1.0",
  "gates": {
    "requirements": {
      "name": "需求门禁",
      "criteria": [
        { "type": "file", "path": "docs/requirements.md", "required": true },
        { "type": "file", "path": "docs/acceptance.md", "required": true },
        { "type": "check", "name": "需求评审", "approved_by": 2 }
      ]
    },
    "design": {
      "name": "设计门禁",
      "criteria": [
        { "type": "file", "path": "docs/architecture.md", "required": true },
        { "type": "file", "path": "docs/api-spec.md", "required": true },
        { "type": "check", "name": "设计评审", "approved_by": 2 }
      ]
    },
    "development": {
      "name": "开发门禁",
      "criteria": [
        { "type": "test", "threshold": 80, "type": "coverage" },
        { "type": "test", "threshold": 0, "type": "failures" },
        { "type": "lint", "required": true }
      ]
    },
    "testing": {
      "name": "测试门禁",
      "criteria": [
        { "type": "test", "threshold": 100, "type": "pass_rate" },
        { "type": "test", "threshold": 90, "type": "coverage" }
      ]
    },
    "deployment": {
      "name": "部署门禁",
      "criteria": [
        { "type": "checklist", "path": ".deploy-checklist.md" },
        { "type": "approval", "role": "tech-lead" }
      ]
    }
  }
}
```

### 3.2 团队级配置

在 `~/.config/phase-gates/default.json` 定义团队通用标准：

```json
{
  "version": "1.0",
  "defaults": {
    "approval_threshold": 2,
    "review_timeout_hours": 48,
    "auto_escalate_after_hours": 72
  },
  "criteria_templates": {
    "code_coverage": {
      "critical": 90,
      "high": 80,
      "medium": 70
    },
    "review_approval": {
      "architect": 1,
      "tech_lead": 1,
      "peer": 1
    }
  }
}
```

### 3.3 OMC 集成配置

在 OMC 工作流中启用 Phase-Gate：

```json
// .omc/workflows/standard.json
{
  "name": "标准开发流程",
  "phases": [
    {
      "name": "需求分析",
      "skill": "deep-interview",
      "gate": {
        "enabled": true,
        "criteria": "requirements",
        "auto_pass_if": ["需求文档完成"]
      }
    },
    {
      "name": "技术设计",
      "skill": "plan",
      "gate": {
        "enabled": true,
        "criteria": "design"
      }
    },
    {
      "name": "开发实现",
      "skill": "autopilot",
      "gate": {
        "enabled": true,
        "criteria": "development"
      }
    },
    {
      "name": "测试验证",
      "skill": "ultraqa",
      "gate": {
        "enabled": true,
        "criteria": "testing"
      }
    }
  ]
}
```

---

## 四、门禁标准示例

### 4.1 需求门禁标准

| 检查项 | 说明 | 必须通过 |
|--------|------|---------|
| 需求文档存在 | `docs/requirements.md` | ✓ |
| 验收标准存在 | `docs/acceptance.md` | ✓ |
| 需求评审通过 | 至少 2 人评审 | ✓ |
| 需求可测试 | 每条需求有对应测试用例 | ✓ |
| 风险评估完成 | 已识别并评估风险 | ○ |

### 4.2 设计门禁标准

| 检查项 | 说明 | 必须通过 |
|--------|------|---------|
| 架构文档存在 | `docs/architecture.md` | ✓ |
| API 规范存在 | `docs/api-spec.md` | ✓ |
| 设计评审通过 | 至少 2 人评审（含架构师） | ✓ |
| 性能目标明确 | QPS、延迟指标定义 | ○ |
| 安全设计完成 | 认证、授权、数据保护 | ○ |

### 4.3 开发门禁标准

| 检查项 | 说明 | 阈值 |
|--------|------|------|
| 代码覆盖率 | 整体覆盖率达到标准 | ≥ 80% |
| 单元测试通过 | 所有单元测试通过 | 100% |
| Lint 检查通过 | 代码风格合规 | 必须通过 |
| 无高危漏洞 | 安全扫描无高危 | 必须通过 |
| 依赖无警告 | 依赖审计无严重警告 | 必须通过 |

### 4.4 测试门禁标准

| 检查项 | 说明 | 阈值 |
|--------|------|------|
| 功能测试通过 | 所有功能测试通过 | 100% |
| 集成测试通过 | 集成测试通过率 | ≥ 95% |
| E2E 测试通过 | 端到端测试通过 | 100% |
| 测试覆盖率 | 核心路径覆盖 | ≥ 90% |
| Bug 修复率 | 已知 Bug 修复 | ≥ 90% |

### 4.5 部署门禁标准

| 检查项 | 说明 | 必须通过 |
|--------|------|---------|
| 部署检查清单 | `.deploy-checklist.md` 完成 | ✓ |
| 技术负责人审批 | Tech Lead 确认 | ✓ |
| 回滚方案就绪 | 回滚计划已准备 | ✓ |
| 监控告警配置 | 关键指标已配置 | ✓ |
| 值班人员确认 | 有人值班应对问题 | ○ |

---

## 五、OMC 工作流集成

### 5.1 自动门禁触发

在 OMC 中，Phase-Gate 与 Skill 系统深度集成：

```
┌─────────────────────────────────────────────────────────┐
│                    OMC Phase-Gate 流程                    │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  /deep-interview 需求澄清                                │
│       │                                                 │
│       ▼                                                 │
│  Gate 1: 需求门禁 ──通过──► 继续                         │
│       │                                                 │
│       ▼                                                 │
│  /plan 技术规划                                          │
│       │                                                 │
│       ▼                                                 │
│  Gate 2: 设计门禁 ──通过──► 继续                         │
│       │                                                 │
│       ▼                                                 │
│  /autopilot 开发实现                                     │
│       │                                                 │
│       ▼                                                 │
│  Gate 3: 开发门禁 ──通过──► 继续                         │
│       │                                                 │
│       ▼                                                 │
│  /ultraqa 测试验证                                       │
│       │                                                 │
│       ▼                                                 │
│  Gate 4: 测试门禁 ──通过──► 继续                         │
│       │                                                 │
│       ▼                                                 │
│  部署上线                                                │
│       │                                                 │
│       ▼                                                 │
│  Gate 5: 部署门禁 ──通过──► 完成                         │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 5.2 门禁跳过条件

```yaml
# .omc/phase-gate-overrides.yaml
skip_gate:
  # 紧急修复可跳过某些门禁
  emergency_hotfix:
    skip_gates: [Gate3, Gate4]
    reason: "紧急安全修复"
    require_approval: tech-lead

  # 小改动可简化门禁
  trivial_change:
    skip_gates: [Gate1, Gate2]
    criteria:
      files_changed: < 5
      coverage_delta: > -5
```

### 5.3 门禁失败处理

```
门禁失败
    │
    ▼
1. 分析失败原因
    │
    ├── 文档缺失 → 补充文档
    ├── 覆盖率不足 → 增加测试
    ├── 评审未通过 → 修改 + 重新评审
    └── 超时 → 升级处理
    │
    ▼
2. 执行修复
    │
    ▼
3. 重新触发门禁
    │
    ▼
4. 记录失败日志
```

---

## 六、最佳实践

### 6.1 门禁配置原则

```
门禁配置原则
├── 门禁不是障碍，是保护
├── 标准要具体、可衡量
├── 太少门禁 = 质量风险
├── 太多门禁 = 效率瓶颈
└── 根据项目规模调整
```

### 6.2 项目规模与门禁策略

| 项目规模 | 推荐门禁数量 | 说明 |
|---------|-------------|------|
| **小型** | 2-3 个 | 需求-上线，其他简化 |
| **中型** | 3-4 个 | 需求-设计-测试-上线 |
| **大型** | 4-5 个 | 完整五阶段 |
| **超大型** | 5+ 个 | 每阶段可细分 |

### 6.3 常见问题处理

| 问题 | 解决方案 |
|------|---------|
| 门禁总是阻塞进度 | 简化门禁标准或并行处理 |
| 评审人员不足 | 指定备选评审人 |
| 标准无法量化 | 转为 Checklist 主观评估 |
| 频繁超时 | 设置自动升级机制 |

---

## 七、相关章节

- [[14-01-🎯-Skill选择决策树]] - 选择合适的 Skill 配合门禁
- [[14-02-🔄-Ralph循环最佳实践]] - 迭代优化配合门禁
- [[../12-质量保证/12-01-🧪-ultraqa循环]] - 测试验证门禁标准
- [[../09-子Agent与协作/09-01-🤝-子Agent与协作]] - 团队评审协作
- [[../13-OMC完全指南/13-03-🔄-OMC工作流]] - OMC 工作流集成

---

## 八、知识来源

> [!cite]- 知识来源
>
> 本文档核心内容来源：
>
> | 知识点 | 来源 |
> |-------|------|
> | **五阶段模型** | 软件工程经典理论 (Royce, 1970) |
> | **门禁机制原理** | CMMI 阶段表示法 + IEEE 1012 |
> | **OMC 集成方案** | oh-my-claudecode 官方文档 |
> | **配置格式** | 基于 JSON Schema 标准 |
>
> **适用版本**: OMC v1.0+

---

*最后更新：2026-04-03*
