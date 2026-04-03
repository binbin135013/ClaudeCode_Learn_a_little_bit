---
type: dashboard
tags: [dataview, dashboard, obsidian]
description: Claude Code 知识库 Dataview 仪表盘
---

# 📊 Claude Code 知识库仪表盘

> 基于 Dataview 插件的 Obsidian 知识库可视化 | 最后更新: {{date}}

---

## 📈 知识库概览

```dataview
TABLE
  rows.file.link as 文件,
  rows.type as 类型,
  rows.description as 描述
FROM ""
WHERE type != "index"
FLATTEN file.tags as tag
GROUP BY type
```

---

## 📚 按系统分类

### 核心系统 (⭐ 必修)

```dataview
TABLE type, description
FROM ""
WHERE contains(tags, "claude-code") AND (contains(tags, "tool") OR contains(tags, "permission") OR contains(tags, "query-engine"))
SORT type ASC
```

### 进阶系统

```dataview
TABLE type, description
FROM ""
WHERE contains(tags, "hook") OR contains(tags, "skill") OR contains(tags, "memory")
SORT type ASC
```

---

## 🎯 学习进度

```dataview
TABLE
  file.ctime as 开始日期,
  choice(completed = true, "✅ 已完成", "📖 进行中") as 状态,
  priority as 优先级
FROM "00-入口"
WHERE type = "guide" OR type = "index"
SORT file.ctime DESC
```

---

## 🔗 双向链接统计

### 最常被引用的章节

```dataview
TABLE
  length(file.outlinks) as 发出链接数,
  length(file.inlinks) as 被引用数
FROM ""
WHERE file.outlinks > 0
SORT length(file.inlinks) DESC
LIMIT 10
```

### 孤立页面（无引用）

```dataview
TABLE file.link as 孤立文件
FROM ""
WHERE length(file.inlinks) = 0 AND file.name != "🏠-知识库首页"
```

---

## 📅 最近更新

```dataview
TABLE file.mtime as 修改时间, type
FROM ""
SORT file.mtime DESC
LIMIT 5
```

---

## 🎓 学习路线图

| 阶段 | 内容 | 状态 |
|-----|------|------|
| 阶段一 | 架构概览 + Tool系统 | 📖 |
| 阶段二 | 权限系统 + Hook系统 | 📖 |
| 阶段三 | 记忆系统 + 上下文管理 | 📖 |
| 阶段四 | Skill系统 + 设计模式 | 📖 |
| 阶段五 | 子Agent协作 + 实战 | 📖 |

---

## 🏷️ 标签云

```dataview
TABLE
  file.link as 文件,
  tag as 标签
FROM ""
FLATTEN file.tags as tag
WHERE tag != "claude-code"
GROUP BY tag
```

---

## 快速链接

- [[00-02-🚀-快速上手指南]] - 新手入门
- [[01-架构总览/01-01-📐-架构概览]] - 整体架构
- [[02-Tool系统/02-01-🔧-Tool系统]] - 工具系统
- [[03-权限系统/03-01-🔐-权限系统]] - 权限管理
- [[04-Hook系统/04-01-🪝-Hook系统]] - Hook系统
- [[05-记忆系统/05-01-🧠-记忆系统]] - 记忆系统
- [[06-上下文管理/06-01-📦-上下文管理]] - 上下文压缩
- [[07-OpenClaw实战/07-01-🚀-环境搭建指南]] - OpenClaw实战
- [[08-Skill系统/08-01-📚-Skill系统]] - Skill系统
- [[09-子Agent与协作/09-01-🤝-子Agent与协作]] - 子Agent协作
- [[10-设计模式/10-01-♻️-核心设计模式]] - 设计模式
- [[11-QueryEngine/11-01-⚙️-QueryEngine]] - 核心引擎
- [[12-质量保证/12-01-🧪-ultraqa循环]] - 质量保证
- [[13-OMC完全指南/13-01-🏁-OMC入门]] - OMC框架
- [[14-方法论/14-01-🎯-Skill选择决策树]] - 方法论
- [[15-ClaudeCode最佳实践/15-01-⚡-ClaudeCode插件推荐]] - 最佳实践
- [[17-Superpowers/17-01-🚀-Superpowers入门]] - Superpowers
- [[18-MCP系统/18-01-📚-MCP系统]] - MCP系统

---

## ⚙️ Dataview 设置说明

如果你看不到上面的图表：

1. 安装 Dataview 插件
2. 启用 JavaScript 视图
3. 刷新本页面

---

> 💡 提示: 按 `Ctrl/Cmd + E` 切换编辑/预览模式
> 按 `Ctrl/Cmd + P` 打开命令面板，输入 "Dataview" 查看更多查询