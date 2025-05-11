# Dify 使用 Docker Compose 部署教程

## 目录

- [项目介绍](#项目介绍)
- [环境要求](#环境要求)
- [快速开始](#快速开始)
- [详细配置](#详细配置)
  - [基础环境变量](#基础环境变量)
  - [数据库配置](#数据库配置)
  - [Redis配置](#redis配置)
  - [存储配置](#存储配置)
  - [向量数据库配置](#向量数据库配置)
  - [插件系统配置](#插件系统配置)
  - [邮件服务配置](#邮件服务配置)
  - [其他配置](#其他配置)
- [服务介绍](#服务介绍)
  - [API服务](#api服务)
  - [Worker服务](#worker服务)
  - [Web前端](#web前端)
  - [沙箱服务](#沙箱服务)
  - [插件守护进程](#插件守护进程)
  - [SSRF代理](#ssrf代理)
  - [非结构化数据处理服务](#非结构化数据处理服务)
  - [Nginx服务](#nginx服务)
  - [Redis服务](#redis服务)
- [目录结构](#目录结构)
- [常见问题](#常见问题)
- [升级指南](#升级指南)
- [贡献指南](#贡献指南)
- [许可证](#许可证)

## 项目介绍

本项目提供了一套完整的 Docker Compose 配置，用于快速部署 Dify 平台。Dify 是一个强大的 LLM 应用开发平台，支持构建各种基于大语言模型的应用，如聊天机器人、知识库问答等。

本项目中的 `docker-compose.yml` 和 `conf` 目录下的配置文件都是从 [GitHub Dify项目](https://github.com/langgenius/dify) 中复制出来的，经过了精心的整理和补充，方便用户快速部署。

## 环境要求

- Docker >= 20.10.0
- Docker Compose >= v2.0.0
- 至少4GB内存
- 至少20GB磁盘空间

### Docker 和 Docker Compose 安装

Docker 和 Docker Compose 的安装请参考[官方文档](https://docs.docker.com/engine/install/)，这里不再赘述。

## 快速开始

1. 克隆本仓库：

```bash
git clone https://github.com/jiaxudonggit/dify-deployment.git
cd dify-deployment
```

2. 创建配置文件：

```bash
# 复制示例配置文件
cp .env.example .env

# 编辑配置文件，按需修改
vim .env
```

3. 创建必要的目录：

```bash
mkdir -p storage/logs nginx/logs pgvector/data redis/data
```

4. 启动服务：

```bash
# 启动所有服务
docker-compose up -d
```

5. 访问服务：

浏览器访问 `http://localhost` 即可看到 Dify 的登录页面。可以通过浏览器访问 `http://localhost/install` 来访问 Dify 仪表板并开始初始化过程。

## 详细配置

### 基础环境变量

以下是一些基础环境变量，您可以在 `.env` 文件中进行配置：

| 变量名 | 说明 | 默认值 | 示例 |
|------|------|------|------|
| CONSOLE_API_URL | 控制台API URL | 空 | `http://localhost/console/api` |
| CONSOLE_WEB_URL | 控制台Web URL | 空 | `http://localhost` |
| SERVICE_API_URL | 服务API URL | 空 | `http://localhost/api` |
| APP_API_URL | 应用API URL | 空 | `http://localhost/v1` |
| APP_WEB_URL | 应用Web URL | 空 | `http://localhost` |
| FILES_URL | 文件服务URL | 空 | `http://localhost/files` |
| CONTAINER_NAME | 容器名称前缀 | dify | `myproject` |
| NGINX_PORT | Nginx HTTP端口 | 80 | `8080` |
| NGINX_SSL_PORT | Nginx HTTPS端口 | 443 | `8443` |
| DIFY_ROOT_PATH | 数据存储根路径 | ./storage | `/data/dify` |

### 数据库配置

Dify 使用 PostgreSQL 作为主数据库，配置如下：

| 变量名 | 说明 | 默认值 | 示例 |
|------|------|------|------|
| DB_USERNAME | 数据库用户名 | postgres | `dify` |
| DB_PASSWORD | 数据库密码 | difyai123456 | `your-strong-password` |
| DB_HOST | 数据库主机 | db | `postgres` |
| DB_PORT | 数据库端口 | 5432 | `5432` |
| DB_DATABASE | 数据库名称 | dify | `mydify` |
| SQLALCHEMY_POOL_SIZE | 数据库连接池大小 | 30 | `50` |
| SQLALCHEMY_POOL_RECYCLE | 连接池回收时间(秒) | 3600 | `1800` |

### Redis配置

Dify 使用 Redis 作为缓存和消息队列：

| 变量名 | 说明 | 默认值 | 示例 |
|------|------|------|------|
| REDIS_HOST | Redis主机 | redis | `redis` |
| REDIS_PORT | Redis端口 | 6379 | `6379` |
| REDIS_USERNAME | Redis用户名 | 空 | `default` |
| REDIS_PASSWORD | Redis密码 | difyai123456 | `your-redis-password` |
| REDIS_DB | Redis数据库索引 | 0 | `1` |
| CELERY_BROKER_URL | Celery消息队列URL | redis://:difyai123456@redis:6379/1 | `redis://:password@redis:6379/2` |
| REDIS_PORT_EXPOSED | Redis暴露端口 | 6379 | `6379` |

### 存储配置

Dify 支持多种存储方式，包括本地存储、S3、Azure Blob等：

| 变量名 | 说明 | 默认值 | 示例 |
|------|------|------|------|
| STORAGE_TYPE | 存储类型 | opendal | `s3` |
| OPENDAL_SCHEME | OpenDAL模式(本地存储) | fs | `fs` |
| OPENDAL_FS_ROOT | 本地存储根目录 | storage | `/data/storage` |
| S3_ENDPOINT | S3端点 | 空 | `https://s3.amazonaws.com` |
| S3_REGION | S3区域 | us-east-1 | `us-west-2` |
| S3_BUCKET_NAME | S3存储桶名称 | difyai | `my-dify-bucket` |
| S3_ACCESS_KEY | S3访问密钥 | 空 | `your-access-key` |
| S3_SECRET_KEY | S3秘密密钥 | 空 | `your-secret-key` |
| S3_USE_AWS_MANAGED_IAM | 使用AWS管理的IAM | false | `true` |

### 向量数据库配置

本项目默认使用 pgvector 作为向量数据库，配置如下：

| 变量名 | 说明 | 默认值 | 示例 |
|------|------|------|------|
| VECTOR_STORE | 向量数据库类型 | pgvector | `pgvector` |
| PGVECTOR_HOST | PGVector主机 | pgvector | `pgvector` |
| PGVECTOR_PORT | PGVector端口 | 5432 | `5432` |
| PGVECTOR_USER | PGVector用户名 | postgres | `postgres` |
| PGVECTOR_PASSWORD | PGVector密码 | difyai123456 | `your-password` |
| PGVECTOR_DATABASE | PGVector数据库名 | dify | `dify_vector` |
| PGVECTOR_PORT_EXPOSED | PGVector暴露的端口 | 5432 | `5432` |
| PGVECTOR_PG_BIGM | 启用pg_bigm模块 | false | `true` |

pgvector是一个强大的开源向量数据库扩展，它为PostgreSQL添加了向量相似度搜索功能，非常适合AI应用的向量检索需求。相比其他向量数据库，pgvector具有以下优势：

1. 与PostgreSQL无缝集成，可以利用PostgreSQL成熟的事务、备份和扩展生态
2. 支持多种距离计算方法：欧几里得距离、余弦相似度和内积
3. 支持HNSW索引，查询性能优异
4. 开源免费，部署简单

Dify也支持其他向量数据库，如Weaviate、Qdrant、Milvus等，可通过修改`VECTOR_STORE`环境变量来切换，但是需要自行部署向量数据库。

### 插件系统配置

Dify 支持插件系统，以下是插件相关配置：

| 变量名 | 说明 | 默认值 | 示例 |
|------|------|------|------|
| PLUGIN_DAEMON_PORT | 插件守护进程端口 | 5002 | `5002` |
| PLUGIN_DAEMON_KEY | 插件守护进程密钥 | lYkiYYT6owG+71oLerGzA7GXCgOT++6ovaezWAjpCjf+Sjc3ZtU+qUEi | `your-key` |
| PLUGIN_DIFY_INNER_API_URL | 插件内部API URL | <http://dify-api:5001> | `http://api:5001` |
| PLUGIN_DIFY_INNER_API_KEY | 插件内部API密钥 | QaHbTe77CtuXmsfyhR7+vRjI/+XbV1AaFy691iy+kGDv2Jvy0/eAh8Y1 | `your-key` |
| PLUGIN_MAX_PACKAGE_SIZE | 插件包最大大小(字节) | 52428800 | `104857600` |
| FORCE_VERIFYING_SIGNATURE | 强制验证插件签名 | true | `false` |
| PLUGIN_PYTHON_ENV_INIT_TIMEOUT | 插件Python环境初始化超时(秒) | 120 | `180` |
| PLUGIN_MAX_EXECUTION_TIMEOUT | 插件最大执行超时(秒) | 600 | `900` |

### 邮件服务配置

Dify 支持邮件服务，用于发送通知、重置密码等：

| 变量名 | 说明 | 默认值 | 示例 |
|------|------|------|------|
| MAIL_TYPE | 邮件服务类型 | resend | `smtp` |
| MAIL_DEFAULT_SEND_FROM | 发信人地址 | 空 | `noreply@yourdomain.com` |
| RESEND_API_URL | Resend API URL | <https://api.resend.com> | `https://api.resend.com` |
| RESEND_API_KEY | Resend API密钥 | your-resend-api-key | `your-key` |
| SMTP_SERVER | SMTP服务器 | 空 | `smtp.gmail.com` |
| SMTP_PORT | SMTP端口 | 465 | `587` |
| SMTP_USERNAME | SMTP用户名 | 空 | `user@gmail.com` |
| SMTP_PASSWORD | SMTP密码 | 空 | `your-password` |
| SMTP_USE_TLS | 使用TLS | true | `false` |

### 其他配置

还有一些其他重要配置：

| 变量名 | 说明 | 默认值 | 示例 |
|------|------|------|------|
| LOG_LEVEL | 日志级别 | INFO | `DEBUG` |
| SECRET_KEY | 应用密钥 | sk-9f73s3ljTXVcMT3Blb3ljTqtsKiGHXVcMT3BlbkFJLK7U | `your-secret-key` |
| INIT_PASSWORD | 初始密码 | 空 | `Admin@2023` |
| DEPLOY_ENV | 部署环境 | PRODUCTION | `DEVELOPMENT` |
| DEBUG | 调试模式 | false | `true` |
| FLASK_DEBUG | Flask调试模式 | false | `true` |
| UPLOAD_FILE_SIZE_LIMIT | 上传文件大小限制(MB) | 100 | `200` |
| UPLOAD_FILE_BATCH_LIMIT | 批量上传文件限制 | 10 | `20` |
| ACCESS_TOKEN_EXPIRE_MINUTES | 访问令牌过期时间(分钟) | 60 | `120` |
| REFRESH_TOKEN_EXPIRE_DAYS | 刷新令牌过期时间(天) | 30 | `60` |

## 服务介绍

### API服务

API服务是Dify的核心服务，提供所有API接口：

```yaml
api:
  container_name: ${CONTAINER_NAME}-api
  image: langgenius/dify-api:1.3.1
  restart: always
  environment:
    MODE: api
  ports:
    - "${API_PORT:-5001}:5001"
  volumes:
    - ${DIFY_ROOT_PATH}/storage:/app/api/storage
```

### Worker服务

Worker服务用于处理异步任务和队列：

```yaml
worker:
  container_name: ${CONTAINER_NAME}-worker
  image: langgenius/dify-api:1.3.1
  restart: always
  environment:
    MODE: worker
  volumes:
    - ${DIFY_ROOT_PATH}/storage:/app/api/storage
```

### Web前端

Web前端提供用户界面：

```yaml
web:
  container_name: ${CONTAINER_NAME}-web
  image: langgenius/dify-web:1.3.1
  restart: always
  ports:
    - "${WEB_PORT:-3000}:3000"
```

### 沙箱服务

沙箱服务用于安全执行代码：

```yaml
sandbox:
  container_name: ${CONTAINER_NAME}-sandbox
  image: langgenius/dify-sandbox:0.2.11
  restart: always
  environment:
    API_KEY: ${SANDBOX_API_KEY:-dify-sandbox}
  volumes:
    - ./conf/volumes/sandbox/dependencies:/dependencies
    - ./conf/volumes/sandbox/conf:/conf
```

### 插件守护进程

插件守护进程管理Dify的插件系统：

```yaml
plugin_daemon:
  container_name: ${CONTAINER_NAME}-plugin_daemon
  image: langgenius/dify-plugin-daemon:0.0.9-local
  restart: always
  ports:
    - "${PANEL_PLUGIN_DAEMON_PORT:-5002}:${PLUGIN_DAEMON_PORT:-5002}"
    - "${EXPOSE_PLUGIN_DEBUGGING_PORT:-5003}:${PLUGIN_DEBUGGING_PORT:-5003}"
  volumes:
    - ${DIFY_ROOT_PATH}/plugin_daemon:/app/storage
```

### SSRF代理

SSRF代理用于防止服务器端请求伪造攻击：

```yaml
ssrf_proxy:
  container_name: ${CONTAINER_NAME}-ssrf_proxy
  image: ubuntu/squid:latest
  restart: always
  volumes:
    - ./conf/ssrf_proxy/squid.conf.template:/etc/squid/squid.conf.template
    - ./conf/ssrf_proxy/docker-entrypoint.sh:/docker-entrypoint-mount.sh
  ports:
    - "${SSRF_HTTP_PORT:-3128}:3128"
```

### 非结构化数据处理服务

非结构化数据处理服务用于处理非结构化数据：

```yaml
unstructured:
  container_name: ${CONTAINER_NAME}-unstructured
  image: downloads.unstructured.io/unstructured-io/unstructured-api:latest
  restart: always
  ports:
    - "${UNSTRUCTURED_PORT:-8000}:8000"
  volumes:
    - ${DIFY_ROOT_PATH}/unstructured:/app/data
```

### Nginx服务

Nginx服务作为反向代理，将请求路由到正确的服务：

```yaml
nginx:
  container_name: ${CONTAINER_NAME}-nginx
  image: nginx:latest
  restart: always
  ports:
    - "${NGINX_PORT:-80}:80"
    - "${NGINX_SSL_PORT:-443}:443"
  volumes:
    - ./conf/nginx/nginx.conf:/etc/nginx/nginx.conf
    - ./conf/nginx/conf.d:/etc/nginx/conf.d
    - ./conf/nginx/ssl:/etc/nginx/ssl
    - ${DIFY_ROOT_PATH}/nginx/logs:/var/log/nginx
  depends_on:
    - api
    - web
    - plugin_daemon
```

### Redis服务

Redis服务用于缓存和消息队列处理：

```yaml
redis:
  container_name: ${CONTAINER_NAME}-redis
  image: redis:7-alpine
  restart: always
  command: >
    --requirepass ${REDIS_PASSWORD:-difyai123456}
    --appendonly yes
    --maxmemory 512mb
    --maxmemory-policy volatile-lru
  volumes:
    - ${DIFY_ROOT_PATH}/redis/data:/data
```

Redis是一个开源的内存数据结构存储，被Dify用作：

1. 缓存系统，提高应用响应速度
2. Celery任务队列的消息代理
3. 会话数据存储
4. 临时状态管理

## 目录结构

```
├── .env                      # 环境变量配置文件
├── docker-compose.yml        # Docker Compose 配置文件
├── conf/                     # 配置文件目录
│   ├── nginx/                # Nginx 配置
│   │   ├── conf.d/           # Nginx 站点配置
│   │   │   └── default.conf  # 默认站点配置
│   │   ├── ssl/              # SSL 证书
│   │   └── nginx.conf        # Nginx 主配置
│   ├── ssrf_proxy/           # SSRF 代理配置
│   ├── volumes/              # 持久化存储卷
│   │   ├── sandbox/          # 沙箱配置
│   │   │   ├── conf/         # 沙箱配置文件
│   │   │   └── dependencies/ # 沙箱依赖
│   └── middleware.env.example # 中间件环境变量示例
├── storage/                  # 数据存储目录
│   ├── logs/                 # 日志目录
│   ├── files/                # 上传文件目录
│   ├── plugin_daemon/        # 插件数据目录
│   ├── redis/                # Redis数据目录
│   │   └── data/             # Redis持久化数据
│   └── pgvector/             # PGVector数据目录
│   │   └── data/             # PGVector持久化数据
└── README.md                 # 说明文档
```

## 常见问题

### 1. 如何修改默认端口？

在 `.env` 文件中修改相应的端口配置，例如：

```
WEB_PORT=3001
API_PORT=5002
```

### 2. 如何开启HTTPS？

1. 准备好SSL证书和密钥，放在 `conf/nginx/ssl` 目录下
2. 修改 `conf/nginx/conf.d/default.conf`，添加HTTPS配置：

```nginx
server {
    listen 443 ssl;
    server_name localhost;

    ssl_certificate /etc/nginx/ssl/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;

    # 其他配置与HTTP配置相同
    # ...
}
```

### 3. 数据存储在哪里？

所有数据都存储在 `storage` 目录下，包括上传的文件、日志等。您可以通过 `.env` 中的 `DIFY_ROOT_PATH` 变量修改存储路径。

### 4. 如何升级Dify版本？

1. 首先备份数据：

```bash
cp -r storage storage_backup
```

2. 更新镜像版本：

```bash
# 修改docker-compose.yml中的镜像版本
# 例如：langgenius/dify-api:1.3.1 -> langgenius/dify-api:1.4.0

# 重新拉取镜像并重启服务
docker-compose pull
docker-compose up -d
```

### 5. 如何添加自定义域名？

修改 `conf/nginx/conf.d/default.conf` 文件，将 `server_name localhost;` 改为 `server_name yourdomain.com;`，然后重启Nginx：

```bash
docker-compose restart dify-nginx
```

### 6. Redis连接失败怎么办？

如果Redis连接失败，可能的原因和解决方案：

1. 检查Redis容器是否正常运行：

```bash
docker-compose ps dify-redis
```

2. 查看Redis日志：

```bash
docker-compose logs dify-redis
```

3. 检查Redis密码配置是否一致：
确保`.env`文件中的`REDIS_PASSWORD`与API服务环境变量中的一致。

4. 直接连接Redis进行测试：

```bash
docker exec -it dify-redis redis-cli
auth your-redis-password
ping
```

如果返回`PONG`，则Redis服务正常。

## 升级指南

### 从1.2.x升级到1.3.x

1. 备份数据：

```bash
cp -r storage storage_backup
```

2. 更新配置文件和Docker Compose文件：

```bash
git pull
```

3. 更新环境变量，添加新的必要配置：

```bash
vim .env
# 添加新的环境变量...
```

4. 更新并重启服务：

```bash
docker-compose down
docker-compose pull
docker-compose up -d
```

## 贡献指南

欢迎贡献代码或提交问题报告！请遵循以下步骤：

1. Fork本仓库
2. 创建您的功能分支 (`git checkout -b feature/amazing-feature`)
3. 提交您的更改 (`git commit -m 'Add some amazing feature'`)
4. 推送到分支 (`git push origin feature/amazing-feature`)
5. 创建一个Pull Request

## 许可证

本项目遵循MIT许可证。详情请参见LICENSE文件。
