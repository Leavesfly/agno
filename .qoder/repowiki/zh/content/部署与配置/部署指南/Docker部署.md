# Agno应用Docker部署指南

<cite>
**本文档中引用的文件**
- [README.md](file://README.md)
- [libs/agno/requirements.txt](file://libs/agno/requirements.txt)
- [libs/agno/pyproject.toml](file://libs/agno/pyproject.toml)
- [libs/agno_infra/pyproject.toml](file://libs/agno_infra/pyproject.toml)
- [libs/agno_infra/agno/docker/app/base.py](file://libs/agno_infra/agno/docker/app/base.py)
- [libs/agno_infra/agno/docker/resource/container.py](file://libs/agno_infra/agno/docker/resource/container.py)
- [libs/agno_infra/agno/docker/resource/network.py](file://libs/agno_infra/agno/docker/resource/network.py)
- [libs/agno_infra/agno/docker/resource/image.py](file://libs/agno_infra/agno/docker/resource/image.py)
- [libs/agno_infra/agno/docker/resources.py](file://libs/agno_infra/agno/docker/resources.py)
- [cookbook/demo/config.yaml](file://cookbook/demo/config.yaml)
- [cookbook/demo/run.py](file://cookbook/demo/run.py)
</cite>

## 目录
1. [简介](#简介)
2. [项目架构概览](#项目架构概览)
3. [Docker部署架构](#docker部署架构)
4. [Dockerfile配置](#dockerfile配置)
5. [Docker Compose编排](#docker-compose编排)
6. [多阶段构建优化](#多阶段构建优化)
7. [容器配置与环境变量](#容器配置与环境变量)
8. [数据库连接配置](#数据库连接配置)
9. [健康检查与监控](#健康检查与监控)
10. [部署最佳实践](#部署最佳实践)
11. [故障排除指南](#故障排除指南)
12. [总结](#总结)

## 简介

Agno是一个高性能的多智能体系统运行时框架，提供了完整的AgentOS运行时支持。本指南详细介绍了如何使用Docker部署Agno应用，包括从构建镜像到运行容器的完整流程。

Agno的核心优势在于其预构建的FastAPI应用和控制平面，这使得开发者可以立即开始构建产品，而无需从头开发基础设施。通过Docker部署，您可以确保在云端安全运行AgentOS，并获得完全的数据隐私保护。

## 项目架构概览

Agno项目采用模块化架构设计，主要包含以下核心组件：

```mermaid
graph TB
subgraph "Agno项目结构"
A[libs/agno] --> B[核心框架]
C[libs/agno_infra] --> D[基础设施管理]
E[cookbook] --> F[示例和演示]
G[scripts] --> H[部署脚本]
end
subgraph "Docker部署层"
I[Dockerfile] --> J[基础镜像]
K[Docker Compose] --> L[服务编排]
M[容器配置] --> N[环境变量]
end
B --> I
D --> K
F --> M
```

**图表来源**
- [libs/agno/pyproject.toml](file://libs/agno/pyproject.toml#L1-L50)
- [libs/agno_infra/pyproject.toml](file://libs/agno_infra/pyproject.toml#L1-L50)

**章节来源**
- [README.md](file://README.md#L1-L140)
- [libs/agno/pyproject.toml](file://libs/agno/pyproject.toml#L1-L100)

## Docker部署架构

Agno的Docker部署采用分层架构设计，支持多种部署场景：

```mermaid
graph LR
subgraph "Docker部署架构"
A[客户端应用] --> B[负载均衡器]
B --> C[Agno容器]
C --> D[数据库容器]
C --> E[缓存容器]
C --> F[存储容器]
subgraph "容器类型"
G[应用容器]
H[数据容器]
I[网络容器]
end
C -.-> G
D -.-> H
E -.-> H
F -.-> H
end
```

**图表来源**
- [libs/agno_infra/agno/docker/app/base.py](file://libs/agno_infra/agno/docker/app/base.py#L1-L100)
- [libs/agno_infra/agno/docker/resource/container.py](file://libs/agno_infra/agno/docker/resource/container.py#L1-L100)

## Dockerfile配置

### 基础Dockerfile示例

以下是为Agno应用设计的基础Dockerfile配置：

```dockerfile
FROM python:3.11-slim

# 设置工作目录
WORKDIR /app

# 安装系统依赖
RUN apt-get update && apt-get install -y \
    curl \
    git \
    && rm -rf /var/lib/apt/lists/*

# 创建非root用户
RUN groupadd -r agno && useradd -r -g agno agno

# 复制依赖文件
COPY libs/agno/requirements.txt .
COPY libs/agno_infra/requirements.txt .

# 安装Python依赖
RUN pip install --no-cache-dir -r requirements.txt

# 复制应用代码
COPY libs/agno/ ./libs/agno/
COPY libs/agno_infra/ ./libs/agno_infra/

# 设置环境变量
ENV PYTHONPATH=/app
ENV AGNO_RUNTIME=docker
ENV INSTALL_REQUIREMENTS=true

# 切换到非root用户
USER agno

# 暴露端口
EXPOSE 7777

# 健康检查
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:7777/health || exit 1

# 启动命令
CMD ["python", "-m", "agno.os", "--port", "7777"]
```

### 多阶段构建优化

为了进一步优化镜像大小和安全性，可以使用多阶段构建：

```dockerfile
# 阶段1：构建阶段
FROM python:3.11-alpine AS builder

WORKDIR /app

# 安装构建工具
RUN apk add --no-cache gcc musl-dev linux-headers

# 复制依赖文件
COPY libs/agno/requirements.txt .
COPY libs/agno_infra/requirements.txt .

# 安装构建依赖
RUN pip wheel --no-cache-dir --no-deps --wheel-dir /wheels -r requirements.txt

# 阶段2：运行阶段
FROM python:3.11-slim AS runtime

WORKDIR /app

# 安装运行时依赖
RUN apt-get update && apt-get install -y \
    curl \
    && rm -rf /var/lib/apt/lists/*

# 创建非root用户
RUN groupadd -r agno && useradd -r -g agno agno

# 复制构建好的wheel包
COPY --from=builder /wheels /wheels

# 安装应用依赖
RUN pip install --no-cache /wheels/* \
    && rm -rf /wheels

# 复制应用代码
COPY libs/agno/ ./libs/agno/
COPY libs/agno_infra/ ./libs/agno_infra/

# 设置环境变量
ENV PYTHONPATH=/app
ENV AGNO_RUNTIME=docker
ENV INSTALL_REQUIREMENTS=true

# 切换到非root用户
USER agno

# 暴露端口
EXPOSE 7777

# 健康检查
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:7777/health || exit 1

# 启动命令
CMD ["python", "-m", "agno.os", "--port", "7777"]
```

**章节来源**
- [libs/agno/requirements.txt](file://libs/agno/requirements.txt#L1-L72)
- [libs/agno/pyproject.toml](file://libs/agno/pyproject.toml#L25-L40)

## Docker Compose编排

### 基础docker-compose.yml配置

```yaml
version: '3.8'

services:
  agno-app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: agno-app
    restart: unless-stopped
    ports:
      - "7777:7777"
    environment:
      - AGNO_RUNTIME=docker
      - INSTALL_REQUIREMENTS=true
      - PYTHONPATH=/app
      - DATABASE_URL=${DATABASE_URL}
      - REDIS_URL=${REDIS_URL}
    volumes:
      - ./workspace:/app/workspace
      - ./logs:/app/logs
      - ~/.config/ag:/app/.config/ag:ro
    networks:
      - agno-network
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:7777/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 30s
    depends_on:
      - postgres
      - redis
      - minio

  postgres:
    image: postgres:15
    container_name: agno-postgres
    restart: unless-stopped
    environment:
      - POSTGRES_DB=agno
      - POSTGRES_USER=agno
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - agno-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U agno"]
      interval: 30s
      timeout: 10s
      retries: 3

  redis:
    image: redis:7-alpine
    container_name: agno-redis
    restart: unless-stopped
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data
    networks:
      - agno-network
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 30s
      timeout: 10s
      retries: 3

  minio:
    image: minio/minio
    container_name: agno-minio
    restart: unless-stopped
    ports:
      - "9000:9000"
      - "9001:9001"
    environment:
      - MINIO_ROOT_USER=${MINIO_ROOT_USER}
      - MINIO_ROOT_PASSWORD=${MINIO_ROOT_PASSWORD}
    volumes:
      - minio_data:/data
    command: server /data --console-address ":9001"
    networks:
      - agno-network
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 10s
      retries: 3

volumes:
  postgres_data:
  redis_data:
  minio_data:

networks:
  agno-network:
    driver: bridge
```

### 高可用部署配置

```yaml
version: '3.8'

services:
  agno-app-1:
    <<: *agno-app-base
    container_name: agno-app-1
    ports:
      - "7777:7777"
    networks:
      - agno-network

  agno-app-2:
    <<: *agno-app-base
    container_name: agno-app-2
    ports:
      - "7778:7777"
    networks:
      - agno-network

  nginx:
    image: nginx:alpine
    container_name: agno-nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./ssl:/etc/nginx/ssl:ro
    networks:
      - agno-network
    depends_on:
      - agno-app-1
      - agno-app-2

  postgres:
    <<: *postgres-base
    container_name: agno-postgres-primary
    volumes:
      - postgres_primary_data:/var/lib/postgresql/data
    networks:
      - agno-network

  postgres-replica:
    <<: *postgres-base
    container_name: agno-postgres-replica
    environment:
      - POSTGRES_REPLICATION_MODE=SLAVE
      - POSTGRES_MASTER_HOST=agno-postgres-primary
    volumes:
      - postgres_replica_data:/var/lib/postgresql/data
    networks:
      - agno-network
    depends_on:
      - postgres

  redis-cluster:
    image: redis:7-alpine
    container_name: agno-redis-cluster
    restart: unless-stopped
    command: redis-server --cluster-enabled yes --cluster-config-file nodes.conf --appendonly yes
    volumes:
      - redis_cluster_data:/data
    networks:
      - agno-network

networks:
  agno-network:
    driver: overlay
    attachable: true
```

**章节来源**
- [libs/agno_infra/agno/docker/app/base.py](file://libs/agno_infra/agno/docker/app/base.py#L25-L100)
- [libs/agno_infra/agno/docker/resource/network.py](file://libs/agno_infra/agno/docker/resource/network.py#L1-L50)

## 多阶段构建优化

### 构建优化策略

Agno的多阶段构建优化主要体现在以下几个方面：

```mermaid
flowchart TD
A[源代码] --> B[构建阶段]
B --> C[依赖安装]
C --> D[编译优化]
D --> E[镜像压缩]
E --> F[运行阶段]
F --> G[应用启动]
subgraph "优化技术"
H[多阶段构建]
I[最小化镜像]
J[缓存优化]
K[安全加固]
end
B -.-> H
E -.-> I
C -.-> J
F -.-> K
```

**图表来源**
- [libs/agno_infra/agno/docker/resource/image.py](file://libs/agno_infra/agno/docker/resource/image.py#L1-L100)

### 安全配置

```dockerfile
# 使用Alpine Linux作为基础镜像
FROM python:3.11-alpine AS builder

# 安装安全更新
RUN apk add --no-cache \
    ca-certificates \
    tzdata \
    && update-ca-certificates

# 创建非特权用户
RUN addgroup -S agno && adduser -S agno -G agno

# 设置安全环境变量
ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PIP_NO_CACHE_DIR=1 \
    PIP_DISABLE_PIP_VERSION_CHECK=1

# 使用非root用户构建
USER agno

# 最终运行镜像
FROM python:3.11-alpine AS runtime

# 安装运行时依赖
RUN apk add --no-cache \
    libstdc++ \
    && addgroup -S agno && adduser -S agno -G agno

# 复制构建产物
COPY --from=builder --chown=agno:agno /app /app

# 设置工作目录
WORKDIR /app

# 切换到非root用户
USER agno

# 设置安全的启动命令
ENTRYPOINT ["su-exec", "agno", "python", "-m", "agno.os"]
```

**章节来源**
- [libs/agno_infra/agno/docker/resource/image.py](file://libs/agno_infra/agno/docker/resource/image.py#L100-L200)

## 容器配置与环境变量

### 核心环境变量配置

Agno应用需要配置以下关键环境变量：

```yaml
environment:
  # 运行时配置
  - AGNO_RUNTIME=docker
  - INSTALL_REQUIREMENTS=true
  - PRINT_ENV_ON_LOAD=true
  
  # 应用配置
  - PYTHONPATH=/app
  - RESOURCES_DIR_CONTAINER_PATH=/mnt/resources
  - MOUNT_RESOURCES=true
  - MOUNT_INFRA_DIR=true
  
  # 数据库配置
  - DATABASE_URL=postgresql://agno:${POSTGRES_PASSWORD}@postgres:5432/agno
  - REDIS_URL=redis://redis:6379
  
  # 存储配置
  - MINIO_ENDPOINT=http://minio:9000
  - MINIO_ACCESS_KEY=${MINIO_ROOT_USER}
  - MINIO_SECRET_KEY=${MINIO_ROOT_PASSWORD}
  
  # 日志配置
  - LOG_LEVEL=INFO
  - LOG_FORMAT=json
  
  # 性能配置
  - MAX_WORKERS=4
  - TIMEOUT_SECONDS=30
```

### 容器资源限制

```yaml
services:
  agno-app:
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 4G
        reservations:
          cpus: '1.0'
          memory: 2G
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
```

**章节来源**
- [libs/agno_infra/agno/docker/app/base.py](file://libs/agno_infra/agno/docker/app/base.py#L100-L200)

## 数据库连接配置

### PostgreSQL配置

```yaml
postgres:
  image: postgres:15
  container_name: agno-postgres
  restart: unless-stopped
  environment:
    - POSTGRES_DB=agno
    - POSTGRES_USER=agno
    - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
    - POSTGRES_MULTIPLE_EXTENSIONS=pg_trgm,uuid-ossp,pgcrypto
  volumes:
    - postgres_data:/var/lib/postgresql/data
    - ./init.sql:/docker-entrypoint-initdb.d/init.sql:ro
  networks:
    - agno-network
  healthcheck:
    test: ["CMD-SHELL", "pg_isready -U agno"]
    interval: 30s
    timeout: 10s
    retries: 3
```

### Redis配置

```yaml
redis:
  image: redis:7-alpine
  container_name: agno-redis
  restart: unless-stopped
  command: redis-server --appendonly yes --maxmemory 2gb --maxmemory-policy allkeys-lru
  volumes:
    - redis_data:/data
  networks:
    - agno-network
  sysctls:
    - net.core.somaxconn=65535
  ulimits:
    nproc: 65535
    nofile:
      soft: 65535
      hard: 65535
```

### MinIO对象存储配置

```yaml
minio:
  image: minio/minio
  container_name: agno-minio
  restart: unless-stopped
  ports:
    - "9000:9000"
    - "9001:9001"
  environment:
    - MINIO_ROOT_USER=${MINIO_ROOT_USER}
    - MINIO_ROOT_PASSWORD=${MINIO_ROOT_PASSWORD}
    - MINIO_STORAGE_CLASS_STANDARD=EC:4
  volumes:
    - minio_data:/data
  command: server /data --console-address ":9001"
  networks:
    - agno-network
  healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
    interval: 30s
    timeout: 10s
    retries: 3
```

**章节来源**
- [libs/agno_infra/agno/docker/app/base.py](file://libs/agno_infra/agno/docker/app/base.py#L200-L300)

## 健康检查与监控

### 容器健康检查配置

```yaml
services:
  agno-app:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:7777/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 30s
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
        labels: "service=agno-app"
```

### 监控指标配置

```yaml
prometheus:
  image: prom/prometheus
  container_name: agno-prometheus
  restart: unless-stopped
  ports:
    - "9090:9090"
  volumes:
    - ./prometheus.yml:/etc/prometheus/prometheus.yml
    - prometheus_data:/prometheus
  networks:
    - agno-network

grafana:
  image: grafana/grafana
  container_name: agno-grafana
  restart: unless-stopped
  ports:
    - "3000:3000"
  environment:
    - GF_SECURITY_ADMIN_PASSWORD=admin
  volumes:
    - grafana_data:/var/lib/grafana
  networks:
    - agno-network
```

### 自定义健康检查

```python
# health_check.py
import requests
import time
from typing import Dict, Any

def check_service_health(url: str, timeout: int = 30) -> Dict[str, Any]:
    """检查服务健康状态"""
    try:
        response = requests.get(f"{url}/health", timeout=timeout)
        if response.status_code == 200:
            return {"status": "healthy", "response_time": response.elapsed.total_seconds()}
        else:
            return {"status": "unhealthy", "code": response.status_code}
    except requests.exceptions.RequestException as e:
        return {"status": "unhealthy", "error": str(e)}

def check_database_connection(db_url: str) -> Dict[str, Any]:
    """检查数据库连接"""
    import psycopg2
    try:
        conn = psycopg2.connect(db_url)
        cursor = conn.cursor()
        cursor.execute("SELECT 1")
        cursor.close()
        conn.close()
        return {"status": "healthy"}
    except Exception as e:
        return {"status": "unhealthy", "error": str(e)}
```

## 部署最佳实践

### 镜像构建最佳实践

```bash
#!/bin/bash
# build-image.sh

# 设置环境变量
export DOCKER_BUILDKIT=1
export COMPOSE_DOCKER_CLI_BUILD=1

# 构建多平台镜像
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  --tag agno/app:latest \
  --tag agno/app:v2.0.7 \
  --push \
  .

# 验证镜像
docker run --rm -it agno/app:latest python -c "import agno; print(agno.__version__)"
```

### 部署脚本

```bash
#!/bin/bash
# deploy.sh

set -e

# 环境变量验证
if [[ -z "${DOCKER_REGISTRY}" ]]; then
    echo "错误: DOCKER_REGISTRY未设置"
    exit 1
fi

# 创建网络
docker network create agno-network || true

# 部署应用
docker-compose up -d

# 等待服务启动
echo "等待Agno服务启动..."
sleep 30

# 验证部署
docker-compose ps
docker-compose logs -n 50 agno-app

echo "部署完成!"
```

### CI/CD集成

```yaml
# .github/workflows/deploy.yml
name: Deploy Agno Application

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v3
        with:
          python-version: '3.11'
          
      - name: Install dependencies
        run: |
          pip install -r libs/agno/requirements.txt
          pip install -r libs/agno_infra/requirements.txt
          
      - name: Run tests
        run: pytest libs/agno/tests/

  build-and-push:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v3
      
      - name: Login to Docker Registry
        uses: docker/login-action@v2
        with:
          registry: ${{ secrets.DOCKER_REGISTRY }}
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
      
      - name: Build and push Docker image
        run: |
          docker buildx create --use
          docker buildx build \
            --platform linux/amd64,linux/arm64 \
            --tag ${{ secrets.DOCKER_REGISTRY }}/agno/app:${{ github.sha }} \
            --tag ${{ secrets.DOCKER_REGISTRY }}/agno/app:latest \
            --push \
            .
```

**章节来源**
- [libs/agno_infra/agno/docker/resources.py](file://libs/agno_infra/agno/docker/resources.py#L1-L100)

## 故障排除指南

### 常见问题诊断

```bash
# 检查容器状态
docker ps -a

# 查看容器日志
docker logs agno-app

# 检查网络连接
docker exec agno-app ping postgres
docker exec agno-app ping redis

# 检查资源使用情况
docker stats agno-app

# 进入容器调试
docker exec -it agno-app bash
```

### 性能调优

```yaml
# 调优后的容器配置
services:
  agno-app:
    deploy:
      resources:
        limits:
          cpus: '4.0'
          memory: 8G
        reservations:
          cpus: '2.0'
          memory: 4G
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 5
    environment:
      - PYTHONOPTIMIZE=1
      - PYTHONHASHSEED=random
      - OMP_NUM_THREADS=1
      - OPENBLAS_NUM_THREADS=1
      - MKL_NUM_THREADS=1
      - VECLIB_MAXIMUM_THREADS=1
      - NUMEXPR_NUM_THREADS=1
```

### 数据备份与恢复

```bash
#!/bin/bash
# backup.sh

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backups/$DATE"

mkdir -p $BACKUP_DIR

# 备份PostgreSQL
docker exec agno-postgres pg_dumpall -U agno | gzip > $BACKUP_DIR/postgres.sql.gz

# 备份Redis
docker exec agno-redis redis-cli BGSAVE
docker cp agno-redis:/data/dump.rdb $BACKUP_DIR/redis.rdb

# 备份MinIO
mc mirror minio-source/ minio-backup/

echo "备份完成: $BACKUP_DIR"
```

### 监控告警配置

```yaml
# alertmanager.yml
global:
  smtp_smarthost: 'smtp.gmail.com:587'
  smtp_from: 'alerts@example.com'

route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'web.hook'

receivers:
- name: 'web.hook'
  email_configs:
  - to: 'admin@example.com'
    subject: 'Agno Alert: {{ .GroupLabels.alertname }}'
    body: |
      {{ range .Alerts }}
      Service: {{ .Labels.service }}
      Status: {{ .Status }}
      Details: {{ .Annotations.description }}
      {{ end }}
```

## 总结

本指南详细介绍了如何使用Docker部署Agno应用，涵盖了从基础Dockerfile配置到高级容器编排的各个方面。通过遵循这些最佳实践，您可以：

1. **高效构建**：使用多阶段构建优化镜像大小和安全性
2. **可靠部署**：通过Docker Compose实现服务编排和依赖管理
3. **高可用性**：配置负载均衡和故障转移机制
4. **可观测性**：集成监控和日志收集系统
5. **自动化运维**：实现CI/CD流水线和自动部署

Agno的Docker部署方案不仅简化了开发和运维流程，还确保了应用在生产环境中的稳定性和可扩展性。通过合理配置容器资源、网络和存储，您可以构建一个高性能、高可用的多智能体系统运行环境。