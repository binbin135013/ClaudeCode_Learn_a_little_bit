---
type: system
tags: [openclaw, docker, deployment, production]
description: OpenClaw Docker容器化部署指南
---

# 🐳 OpenClaw Docker部署

> ⏱ 难度: ★★☆ | 重要性: ★★★ | 推荐学习时间: 2-3天
> 💡 **特别推荐**: 想 7x24 小时运行 OpenClaw？这是必学内容！

---

## 概述

### 为什么需要 Docker 部署？

| 场景 | 裸机部署 | Docker 部署 |
|------|---------|------------|
| 个人使用 | ✅ 足够 | ⚠️ 可选 |
| 团队使用 | ⚠️ 复杂 | ✅ 推荐 |
| 7x24 运行 | ⚠️ 需配置 | ✅ 内置 |
| 多实例 | ❌ 困难 | ✅ 简单 |
| 远程访问 | ⚠️ 需配置 | ✅ 简单 |

---

## 部署架构

```
VPS / 云服务器
├── Docker Compose
│   ├── openclaw-gateway    ← 核心服务（含 Web 控制面板）
│   └── nginx               ← 反向代理 + TLS
└── 数据持久化
    ├── ~/.openclaw/        ← 配置和工作空间
    └── sqlite-data         ← 记忆数据库
```

---

## 基础 Docker 部署

```bash
# 拉取镜像
docker pull openclaw/openclaw:latest

# 运行
docker run -d \
  --name openclaw \
  -p 18789:18789 \
  -v ~/.openclaw:/home/node/.openclaw \
  -e OPENCLAW_GATEWAY_TOKEN=your-token \
  openclaw/openclaw:latest
```

---

## Docker Compose（推荐）

> [!note] 命令语法
> 使用 `docker compose`（v2，无连字符）。v1 的 `docker-compose` 已弃用，请确认你的 Docker Desktop 版本支持 Compose v2。

```yaml
version: '3.8'
services:
  gateway:
    image: openclaw/openclaw:latest
    ports:
      - "18789:18789"
    volumes:
      - ./openclaw-data:/home/node/.openclaw
    environment:
      - OPENCLAW_GATEWAY_TOKEN=${OPENCLAW_GATEWAY_TOKEN}
    restart: always

  nginx:
    image: nginx:alpine
    ports:
      - "443:443"
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./certs:/etc/nginx/certs
    restart: always
```

---

## 云平台部署

| 平台 | 难度 | 费用 | 说明 |
|------|------|------|------|
| Fly.io | ⭐ | 有免费层 | 推荐入门 |
| Render | ⭐ | 有免费层 | 简单易用 |
| VPS (搬瓦工等) | ⭐⭐ | $5+/月 | 完全控制 |
| Railway | ⭐ | 按量计费 | 开发者友好 |

> [!warning] 安全必做
> 公网部署时**必须**：
> 1. 启用 TLS 1.3
> 2. 配置 Gateway Token 认证
> 3. 设置 allowFrom 允许列表
> 4. 配置 DM 策略

---

## 监控和维护

```bash
# 查看日志
docker logs -f openclaw

# 健康检查
openclaw doctor

# 更新
docker pull openclaw/openclaw:latest
docker compose up -d
```

---

## 相关章节

- [[../01-架构总览/📐-架构概览]] - OpenClaw 架构
- [[📐-架构概览]] - OpenClaw 架构
- [[🔐-安全最佳实践]] - 安全加固指南
- [[🏠-知识库首页]] - 返回首页
