---
type: system
tags: [superpowers, best-practices, workflow]
description: Superpowers 最佳实践 - Skill 组合、工作流编排和使用场景
---

# 🛠️ Superpowers 最佳实践

## 概述

本文档介绍如何有效地组合使用 Superpowers Skill，以及常见的使用场景。

---

## 1. 如何组合使用多个 Skill

### 1.1 Skill 组合原则

| 原则 | 说明 | 示例 |
|-----|------|-----|
| **互补性** | 技能功能互补，可互相增强 | brainstorming + tdd |
| **顺序性** | 按逻辑顺序串联使用 | tdd → systematic-debugging |
| **并行性** | 独立任务可并行执行 | subagent-driven-development |
| **循环性** | 任务形成闭环 | review → fix → review |

### 1.2 基础组合模式

#### 模式 1: 设计-实现-验证

```
brainstorming ──▶ tdd ──▶ systematic-debugging
```

**适用场景**: 新功能开发

**流程说明**:
1. `brainstorming` - 明确需求和设计方案
2. `tdd` - 测试驱动实现功能
3. `systematic-debugging` - 如遇问题则系统调试

#### 模式 2: 实现-审查-修复

```
tdd ──▶ requesting-code-review ──▶ receiving-code-review
```

**适用场景**: 代码质量保证

**流程说明**:
1. `tdd` - TDD 方式实现
2. `requesting-code-review` - 发起代码审查
3. `receiving-code-review` - 处理审查反馈

#### 模式 3: 规划-执行-验证

```
brainstorming ──▶ subagent-driven-development ──▶ systematic-debugging
```

**适用场景**: 大型复杂任务

### 1.3 Skill 链式组合

#### 完整开发链

```
brainstorming ──▶ tdd ──▶ systematic-debugging ──▶ requesting-code-review
       │              │               │                    │
       ▼              ▼               ▼                    ▼
    需求设计       红绿重构         Bug修复              发起审查
                                                              │
                                                              ▼
                                              receiving-code-review
                                                              │
                                                              ▼
                                              (如需修复) ──▶ systematic-debugging
```

#### 快速修复链

```
systematic-debugging ──▶ receiving-code-review ──▶ tdd
```

**适用场景**: Bug 修复

### 1.4 并行组合

当任务包含多个独立部分时，可并行使用 Skill：

```
subagent-driven-development
    │
    ├── 子代理 1: tdd (模块 A)
    ├── 子代理 2: tdm (模块 B)
    └── 子代理 3: tdd (模块 C)
```

---

## 2. 工作流编排

### 2.1 工作流设计原则

| 原则 | 说明 |
|-----|------|
| **单一职责** | 每个 Skill 只做一件事 |
| **清晰入口** | 明确工作流的开始条件 |
| **明确出口** | 定义工作流完成的标准 |
| **可观测** | 每个步骤可追踪和验证 |

### 2.2 常见工作流模板

#### 工作流 1: 新功能开发

```yaml
name: new-feature-development
description: 新功能完整开发流程

stages:
  - name: 设计
    skill: brainstorming
    input: 功能需求
    output: 设计方案

  - name: 实现
    skill: tdd
    input: 设计方案
    output: 通过测试的代码

  - name: 调试（如需要）
    skill: systematic-debugging
    input: Bug 信息
    output: 修复后的代码

  - name: 审查
    skill: requesting-code-review
    input: 待审查代码
    output: 审查通过

trigger:
  - "new-feature"
```

#### 工作流 2: Bug 修复

```yaml
name: bug-fix
description: 系统化 Bug 修复流程

stages:
  - name: 复现
    skill: systematic-debugging
    phase: reproduce

  - name: 定位
    skill: systematic-debugging
    phase: localize

  - name: 修复
    skill: systematic-debugging
    phase: fix

  - name: 验证
    skill: systematic-debugging
    phase: verify

  - name: 审查
    skill: receiving-code-review
    input: 修复代码

trigger:
  - "fix-bug"
```

#### 工作流 3: 大型任务

```yaml
name: large-task
description: 大型任务并行处理

stages:
  - name: 分解
    skill: brainstorming
    input: 整体需求
    output: 任务分解方案

  - name: 并行执行
    skill: subagent-driven-development
    input: 分解后的子任务
    output: 子任务结果

  - name: 结果聚合
    skill: receiving-code-review
    input: 所有子任务结果
    output: 整合后的系统

trigger:
  - "large-task"
```

### 2.3 工作流执行检查清单

**开始前**:
- [ ] 明确任务目标和范围
- [ ] 确定适用的工作流模板
- [ ] 准备必要的输入材料

**执行中**:
- [ ] 每个 Skill 完成后验证输出
- [ ] 记录关键决策和变更
- [ ] 及时识别阻塞和风险

**完成后**:
- [ ] 验证最终结果满足目标
- [ ] 更新相关文档
- [ ] 复盘总结改进点

---

## 3. 常见使用场景

### 3.1 场景矩阵

| 场景 | 推荐 Skill | 组合方式 |
|-----|-----------|---------|
| **新项目启动** | brainstorming | 单独使用或 + tdd |
| **新功能开发** | brainstorming + tdd | 链式组合 |
| **Bug 修复** | systematic-debugging | 单独使用 |
| **代码重构** | tdd + receiving-code-review | 链式组合 |
| **大型项目** | subagent + brainstorming | 并行 + 链式 |
| **代码审查** | requesting/receiving-code-review | 根据角色选择 |
| **创建新 Skill** | writing-skills | 单独使用 |
| **技术选型** | brainstorming | 单独使用 |

### 3.2 详细场景指南

#### 场景 1: 新功能开发

**触发命令**:
```bash
brainstorm 设计一个用户认证模块
```

