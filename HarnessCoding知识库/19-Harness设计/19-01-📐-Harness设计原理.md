---
type: architecture
tags: [harness, multi-agent, context-management, claude-code]
description: Claude Code团队Harness设计原理与多Agent架构
---

# Harness 设计原理

## 核心概念
- **Harness**: 支撑AI编程工具的骨架系统
- **Purpose**: 让AI能够构建完整的长时间运行的应用

## GAN式多Agent架构
Inspired by Generative Adversarial Networks - generator-evaluator pattern:
- **Generator**: 产出输出（代码、设计）
- **Evaluator**: 批判性评估（与Generator分离，避免自我服务偏差）

## 三Agent架构
```
Planner → Generator → Evaluator
```
1. **Planner**: 将1-4句prompt扩展为完整规格说明
2. **Generator**: 按sprint契约构建功能
3. **Evaluator**: 使用Playwright进行QA验证

## 上下文管理策略

| 方案 | 描述 | 适用场景 |
|------|------|----------|
| **Compaction** | 就地总结对话历史 | Sonnet 4.5及以下 |
| **Context Reset** | 完全清空上下文 | Opus强model时代 |

**关键发现**: Claude Sonnet 4.5存在严重"上下文焦虑"——接近上下文限制时会提前结束

## 评估器设计原则
- Agents自我评估总是偏正面
- 独立评估器比生成器自我批判更可控
- 设计质量 > 原创性 > 工艺 > 功能性（权重排序）

## 成本/质量权衡
| Harness类型 | 时长 | 成本 |
|-------------|------|------|
| Solo agent | 20min | $9 |
| Full harness (v1) | 6h | $200 |
| Simplified harness (v2) | ~4h | $125 |

## 核心启示
1. Harness设计显著影响长时间运行能力
2. 关注分离：Generator/Evaluator分离解决自我评估偏差
3. Context management随model能力进化
4. Evaluator价值取决于任务是否超过model独立能力边界
5. 通过反馈循环的迭代改进驱动质量

## 相关文档
- 01-架构总览/01-01-📐-架构概览.md
- 11-QueryEngine/11-01-⚙️-QueryEngine.md
- 14-方法论/14-02-🔄-Ralph循环最佳实践.md
