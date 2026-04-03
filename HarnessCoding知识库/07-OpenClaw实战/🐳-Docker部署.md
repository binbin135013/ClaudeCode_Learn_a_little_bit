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
│   ├── nginx               ← 反向代理 + TLS
│   ├── postgres            ← 记忆数据库（可选）
│   └── redis               ← 缓存服务（可选）
└── 数据持久化
    ├── openclaw-data       ← 配置和数据卷
    ├── openclaw-workspace  ← 工作空间
    ├── postgres-data       ← PostgreSQL 数据
    └── redis-data          ← Redis 数据
```

---

## 完整 Docker Compose 配置

> 以下是生产环境可用的完整配置，包含资源限制、网络隔离、健康检查等。

```yaml
# docker-compose.yml
# 生产环境完整配置 - 包含 Gateway + Nginx 反向代理
version: "3.8"

services:
  # ========== OpenClaw Gateway 核心服务 ==========
  openclaw-gateway:
    image: openclaw/openclaw:latest
    container_name: openclaw-gateway
    hostname: openclaw-gateway

    # 端口映射 - 仅本地绑定，由 Nginx 代理对外暴露
    ports:
      - "127.0.0.1:18789:18789"   # Gateway 主服务（本地绑定更安全）

    # 数据卷挂载 - 持久化配置和工作空间
    volumes:
      # 配置和数据持久化（必需）
      - openclaw-data:/home/node/.openclaw
      # 工作空间持久化（必需）
      - openclaw-workspace:/home/node/.openclaw/workspace
      # 本地模型挂载（可选 - 如果使用 Ollama）
      - /var/run/docker.sock:/var/run/docker.sock

    # 环境变量配置
    environment:
      # 核心配置
      - NODE_ENV=production
      - OPENCLAW_GATEWAY_TOKEN=${OPENCLAW_GATEWAY_TOKEN}
      - LOG_LEVEL=info

      # 数据库配置（可选 - 如不使用可删除）
      - DATABASE_URL=postgresql://${POSTGRES_USER:-openclaw}:${POSTGRES_PASSWORD}@postgres:5432/${POSTGRES_DB:-openclaw}

      # Redis 配置（可选 - 如不使用可删除）
      - REDIS_URL=redis://redis:6379/0

      # 本地模型配置（可选）
      - OLLAMA_BASE_URL=http://host.docker.internal:11434

      # API Keys（建议使用 .env 文件管理）
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}

    # 健康检查配置 - 确保服务可用性
    healthcheck:
      # 检查 Gateway 健康端点
      test: ["CMD", "node", "-e", "fetch('http://localhost:18789/health').then(r => { if (!r.ok) process.exit(1) }).catch(() => process.exit(1))"]
      interval: 30s      # 每 30 秒检查一次
      timeout: 10s      # 超时时间
      retries: 3        # 连续失败 3 次标记为 unhealthy
      start_period: 60s # 启动后等待 60 秒再开始检查

    # 重启策略
    restart: unless-stopped

    # 资源限制（防止容器占用过多资源）
    deploy:
      resources:
        limits:
          cpus: "2.0"           # 最多使用 2 个 CPU 核心
          memory: 4G            # 最多使用 4GB 内存
        reservations:
          cpus: "0.5"           # 保留最少 0.5 个 CPU
          memory: 1G             # 保留最少 1GB 内存

    # 网络配置
    networks:
      - openclaw-net

    # 依赖服务（等待依赖服务就绪后再启动）
    depends_on:
      redis:
        condition: service_healthy
      postgres:
        condition: service_healthy

    # 安全配置 - 非主会话在 Docker 沙箱中运行
    cap_drop:
      - ALL                    # 移除所有 Linux capabilities

    # 日志配置 - 限制日志大小防止磁盘占满
    logging:
      driver: "json-file"
      options:
        max-size: "50m"        # 单个日志文件最大 50MB
        max-file: "5"          # 最多保留 5 个日志文件
        compress: "true"       # 压缩旧日志

  # ========== PostgreSQL 数据库（可选）==========
  postgres:
    image: postgres:16-alpine
    container_name: openclaw-postgres
    hostname: openclaw-postgres

    environment:
      - POSTGRES_USER=${POSTGRES_USER:-openclaw}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_DB=${POSTGRES_DB:-openclaw}
      - PGDATA=/var/lib/postgresql/data/pgdata

    volumes:
      - postgres-data:/var/lib/postgresql/data

    # 健康检查
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER:-openclaw} -d ${POSTGRES_DB:-openclaw}"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s

    restart: unless-stopped

    deploy:
      resources:
        limits:
          cpus: "1.0"
          memory: 2G
        reservations:
          cpus: "0.25"
          memory: 512M

    networks:
      - openclaw-net

    logging:
      driver: "json-file"
      options:
        max-size: "20m"
        max-file: "3"

  # ========== Redis 缓存（可选）==========
  redis:
    image: redis:7-alpine
    container_name: openclaw-redis
    hostname: openclaw-redis

    command: >
      redis-server
      --appendonly yes
      --maxmemory 512mb
      --maxmemory-policy allkeys-lru
      --save 900 1
      --save 300 10
      --save 60 10000

    volumes:
      - redis-data:/data

    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

    restart: unless-stopped

    deploy:
      resources:
        limits:
          cpus: "0.5"
          memory: 1G
        reservations:
          cpus: "0.1"
          memory: 256M

    networks:
      - openclaw-net

    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

  # ========== Nginx 反向代理 ==========
  nginx:
    image: nginx:alpine
    container_name: openclaw-nginx
    hostname: openclaw-nginx

    ports:
      - "80:80"                 # HTTP（重定向到 HTTPS）
      - "443:443"               # HTTPS

    volumes:
      # Nginx 配置文件
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      # TLS 证书
      - ./nginx/certs:/etc/nginx/certs:ro
      # 证书 DH 参数（用于 Perfect Forward Secrecy）
      - ./nginx/dhparam:/etc/nginx/dhparam:ro
      # 自定义日志格式
      - ./nginx/conf.d:/etc/nginx/conf.d:ro

    # 健康检查
    healthcheck:
      test: ["CMD", "wget", "-q", "--spider", "http://localhost/health", "||", "exit", "1"]
      interval: 30s
      timeout: 10s
      retries: 3

    restart: unless-stopped

    deploy:
      resources:
        limits:
          cpus: "0.5"
          memory: 256M

    networks:
      - openclaw-net

    depends_on:
      - openclaw-gateway

    logging:
      driver: "json-file"
      options:
        max-size: "20m"
        max-file: "5"