**后续流程**:
```
设计完成 ──▶ tdd 实现用户注册功能
                       │
                       ▼
              遇到 Bug ──▶ systematic-debugging
                       │
                       ▼
                 修复完成 ──▶ requesting-code-review
```

**完整示例**:
```
用户: brainstorm 设计一个 REST API

输出:
## 头脑风暴：REST API 设计

### 需求分析
- 用户管理 API
- 资源管理 API
- 认证授权

### 方案选择
选择方案：Express + RESTful 设计

### 实施计划
1. 设计 API 规范
2. 实现用户管理
3. 实现资源管理
4. 添加认证

---

用户: tdd 实现用户管理 API

输出:
## TDD：用户管理 API

### 红
```javascript
test('创建用户', async () => {
  const user = await UserService.create({
    name: 'Test',
    email: 'test@example.com'
  });
  expect(user.id).toBeDefined();
});
```

### 绿
```javascript
// 实现最小代码
async create(data) {
  return db.insert('users', data);
}
```

### 重构
添加验证逻辑，完善错误处理...
```

#### 场景 2: Bug 修复

**触发命令**:
```bash
systematic-debugging
```

**执行流程**:
```
用户: 用户无法登录系统

### Stage 1: 复现
- 收集错误：Authentication failed
- 复现步骤：使用正确凭据登录
- 验证：问题稳定复现

### Stage 2: 定位
- 检查网络请求：请求正常发出
- 检查服务端日志：收到请求但验证失败
- 检查数据库：用户记录存在
- 检查密码哈希：哈希值不匹配
- 根因：密码更新时算法未正确应用

### Stage 3: 修复
- 重新生成密码哈希
- 更新数据库

### Stage 4: 验证
- 用户可正常登录
- 其他用户未受影响
```

#### 场景 3: 代码审查

**作为代码作者**:
```bash
request-review
```

**准备清单**:
- [ ] 运行所有测试
- [ ] 检查代码风格
- [ ] 准备 PR 描述
- [ ] 自我审查一遍

**作为审查者**:
```bash
receive-review
```

**审查清单**:
- [ ] 功能正确性
- [ ] 代码质量
- [ ] 测试覆盖
- [ ] 安全考虑
- [ ] 性能影响

#### 场景 4: 大型任务并行处理

**触发命令**:
```bash
subagent
```

**任务分解**:
```
原始任务：构建博客系统

分解：
├── 子代理 1: 用户模块
│   └── tdd 实现
├── 子代理 2: 文章模块
│   └── tdd 实现
├── 子代理 3: 评论模块
│   └── tdd 实现
└── 子代理 4: 认证模块
    └── tdd 实现

并行执行 → 结果聚合 → 集成测试
```

---

## 4. 高级技巧

### 4.1 Skill 参数化

使用参数定制 Skill 行为：

```bash
# 指定测试框架
tdd --framework jest

# 指定调试策略
systematic-debugging --strategy binary-search

# 指定审查者
request-review --reviewer @alice
```

### 4.2 条件触发

根据条件自动调用 Skill：

| 条件 | 触发的 Skill | 说明 |
|-----|-------------|------|
| 代码变更超过 500 行 | requesting-code-review | 大变更需审查 |
| 测试覆盖率低于 80% | tdd | 需要更多测试 |
| 遇到编译错误 | systematic-debugging | 启动调试 |
| 新项目创建 | brainstorming | 先规划设计 |

### 4.3 自定义工作流

创建自定义 Skill 链：

```markdown
---
name: my-workflow
description: 我的自定义工作流
type: skill
trigger:
  - "mywf"
workflow:
  - brainstorming
  - tdd
  - receiving-code-review
---

# 我的工作流

## 使用方式
/mywf <任务描述>

## 流程说明
1. brainstorming - 需求设计
2. tdd - 测试驱动实现
3. receiving-code-review - 代码审查
```

---

## 5. 常见问题

### Q1: 何时使用 brainstorming？

**何时使用**:
- 新项目启动
- 大型功能设计
- 技术方案选型
- 需求不明确

**何时跳过**:
- 简单明确的任务
- 有现成模式可循
- 紧急修复

### Q2: TDD 是否适用于所有场景？

**适用**:
- 核心业务逻辑
- 复杂算法
- 数据处理
- API 开发

**不适用**:
- 探索性实验
- 原型快速验证
- 一次性脚本

### Q3: 如何决定是否使用 subagent？

**使用**:
- 任务可分解为独立子任务
- 子任务可并行执行
- 总任务量大

**不使用**:
- 任务强耦合
- 子任务数量少（<3）
- 任务简单（时间节省不明显）

### Q4: 代码审查是必须的吗？

**建议审查**:
- 生产代码变更
- 公共 API
- 核心模块
- 安全相关代码

**可跳过**:
- 文档更新
- 配置变更
- 小型 Bug 修复
- 实验性代码

---

## 6. 总结

### 6.1 核心原则

1. **使用 HARD-GATE** - 每个阶段都要验证后再继续
2. **遵循 TDD 铁律** - 红、绿、重构，循环不止
3. **应用 1% 原则** - 有风险就调用对应 Skill
4. **组合使用** - 根据场景灵活组合 Skill

### 6.2 快速参考

| 任务 | 快捷命令 |
|-----|---------|
| 新功能开发 | `brainstorm` → `tdd` |
| Bug 修复 | `systematic-debugging` |
| 大型任务 | `subagent` |
| 代码审查 | `request-review` / `receive-review` |
| 创建 Skill | `write-skills` |

---

## 相关章节

- [[17-01-🚀-Superpowers入门]] - Superpowers 概述
- [[17-02-🧠-Superpowers核心Skill]] - 核心 Skill 详解
- [[17-03-🎯-Superpowers方法论]] - 核心方法论

---

*最后更新：2026-04-03*
