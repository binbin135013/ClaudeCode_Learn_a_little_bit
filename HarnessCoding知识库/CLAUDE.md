# Claude Code 源码学习知识库

这是一个用于学习 Claude Code 源码的 Obsidian 知识库。

## 知识库结构（18个模块）

```
HarnessCoding知识库/
├── 00-入口/                    # 入口和导航（6个文档）
├── 01-架构总览/                # 整体架构
├── 02-Tool系统/                # 工具系统
├── 03-权限系统/                # 权限管理
├── 04-Hook系统/                # 生命周期扩展
├── 05-记忆系统/                # 记忆系统
├── 06-上下文管理/              # 上下文压缩
├── 07-OpenClaw实战/            # OpenClaw实战（5个文档）
├── 08-Skill系统/               # 技能系统
├── 09-子Agent与协作/           # 多Agent协作
├── 10-设计模式/                # 设计模式
├── 11-QueryEngine/            # 核心引擎
├── 12-质量保证/                # 质量保证（3个文档）
├── 13-OMC完全指南/             # OMC框架指南（4个文档）
├── 14-方法论/                  # 方法论（4个文档）
├── 15-ClaudeCode最佳实践/      # Claude Code最佳实践（5个文档）
├── 17-Superpowers/            # Superpowers技能（4个文档）
└── 18-MCP系统/                # MCP协议系统
```

## 学习路径

推荐按顺序学习：

1. **🚀-快速上手指南** - 先读这个
2. **📐-架构概览** - 理解全局
3. **🔧-Tool系统** - 核心中的核心
4. **🔐-权限系统** - 安全设计
5. **🧠-记忆系统** - 知识库建设
6. **📦-上下文管理** - 上下文优化
7. **🪝-Hook系统** - 进阶扩展
8. **📚-Skill系统** - 能力封装
9. **🤝-子Agent与协作** - 多Agent
10. **⚙️-QueryEngine** - 核心引擎
11. **♻️-核心设计模式** - 设计总结
12. **🧪-MCP系统** - 外部扩展协议

## Obsidian 插件需求

推荐安装以下插件：

- **Dataview** - 知识库索引和统计
- **Reminder** - 学习打卡提醒
- **Breadcrumbs** - 导航增强
- **Link local** - 更好地处理本地链接

## 相关资源

- Claude Code 官方文档
- [lintsinghua/claude-code-book](https://github.com/lintsinghua/claude-code-book)（1,190 stars）
- [liuup/claude-code-analysis](https://github.com/liuup/claude-code-analysis)（537 stars）
- [claude-code-best/claude-code](https://github.com/claude-code-best/claude-code)（8,712 stars）

## 知识库维护注意事项

- 模块编号在1-9后使用两位数（10, 11, 12...）避免冲突
- wiki跨链接需要使用 `[[../模块名/文件名]]` 格式
- 知识来源标注格式：`> [!cite]- 知识来源`
