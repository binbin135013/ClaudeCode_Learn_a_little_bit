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

- [[🚀-快速上手指南]] - 新手入门
- [[📐-架构概览]] - 整体架构
- [[🔧-Tool系统]] - 工具系统
- [[🔐-权限系统]] - 权限管理
- [[🧠-记忆系统]] - 记忆系统
- [[📦-上下文管理]] - 上下文压缩
- [[⚙️-QueryEngine]] - 核心引擎
- [[♻️-核心设计模式]] - 设计模式

---

## ⚙️ Dataview 设置说明

如果你看不到上面的图表：

1. 安装 Dataview 插件
2. 启用 JavaScript 视图
3. 刷新本页面

---

> 💡 提示: 按 `Ctrl/Cmd + E` 切换编辑/预览模式
> 按 `Ctrl/Cmd + P` 打开命令面板，输入 "Dataview" 查看更多查询