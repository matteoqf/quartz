---
type: procedure
tags:
  - Docker
  - 容器化
  - DevOps
  - 部署
创建时间: 2026-04-21
修改时间: 2026-04-21
关联版块:
---

# Docker容器化部署指南

## 概述

本文档提供将应用程序容器化的标准流程，涵盖从环境准备到生产环境部署的完整步骤。

## 前置条件

- 已安装 Docker Engine (>= 20.10)
- 已安装 Docker Compose (>= 2.0)
- 具有基础的命令行操作能力

## 操作步骤

### 1. 准备应用程序

1. 创建应用根目录并进入

```bash
mkdir myapp && cd myapp
```

2. 确保应用结构清晰，依赖文件已准备：

```
myapp/
├── src/
├── package.json  # Python: requirements.txt
├── Dockerfile
└── docker-compose.yml
```

### 2. 编写 Dockerfile

在项目根目录创建 `Dockerfile`：

```dockerfile
# 选择基础镜像
FROM node:20-alpine        # Node.js应用
# FROM python:3.11-slim   # Python应用
# FROM openjdk:17-slim    # Java应用

# 设置工作目录
WORKDIR /app

# 复制依赖文件
COPY package*.json ./

# 安装依赖
RUN npm ci --only=production

# 复制应用代码
COPY . .

# 暴露端口
EXPOSE 3000

# 启动命令
CMD ["node", "src/index.js"]
```

### 3. 创建 .dockerignore 文件

排除不需要打包进镜像的文件：

```
node_modules
.git
.env
*.log
dist
coverage
```

### 4. 编写 Docker Compose 配置

创建 `docker-compose.yml`：

```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgres://db:5432/myapp
    depends_on:
      - db
    restart: unless-stopped

  db:
    image: postgres:15-alpine
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password

volumes:
  postgres_data:
```

### 5. 构建镜像

```bash
# 构建镜像
docker build -t myapp:latest .

# 查看镜像
docker images | grep myapp
```

### 6. 本地测试

```bash
# 启动服务
docker compose up -d

# 查看日志
docker compose logs -f app

# 测试应用
curl http://localhost:3000/health

# 停止服务
docker compose down
```

### 7. 优化镜像体积

- 使用多阶段构建分离构建和运行环境
- 选择合适的基础镜像（alpine、slim）
- 合并 RUN 指令减少层数
- 合理利用 .dockerignore

示例多阶段构建：

```dockerfile
# 构建阶段
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# 运行阶段
FROM node:20-alpine AS runtime
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["node", "dist/index.js"]
```

## 生产环境部署

### 镜像仓库

```bash
# 登录仓库
docker login registry.example.com

# 打标签
docker tag myapp:latest registry.example.com/myapp:v1.0.0

# 推送镜像
docker push registry.example.com/myapp:v1.0.0
```

### 服务器部署

```bash
# 拉取镜像
docker pull registry.example.com/myapp:v1.0.0

# 使用 docker-compose 启动
docker compose -f docker-compose.prod.yml up -d
```

## 常见问题

| 问题 | 解决方案 |
|------|----------|
| 镜像构建失败 | 检查基础镜像可用性，确认依赖安装正常 |
| 端口冲突 | 修改 docker-compose.yml 中的端口映射 |
| 权限问题 | 检查宿主机目录权限，设置合适的用户 |
| 网络不通 | 配置网络模式，检查防火墙规则 |

## 相关资源

- [Docker 官方文档](https://docs.docker.com/)
- [Docker Compose 文档](https://docs.docker.com/compose/)