# ========== 数据卷定义 ==========
volumes:
  # OpenClaw 配置和数据卷
  openclaw-data:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: ./volumes/openclaw-data

  # OpenClaw 工作空间卷
  openclaw-workspace:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: ./volumes/openclaw-workspace

  # PostgreSQL 数据卷
  postgres-data:
    driver: local

  # Redis 数据卷
  redis-data:
    driver: local

# ========== 网络定义 ==========
networks:
  # 内部桥接网络 - 容器间通信
  openclaw-net:
    driver: bridge
    driver_opts:
      # 启用网络隔离 - 容器不能直接 IP 欺骗
      com.docker.network.bridge.enable_icc: "true"
      com.docker.network.bridge.enable_ip_masquerade: "true"
    ipam:
      driver: default
      config:
        - subnet: 172.28.0.0/16   # 私有子网
```

---

## Nginx 完整配置

### 目录结构

```bash
# 创建 Nginx 配置目录
mkdir -p nginx/certs nginx/dhparam nginx/conf.d
```

### nginx.conf 主配置文件

```nginx
# /etc/nginx/nginx.conf
# Nginx 主配置文件 - TLS 1.3 + WebSocket + 安全头

# 运行用户
user nginx;

# 工作进程数（auto = CPU 核心数）
worker_processes auto;

# 错误日志
error_log /var/log/nginx/error.log warn;

# PID 文件
pid /var/run/nginx.pid;

# 加载模块
load_module modules/ngx_http_modsecurity_module.so;

events {
    # 每个工作进程的最大连接数
    worker_connections 2048;

    # 使用 epoll（Linux）提高性能
    use epoll;

    # 允许批量接受连接
    multi_accept on;
}

