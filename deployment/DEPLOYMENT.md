# Chatmill 部署与配置指南

---

## 目录

1. [Docker Compose 部署总览](#docker-compose-部署总览)

   1.1 [核心服务配置（捕捉端、agent、发布端）](#核心服务配置捕捉端agent发布端)

   1.2 [数据库服务配置（MongoDB、向量数据库）](#数据库服务配置mongodb向量数据库)
2. [Discord Application 配置指南](#discord-application-配置指南)

---

# 1. Docker Compose 部署总览

## 1.1 核心服务配置（捕捉端、agent、发布端）

捕捉端可包括 Discord、Slack、Telegram、Web 表单等，均采用类似配置方式。以下以 discord-capture 为示例，其他捕捉端（如 slack-capture、telegram-capture、webform-capture）可参照配置。

```yaml
discord-capture:  # 其他捕捉端如 slack-capture、telegram-capture 亦同理
  image: chatmill/discord-capture:latest
  environment:
    DISCORD_TOKEN: your_discord_token         # Discord Bot Token
    DISCORD_CLIENT_ID: your_discord_client_id # Discord Bot Client ID
    GUILD_IDS: 1234567890,9876543210         # 监听的服务器ID，逗号分隔
    AGENT_ROUTES: >
      {
        "miss_spec": "http://miss-spec-agent:8000/webhook",
        "mr_pulse": "http://mr-pulse-agent:8000/webhook"
      }
    PUBLISH_ROUTES: >
      {
        "github": "http://github-publish:8000/webhook",
        "notion": "http://notion-publish:8000/webhook"
      }
    MAX_MESSAGE_FETCH_COUNT: 25               # 单次消息抓取上限，防止过量拉取
    # JWT 配置
    JWT_SECRET: your_super_secret_key         # 微服务通信 JWT 密钥，所有服务需一致
    # 限流配置
    RATE_LIMIT_PER_MINUTE: 60                 # 每分钟最大请求数
    RATE_LIMIT_BURST: 10                      # 突发流量最大请求数
    LOG_LEVEL: INFO                          # 日志级别
    MONGODB_URI: mongodb://mongodb:27017     # MongoDB 连接地址
    MONGODB_DB_NAME: chatmill                # MongoDB 数据库名
  depends_on:
    - mongodb
```

- **AGENT_ROUTES**、**PUBLISH_ROUTES**、**MAX_MESSAGE_FETCH_COUNT** 等参数适用于所有捕捉端。
- 其他捕捉端（如 slack-capture、telegram-capture、webform-capture 等）请参照上述配置方式，按需调整镜像名和平台专属参数。

---

所有 agent 能力（如 miss-spec-agent、mr-pulse-agent、echo-agent 等）均采用类似配置方式，以下以 miss-spec-agent、mr-pulse-agent 为示例，其他 agent 可参照配置。

```yaml
miss-spec-agent:
  image: chatmill/miss-spec-agent:latest
  environment:
    PUBLISH_ROUTES: >
      {
        "github": "http://github-publish:8000/webhook",
        "notion": "http://notion-publish:8000/webhook"
      }
    CAPTURE_ROUTES: >
      {
        "discord": "http://discord-capture:8000/webhook",
        "slack": "http://slack-capture:8000/webhook"
      }
    API_KEY: your_llm_api_key                # LLM/AI 服务密钥
    MODEL: deepseek-llama3-8b-instruct       # LLM/AI 服务模型名
    VECTOR_DB_URI: http://milvus:19530       # 可选，向量数据库服务地址
    VECTOR_DB_TYPE: milvus                   # 可选，向量数据库类型（如 milvus、qdrant、pinecone、chroma 等）
    # JWT 配置
    JWT_SECRET: your_super_secret_key         # 微服务通信 JWT 密钥，所有服务需一致
    # 限流配置
    RATE_LIMIT_PER_MINUTE: 60                 # 每分钟最大请求数
    RATE_LIMIT_BURST: 10                      # 突发流量最大请求数
    LOG_LEVEL: INFO
    MONGODB_URI: mongodb://mongodb:27017
    MONGODB_DB_NAME: chatmill
  depends_on:
    - mongodb
    # - milvus  # 如启用向量数据库

mr-pulse-agent:
  image: chatmill/mr-pulse-agent:latest
  environment:
    PUBLISH_ROUTES: >
      {
        "notion": "http://notion-publish:8000/webhook"
      }
    CAPTURE_ROUTES: >
      {
        "discord": "http://discord-capture:8000/webhook"
      }
    API_KEY: your_llm_api_key
    MODEL: gpt-4
    VECTOR_DB_URI: http://milvus:19530
    VECTOR_DB_TYPE: milvus
    LOG_LEVEL: INFO
    MONGODB_URI: mongodb://mongodb:27017
    MONGODB_DB_NAME: chatmill
  depends_on:
    - mongodb
    # - milvus
```

- 所有 agent 服务均可参照上述配置，独立管理各自参数。

---

所有发布端（如 github-publish、notion-publish、report-publish 等）均采用类似配置方式。以下以 github-publish 为示例，其他发布端可参照配置。

```yaml
github-publish:  # 其他发布端如 notion-publish、report-publish 亦同理
  image: chatmill/github-publish:latest
  environment:
    GITHUB_TOKEN: your_github_token          # GitHub 访问 Token
    # JWT 配置
    JWT_SECRET: your_super_secret_key         # 微服务通信 JWT 密钥，所有服务需一致
    # 限流配置
    RATE_LIMIT_PER_MINUTE: 60                 # 每分钟最大请求数
    RATE_LIMIT_BURST: 10                      # 突发流量最大请求数
    LOG_LEVEL: INFO                          # 日志级别
    MONGODB_URI: mongodb://mongodb:27017     # MongoDB 连接地址
    MONGODB_DB_NAME: chatmill                # MongoDB 数据库名
  depends_on:
    - mongodb
```

- 其他发布端（如 notion-publish、report-publish 等）请参照上述配置方式，按需调整镜像名和平台专属参数。

---

## 1.2 数据库服务配置（MongoDB、向量数据库）

### MongoDB 服务块

```yaml
mongodb:
  image: mongo:6
  restart: always
  environment:
    MONGO_INITDB_DATABASE: chatmill  # 初始化数据库名
  ports:
    - 27017:27017
  volumes:
    - mongo_data:/data/db
```

### 可选：向量数据库服务块（以 Milvus 为例）

```yaml
milvus:
  image: milvusdb/milvus:v2.4.0
  container_name: milvus
  ports:
    - 19530:19530
    - 9091:9091
  environment:
    ETCD_ENDPOINTS: etcd:2379
    MINIO_ADDRESS: minio:9000
  depends_on:
    - etcd
    - minio

etcd:
  image: quay.io/coreos/etcd:v3.5.5
  container_name: etcd
  environment:
    ETCD_AUTO_COMPACTION_MODE: revision
    ETCD_AUTO_COMPACTION_RETENTION: '1000'
    ETCD_QUOTA_BACKEND_BYTES: '4294967296'
  ports:
    - 2379:2379

minio:
  image: minio/minio:RELEASE.2023-03-20T20-16-18Z
  container_name: minio
  environment:
    MINIO_ACCESS_KEY: minioadmin
    MINIO_SECRET_KEY: minioadmin
  command: server /data
  ports:
    - 9000:9000
```

> 向量数据库为可选项，未启用时可省略相关配置和服务。支持 milvus、qdrant、pinecone、chroma 等主流向量数据库。

---

# 2. Discord Application 配置指南

本节介绍如何在 Discord Developer Portal 创建和配置 Bot 应用，包括获取 Token、Client ID、配置权限、邀请 Bot 等。

## 2.1 创建 Discord 应用与 Bot

1. 访问 [Discord Developer Portal](https://discord.com/developers/applications)。
2. 点击 "New Application"，输入名称（如 Chatmill Bot），点击"Create"。
3. 在左侧菜单选择 "Bot"，点击 "Add Bot"，确认创建。

## 2.2 获取 Bot Token

1. 在 Bot 页面点击 "Reset Token" 或 "Copy"，获取 `DISCORD_TOKEN`。
2. **安全提示**：请妥善保管 Token，切勿泄露。

## 2.3 配置 Bot 权限

1. 在 "OAuth2" → "URL Generator" 页面，勾选 `bot` 和 `applications.commands`。
2. 在 "Bot Permissions" 区域，建议至少勾选：
   - `Send Messages`
   - `Read Message History`
   - `Manage Messages`
   - `Add Reactions`
   - `Use Slash Commands`
   - `Embed Links`
   - `View Channels`
   - `Manage Webhooks`
3. 复制生成的邀请链接。

> 如需通过 webhook 方式向频道发消息，Bot 必须具备 `Manage Webhooks` 权限，否则无法创建和管理 webhook。

## 2.4 邀请 Bot 加入服务器

1. 用管理员账号访问上一步生成的邀请链接。
2. 选择目标服务器，点击"授权"。
3. 在 Discord 服务器成员列表中应能看到 Bot。

## 2.5 其他建议与补充

- **Slash Command 支持**：必须在 Bot 权限中勾选 `applications.commands`，并在代码中注册所有指令，否则用户无法通过 `/missspec` 等命令与 Bot 交互。
- **多服务器部署**：如需让服务同时服务多个 Discord 服务器（Guild），应在配置中指定 `GUILD_IDS`（逗号分隔），并在代码中校验事件来源。
- **SERVER MEMBERS INTENT**：如需让 Bot 判断某成员是否在服务器/频道，需在 Bot 设置页开启 `SERVER MEMBERS INTENT`。

---

- **JWT_SECRET**：微服务通信 JWT 密钥，所有服务需一致。签名算法和有效期为平台统一约定（如 HS256/3600秒），无需用户配置。
- **RATE_LIMIT_PER_MINUTE**：每分钟最大请求数，防止接口被刷爆。
- **RATE_LIMIT_BURST**：突发流量最大请求数，防止短时高并发。

---

> organization_id/project_id/agent 字段为平台无关抽象，映射关系如下：
> - Discord: organization_id = server_id (GUILD_ID), project_id = channel_id
> - Slack: organization_id = workspace_id, project_id = channel_id
> - 其他平台以适配器文档为准

---

agent 字段用于标识本 session 归属的 agent 能力插件（如 missspec、nudge、echo），支持多 agent 扩展和策略分发。

---
