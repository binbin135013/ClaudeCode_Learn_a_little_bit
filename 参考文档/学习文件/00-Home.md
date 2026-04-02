---
tags: [仪表盘, MOC]
aliases: [首页, Dashboard, Home]
date: 2026-04-02
---

# 🏠 Claude Code & OpenClaw 知识库

> [!info] 关于本库
> 基于《Claude Code & OpenClaw 中文教程》(v3.1, 130K字, 22篇教程) 构建。
> 面向已有 3-4 个 Claude Code 中型项目经验、部署过 2 个 OpenClaw 项目的开发者。
> **不是入门教程的搬运，而是结构化的进阶知识图谱。**

> [!tip] 使用建议
> 知识库中标注了 `[!caution]` 的内容为编辑推断，请以官方文档为准。建议配合 [Claude Code 官方文档](https://docs.anthropic.com/en/docs/claude-code) 和 [OpenClaw GitHub](https://github.com/openclaw/openclaw) 使用。

---

## 💚 知识库健康度

```dataview
TABLE length(rows.file.link) AS 笔记数量
FROM "学习文件"
FLATTEN file.tags AS tag
GROUP BY tag
SORT 笔记数量 DESC
```

| 健康指标 | 数值 |
|---------|------|
| 总笔记数 | `= length(file.lists)` |

**孤立笔记**（无入链，可能需要补充双链）：

```dataview
LIST
FROM "学习文件"
WHERE length(file.inlinks) = 0 AND file.name != "00-Home" AND file.name != "00-MOC-学习地图"
SORT file.name ASC
```

**最近活跃**：

```dataview
TABLE file.mtime AS 更新时间, file.tags AS 标签
FROM "学习文件"
SORT file.mtime DESC
LIMIT 10
```

## 📚 核心概念

```dataview
TABLE aliases AS 别名, file.tags AS 标签
FROM "学习文件/01-核心概念"
SORT file.name ASC
```

## 🚀 Claude Code 进阶

```dataview
TABLE aliases AS 别名, file.tags AS 标签
FROM "学习文件/02-Claude Code进阶"
SORT file.name ASC
```

## 🦞 OpenClaw 实战

```dataview
TABLE aliases AS 别名, file.tags AS 标签
FROM "学习文件/03-OpenClaw实战"
SORT file.name ASC
```

## 🏆 最佳实践

```dataview
TABLE aliases AS 别名, file.tags AS 标签
FROM "学习文件/04-最佳实践"
SORT file.name ASC
```

## ⚡ 速查表

```dataview
TABLE aliases AS 别名
FROM "学习文件/05-速查"
SORT file.name ASC
```

---

## 🔗 学习路径推荐

> [!tip] 你的经验水平：中级开发者
> 已完成 3-4 个 CC 项目 + 2 个 OC 部署，建议重点学习以下方向：

| 方向 | 重点笔记 | 优先级 |
|------|---------|--------|
| 深度定制 | [[Skills开发]] · [[Commands自定义命令]] · [[Agent-SDK]] · [[Plugins生态]] | 高 |
| 生产部署 | [[Docker部署]] · [[安全最佳实践]] · [[CI-CD与自动化]] | 高 |
| 避坑指南 | [[常见陷阱与反模式]] · [[环境搭建指南]] | 高 |
| 性能优化 | [[性能与成本优化]] · [[权限与沙箱]] | 中 |
| 架构理解 | [[架构设计]] · [[MCP协议]] · [[Agent与多代理]] · [[消息平台集成]] | 中 |
| 协同工作流 | [[CC-OC协同工作流]] · [[MCP协议]] · [[记忆系统]] | 中 |

---

## 🏷️ 按标签浏览

**核心概念**：
```dataview
TABLE file.mtime AS 更新时间
FROM "学习文件"
WHERE contains(file.tags, "核心概念")
SORT file.name ASC
```

**MOC / 导航**：
```dataview
TABLE file.mtime AS 更新时间
FROM "学习文件"
WHERE contains(file.tags, "MOC")
SORT file.name ASC
```

**仪表盘**：
```dataview
TABLE file.mtime AS 更新时间
FROM "学习文件"
WHERE contains(file.tags, "仪表盘")
SORT file.name ASC
```

---

## 📝 学习日记

```dataview
TABLE file.mtime AS 时间
FROM "学习文件/06-日记"
WHERE file.name != "模板"
SORT file.mtime DESC
LIMIT 7
```

> [!example] 记录学习
> 在 `06-日记/` 目录下创建以日期命名的笔记，如 `2026-04-02.md`