http {
    # ========== 基础配置 ==========
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # 字符集
    charset utf-8;

    # 日志格式
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for" '
                    'rt=$request_time uct="$upstream_connect_time" '
                    'uht="$upstream_header_time" urt="$upstream_response_time"';

    # 访问日志
    access_log /var/log/nginx/access.log main buffer=16k flush=2s;

    # ========== 性能优化 ==========
    # 隐藏 Nginx 版本号
    server_tokens off;

    # 开启高效文件传输
    sendfile on;

    # 减少网络报文段的数量
    tcp_nopush on;
    tcp_nodelay on;

    # 保持连接超时时间（秒）
    keepalive_timeout 65;

    # 请求体大小限制（最大上传文件）
    client_max_body_size 50M;

    # 请求头大小限制
    large_client_header_buffers 4 32k;

    # 开启 gzip 压缩
    gzip on;
    gzip_disable "msie6";
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_min_length 1024;
    gzip_types text/plain text/css text/xml application/json application/javascript
               application/xml application/xml+rss text/javascript application/x-javascript
               image/svg+xml;

    # ========== 安全头 ==========
    # 强制使用 HTTPS（HSTS）
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;

    # 防止点击劫持
    add_header X-Frame-Options "SAMEORIGIN" always;

    # 防止 MIME 类型嗅探
    add_header X-Content-Type-Options "nosniff" always;

    # XSS 防护（现代浏览器支持）
    add_header X-XSS-Protection "1; mode=block" always;

    # 引用策略
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;

    # 权限策略（限制功能接口）
    add_header Permissions-Policy "camera=(), microphone=(), geolocation=()" always;

    # 内容安全策略（CSP）
    add_header Content-Security-Policy "
        default-src 'self';
        script-src 'self' 'unsafe-inline' 'unsafe-eval';
        style-src 'self' 'unsafe-inline';
        img-src 'self' data: https:;
        font-src 'self' data:;
        connect-src 'self' wss: https:;
        frame-ancestors 'self';
        form-action 'self';
        base-uri 'self';
    " always;

    # ========== TLS/SSL 配置 ==========
    ssl_protocols TLSv1.2 TLSv1.3;

    # 优先使用 TLS 1.3
    ssl_prefer_server_ciphers off;

    # SSL 曲线
    ssl_ciphers 'TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256:TLS_AES_128_GCM_SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384';

    # DH 参数（用于 Perfect Forward Secrecy）
    ssl_dhparam /etc/nginx/dhparam/dhparam.pem;

    # SSL 缓存
    ssl_session_cache shared:SSL:50m;
    ssl_session_timeout 1d;
    ssl_session_tickets off;

    # OCSP Stapling（加快 TLS 握手）
    ssl_stapling on;
    ssl_stapling_verify on;
    resolver 8.8.8.8 8.8.4.4 valid=300s;
    resolver_timeout 5s;

    # ========== OpenClaw 代理配置 ==========
    include /etc/nginx/conf.d/*.conf;
}
```

### conf.d/openclaw.conf 代理配置

```nginx
# /etc/nginx/conf.d/openclaw.conf
# OpenClaw Gateway 反向代理配置

# ========== HTTP 重定向到 HTTPS ==========
server {
    listen 80;
    server_name _;

    # Let's Encrypt ACME 挑战（用于自动续期证书）
    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
        try_files $uri $uri/ =404;
    }

    # 其他请求重定向到 HTTPS
    location / {
        return 301 https://$host$request_uri;
    }
}

# ========== HTTPS 服务器 ==========
server {
    listen 443 ssl http2;
    server_name _;

    # SSL 证书配置（替换为你的证书路径）
    ssl_certificate /etc/nginx/certs/fullchain.pem;
    ssl_certificate_key /etc/nginx/certs/privkey.pem;

    # 根目录
    root /var/www/html;
    index index.html;

    # ========== Nginx 健康检查端点 ==========
    location /health {
        access_log off;
        return 200 "OK";
        add_header Content-Type text/plain;
    }

    # ========== OpenClaw Gateway 代理 ==========
    location / {
        # 代理到 OpenClaw Gateway
        proxy_pass http://openclaw-gateway:18789;
        proxy_http_version 1.1;

        # 基础代理头
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Port $server_port;

        # 代理连接头（用于 WebSocket）
        proxy_set_header Connection "";

        # 禁用缓存
        proxy_cache off;

        # 超时配置
        proxy_connect_timeout 60s;
        proxy_send_timeout 300s;
        proxy_read_timeout 300s;

        # 缓冲配置
        proxy_buffering off;
        proxy_request_buffering off;
    }

    # ========== WebSocket 代理 ==========
    location /ws {
        # WebSocket 需要 Upgrade 头
        proxy_pass http://openclaw-gateway:18789;
        proxy_http_version 1.1;

        # WebSocket 升级
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        # 基础代理头
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # WebSocket 超时（较长）
        proxy_connect_timeout 7d;
        proxy_send_timeout 7d;
        proxy_read_timeout 7d;

        # 禁用缓冲
        proxy_buffering off;
    }

    # ========== API 代理（如果有） ==========
    location /api/ {
        proxy_pass http://openclaw-gateway:18789;
        proxy_http_version 1.1;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # API 超时
        proxy_connect_timeout 30s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;

        # 启用缓冲（适合 JSON 响应）
        proxy_buffering on;
        proxy_buffer_size 4k;
        proxy_buffers 8 4k;
    }

    # ========== 静态资源（如果有） ==========
    location /static/ {
        proxy_pass http://openclaw-gateway:18789;
        proxy_http_version 1.1;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        # 缓存静态资源
        proxy_cache_valid 200 1d;
        expires 1d;
        add_header Cache-Control "public, immutable";
    }

    # ========== 速率限制区域定义 ==========
    # 在 http 块中定义：
    # limit_req_zone $binary_remote_addr zone=openclaw_api:10m rate=10r/s;
    location /api/ {
        limit_req zone=openclaw_api burst=20 nodelay;
        proxy_pass http://openclaw-gateway:18789;
    }

    # ========== 错误页面 ==========
    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
        root /usr/share/nginx/html;
        internal;
    }
}
```

### 安全配置 conf.d/security.conf

```nginx
# /etc/nginx/conf.d/security.conf
# 安全相关配置

# 防止常见攻击
location ~ /\.(?!well-known) {
    deny all;
    access_log off;
    log_not_found off;
}

# 禁止访问隐藏文件
location ~ /\. {
    deny all;
    access_log off;
    log_not_found off;
}

# 禁止访问备份文件
location ~* \.(bak|config|sql|fla|psd|ini|log|sh|inc|swp|dist|old|release)$ {
    deny all;
    access_log off;
    log_not_found off;
}

# 限制 HTTP 方法
location / {
    # 只允许 GET, POST, OPTIONS
    if ($request_method !~ ^(GET|POST|OPTIONS)$ ) {
        return 405;
    }
}
```

---

## 环境变量配置模板

### .env 文件

```bash
# ===========================================
# OpenClaw 生产环境配置
# ===========================================
# 注意：请将此文件加入 .gitignore，切勿提交到版本控制
# 创建命令：cp .env.example .env
# ===========================================

# ========== 核心配置 ==========

# Gateway 访问令牌（必填 - 使用 openssl rand -hex 32 生成）
OPENCLAW_GATEWAY_TOKEN=your-32-char-minimum-secure-token-here

# 运行环境
NODE_ENV=production

# 日志级别：silent, fatal, error, warn, info, debug, trace
LOG_LEVEL=info

# ========== 数据库配置 ==========

# PostgreSQL 密码（必填 - 使用强密码）
POSTGRES_PASSWORD=your-strong-db-password-minimum-16-chars

# PostgreSQL 用户名
POSTGRES_USER=openclaw

# PostgreSQL 数据库名
POSTGRES_DB=openclaw

# ========== API Keys ==========

# OpenAI API Key（用于 AI 对话）
OPENAI_API_KEY=sk-proj-xxxxxxxxxxxxxxxxxxxx

# Anthropic API Key
ANTHROPIC_API_KEY=sk-ant-xxxxxxxxxxxxxxxxxxxx

# Google Gemini API Key（可选）
# GEMINI_API_KEY=AIzaSyxxxxxxxxxxxxxxxxxxxx

# ========== 消息平台集成 ==========

# Telegram Bot Token（可选）
# TELEGRAM_BOT_TOKEN=123456789:ABCdefGHIjklMNOpqrsTUVwxyz

# Discord Bot Token（可选）
# DISCORD_BOT_TOKEN=MTIzNDU2Nzg5OTg3NjXyZABCDEFGHIJKLMNOPQRSTUV

# Slack Bot Token（可选）
# SLACK_BOT_TOKEN=xoxb-1234567890123-1234567890123-XXXXXXXXXX
# SLACK_APP_TOKEN=xapp-1234567890123-1234567890123-XXXXXXXXXX

# ========== 本地模型配置（可选）==========

# Ollama 服务地址（如果使用本地模型）
OLLAMA_BASE_URL=http://host.docker.internal:11434

# ========== 备份配置 ==========

# 备份存储路径
BACKUP_DIR=/opt/backups/openclaw

# 备份保留天数
BACKUP_RETENTION_DAYS=30

# ========== 域名配置 ==========

# 你的域名
DOMAIN=openclaw.yourdomain.com

# ========== Docker 配置 ==========

# Docker 镜像加速器（国内服务器推荐使用）
# REGISTRY_MIRROR=https://your-id.mirror.aliyuncs.com
```

---

## 数据备份方案

### 备份脚本

```bash
#!/bin/bash
# backup.sh - OpenClaw 全量备份脚本
# 使用方法：./backup.sh
# 建议配合 cron 使用：0 3 * * * /opt/openclaw/backup.sh

# ========== 配置 ==========
BACKUP_ROOT_DIR="/opt/backups/openclaw"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="${BACKUP_ROOT_DIR}/${DATE}"
RETENTION_DAYS=${BACKUP_RETENTION_DAYS:-30}

# 颜色输出
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

# 日志函数
log_info() {
    echo -e "${GREEN}[INFO]${NC} $1"
}

log_warn() {
    echo -e "${YELLOW}[WARN]${NC} $1"
}

log_error() {
    echo -e "${RED}[ERROR]${NC} $1"
}

# ========== 备份前检查 ==========
# 检查是否以 root 运行
if [[ $EUID -ne 0 ]]; then
   log_warn "建议以 root 运行以避免权限问题"
fi

# 创建备份目录
mkdir -p "${BACKUP_DIR}"

# 进入 docker-compose 目录
cd "$(dirname "$0")"

log_info "开始备份到: ${BACKUP_DIR}"

# ========== 备份 docker-compose 配置 ==========
log_info "备份 docker-compose 配置..."
cp docker-compose.yml "${BACKUP_DIR}/"
cp .env "${BACKUP_DIR}/.env" 2>/dev/null || log_warn ".env 文件不存在或无法读取"
cp nginx/nginx.conf "${BACKUP_DIR}/nginx.conf" 2>/dev/null || log_warn "nginx.conf 不存在"

# ========== 备份 OpenClaw 数据卷 ==========
log_info "备份 OpenClaw 数据卷..."
docker run --rm \
    -v openclaw-data:/source:ro \
    -v "${BACKUP_DIR}":/backup \
    alpine:latest \
    tar czf /backup/openclaw-data.tar.gz -C /source .

docker run --rm \
    -v openclaw-workspace:/source:ro \
    -v "${BACKUP_DIR}":/backup \
    alpine:latest \
    tar czf /backup/openclaw-workspace.tar.gz -C /source .

# ========== 备份 PostgreSQL ==========
log_info "备份 PostgreSQL 数据库..."
if docker compose ps postgres | grep -q "Up"; then
    docker compose exec -T postgres \
        pg_dump -U ${POSTGRES_USER:-openclaw} -d ${POSTGRES_DB:-openclaw} \
        | gzip > "${BACKUP_DIR}/postgres-backup.sql.gz"
    log_info "PostgreSQL 备份完成"
else
    log_warn "PostgreSQL 未运行，跳过数据库备份"
fi

# ========== 备份 Redis ==========
log_info "备份 Redis 数据..."
if docker compose ps redis | grep -q "Up"; then
    docker compose exec -T redis \
        redis-cli SAVE
    docker run --rm \
        -v redis-data:/source:ro \
        -v "${BACKUP_DIR}":/backup \
        alpine:latest \
        tar czf /backup/redis-data.tar.gz -C /source .
    log_info "Redis 备份完成"
else
    log_warn "Redis 未运行，跳过缓存备份"
fi

# ========== 备份 Nginx 配置 ==========
log_info "备份 Nginx 配置..."
mkdir -p "${BACKUP_DIR}/nginx"
cp -r nginx/*.conf "${BACKUP_DIR}/nginx/" 2>/dev/null || true
cp -r nginx/certs "${BACKUP_DIR}/nginx/" 2>/dev/null || log_warn "证书目录不存在"

# ========== 创建备份清单 ==========
log_info "创建备份清单..."
cat > "${BACKUP_DIR}/manifest.txt" << EOF
OpenClaw Backup Manifest
========================
Backup Date: ${DATE}
Hostname: $(hostname)
Docker Version: $(docker --version)
Compose Version: $(docker compose version)

Included Files:
- docker-compose.yml
- .env
- nginx/nginx.conf
- openclaw-data.tar.gz
- openclaw-workspace.tar.gz
$(if [ -f "${BACKUP_DIR}/postgres-backup.sql.gz" ]; then echo "- postgres-backup.sql.gz"; fi)
$(if [ -f "${BACKUP_DIR}/redis-data.tar.gz" ]; then echo "- redis-data.tar.gz"; fi)

Backup Completed: $(date)
EOF

# ========== 验证备份 ==========
log_info "验证备份完整性..."
cd "${BACKUP_ROOT_DIR}"
BACKUP_SIZE=$(du -sh "${DATE}" | cut -f1)
log_info "备份大小: ${BACKUP_SIZE}"
log_info "文件列表:"
ls -la "${BACKUP_DIR}"

# ========== 清理旧备份 ==========
log_info "清理超过 ${RETENTION_DAYS} 天的旧备份..."
find "${BACKUP_ROOT_DIR}" -maxdepth 1 -type d -name "20*" -mtime +${RETENTION_DAYS} -exec rm -rf {} \; 2>/dev/null
OLD_COUNT=$(find "${BACKUP_ROOT_DIR}" -maxdepth 1 -type d -name "20*" | wc -l)
log_info "保留备份数量: ${OLD_COUNT}"

# ========== 完成 ==========
log_info "=========================================="
log_info "备份完成！"
log_info "备份位置: ${BACKUP_DIR}"
log_info "=========================================="

# 输出备份摘要
echo ""
echo "备份摘要:"
echo "  日期: ${DATE}"
echo "  大小: ${BACKUP_SIZE}"
echo "  包含:"
[ -f "${BACKUP_DIR}/openclaw-data.tar.gz" ] && echo "    - OpenClaw 数据"
[ -f "${BACKUP_DIR}/openclaw-workspace.tar.gz" ] && echo "    - 工作空间"
[ -f "${BACKUP_DIR}/postgres-backup.sql.gz" ] && echo "    - PostgreSQL 数据库"
[ -f "${BACKUP_DIR}/redis-data.tar.gz" ] && echo "    - Redis 缓存"
```

### 恢复脚本

```bash
#!/bin/bash
# restore.sh - OpenClaw 恢复脚本
# 使用方法：./restore.sh <备份日期>
# 示例：./restore.sh 20240115_030000

set -e

# ========== 配置 ==========
BACKUP_ROOT_DIR="/opt/backups/openclaw"

# 颜色输出
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

log_info() { echo -e "${GREEN}[INFO]${NC} $1"; }
log_warn() { echo -e "${YELLOW}[WARN]${NC} $1"; }
log_error() { echo -e "${RED}[ERROR]${NC} $1"; exit 1; }

# ========== 参数检查 ==========
if [ -z "$1" ]; then
    echo "使用方法: $0 <备份日期>"
    echo "示例: $0 20240115_030000"
    echo ""
    echo "可用备份:"
    ls -1 "${BACKUP_ROOT_DIR}"/20*/manifest.txt 2>/dev/null | cut -d/ -f5 || echo "  无备份"
    exit 1
fi

BACKUP_DATE=$1
BACKUP_DIR="${BACKUP_ROOT_DIR}/${BACKUP_DATE}"

# 检查备份是否存在
if [ ! -d "${BACKUP_DIR}" ]; then
    log_error "备份不存在: ${BACKUP_DIR}"
fi

log_info "开始从 ${BACKUP_DATE} 恢复..."

# ========== 警告 ==========
echo ""
log_warn "注意：这将覆盖当前数据！"
log_warn "按 Ctrl+C 取消，或等待 5 秒继续..."
sleep 5

# ========== 停止服务 ==========
log_info "停止所有服务..."
cd "$(dirname "$0")"
docker compose down

# ========== 恢复配置 ==========
log_info "恢复配置文件..."
cp "${BACKUP_DIR}/docker-compose.yml" .
if [ -f "${BACKUP_DIR}/.env" ]; then
    cp "${BACKUP_DIR}/.env" .
fi
if [ -f "${BACKUP_DIR}/nginx.conf" ]; then
    mkdir -p nginx
    cp "${BACKUP_DIR}/nginx.conf" nginx/
fi

# ========== 恢复 OpenClaw 数据卷 ==========
log_info "恢复 OpenClaw 数据卷..."

# 创建目标目录
mkdir -p volumes/openclaw-data
mkdir -p volumes/openclaw-workspace

# 恢复数据
docker run --rm \
    -v openclaw-data:/target \
    -v "${BACKUP_DIR}":/backup \
    alpine:latest \
    sh -c "rm -rf /target/* && tar xzf /backup/openclaw-data.tar.gz -C /target"

docker run --rm \
    -v openclaw-workspace:/target \
    -v "${BACKUP_DIR}":/backup \
    alpine:latest \
    sh -c "rm -rf /target/* && tar xzf /backup/openclaw-workspace.tar.gz -C /target"

# ========== 恢复 PostgreSQL ==========
if [ -f "${BACKUP_DIR}/postgres-backup.sql.gz" ]; then
    log_info "恢复 PostgreSQL 数据库..."

    # 启动 PostgreSQL
    docker compose up -d postgres
    sleep 10

    # 恢复数据
    gunzip -c "${BACKUP_DIR}/postgres-backup.sql.gz" | \
        docker compose exec -T postgres psql \
        -U ${POSTGRES_USER:-openclaw} \
        -d ${POSTGRES_DB:-openclaw}
fi

# ========== 恢复 Redis ==========
if [ -f "${BACKUP_DIR}/redis-data.tar.gz" ]; then
    log_info "恢复 Redis 数据..."

    docker run --rm \
        -v redis-data:/target \
        -v "${BACKUP_DIR}":/backup \
        alpine:latest \
        sh -c "rm -rf /target/* && tar xzf /backup/redis-data.tar.gz -C /target"
fi

# ========== 修复权限 ==========
log_info "修复文件权限..."
docker compose run --rm -u root openclaw-gateway \
    chown -R node:node /home/node/.openclaw 2>/dev/null || true

# ========== 启动服务 ==========
log_info "启动服务..."
docker compose up -d

# ========== 验证 ==========
sleep 10
log_info "验证服务状态..."
docker compose ps

log_info "=========================================="
log_info "恢复完成！"
log_info "=========================================="
```

### 定时备份配置

```bash
# 安装定时任务（每天凌晨 3 点执行备份）
crontab -e

# 添加以下行：
# 分 时 日 月 周 命令
# 0  3  *   *   *  cd /opt/openclaw && /opt/openclaw/backup.sh >> /var/log/openclaw-backup.log 2>&1

# 验证 crontab
crontab -l

# 查看备份日志
tail -f /var/log/openclaw-backup.log
```

---

## TLS 证书配置

### 生成自签名证书（仅用于开发/测试）

```bash
# 创建证书目录
mkdir -p nginx/certs nginx/dhparam

# 生成私钥和证书
openssl req -x509 -nodes -days 365 -newkey rsa:4096 \
    -keyout nginx/certs/privkey.pem \
    -out nginx/certs/fullchain.pem \
    -sha256 -subj "/CN=localhost/O=OpenClaw/C=CN"

# 生成 DH 参数（用于 Perfect Forward Secrecy）
openssl dhparam -out nginx/dhparam/dhparam.pem 4096

# 设置证书权限
chmod 600 nginx/certs/*.pem nginx/dhparam/*.pem
```

### 使用 Let's Encrypt（生产环境推荐）

```bash
# 安装 certbot
apt update && apt install -y certbot

# 停止 Nginx（如果运行）
docker compose stop nginx

# 获取证书（standalone 模式）
certbot certonly --standalone -d openclaw.yourdomain.com --agree-tos -m your@email.com --noninteractive

# 复制证书到配置目录
cp /etc/letsencrypt/live/openclaw.yourdomain.com/fullchain.pem nginx/certs/
cp /etc/letsencrypt/live/openclaw.yourdomain.com/privkey.pem nginx/certs/

# 生成 DH 参数
openssl dhparam -out nginx/dhparam/dhparam.pem 4096

# 重新启动 Nginx
docker compose up -d nginx

# 设置自动续期
echo "0 0 * * * certbot renew --noninteractive --deploy-hook 'docker compose exec -T nginx nginx -s reload'" | crontab -
```

---

## 目录结构

部署完成后，目录结构如下：

```
/opt/openclaw/
├── docker-compose.yml       # Docker Compose 配置
├── .env                     # 环境变量（敏感信息）
├── backup.sh                # 备份脚本
├── restore.sh               # 恢复脚本
├── nginx/
│   ├── nginx.conf           # Nginx 主配置
│   ├── certs/               # TLS 证书
│   │   ├── fullchain.pem    # 证书
│   │   └── privkey.pem      # 私钥
│   ├── dhparam/             # DH 参数
│   │   └── dhparam.pem
│   └── conf.d/              # 额外配置
│       ├── openclaw.conf    # OpenClaw 代理配置
│       └── security.conf    # 安全配置
├── volumes/                 # 数据卷挂载点
│   ├── openclaw-data/      # OpenClaw 配置数据
│   └── openclaw-workspace/ # 工作空间
└── backups/                 # 备份存储
    └── 20240115_030000/     # 备份快照
        ├── manifest.txt
        ├── docker-compose.yml
        ├── .env
        ├── openclaw-data.tar.gz
        ├── openclaw-workspace.tar.gz
        ├── postgres-backup.sql.gz
        └── redis-data.tar.gz
```

---

## 快速启动命令

```bash
# 1. 克隆或创建项目目录
cd /opt/openclaw

# 2. 创建目录结构
mkdir -p nginx/certs nginx/dhparam nginx/conf.d volumes

# 3. 复制配置文件（参考本文档）

# 4. 配置环境变量
cp .env.example .env
vim .env  # 编辑必填项

# 5. 生成 TLS 证书（开发环境）
openssl req -x509 -nodes -days 365 -newkey rsa:4096 \
    -keyout nginx/certs/privkey.pem \
    -out nginx/certs/fullchain.pem \
    -sha256 -subj "/CN=localhost"
openssl dhparam -out nginx/dhparam/dhparam.pem 4096

# 6. 启动服务
docker compose up -d

# 7. 查看状态
docker compose ps

# 8. 查看日志
docker compose logs -f

# 9. 验证健康状态
curl http://localhost/health

# 10. 打开浏览器访问
# https://localhost (开发环境使用 http://localhost)
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
docker compose logs -f openclaw-gateway

# 查看资源使用
docker stats

# 健康检查
docker inspect --format='{{.State.Health.Status}}' openclaw-gateway

# 进入容器
docker compose exec openclaw-gateway /bin/bash

# 更新服务
docker compose pull && docker compose up -d

# 清理旧镜像
docker image prune -f
```

---

## 相关章节

- [[../../01-架构总览/📐-架构概览]] - OpenClaw 架构
- [[📐-架构概览]] - OpenClaw 架构
- [[🔐-安全最佳实践]] - 安全加固指南
- [[🏠-知识库首页]] - 返回首页
