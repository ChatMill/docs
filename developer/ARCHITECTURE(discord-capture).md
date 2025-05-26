# discord-capture 架构分层与开发约定

## 1. 组件图与分层依赖

本模块采用 DDD 分层架构，所有外部通信均通过 infrastructure 层适配，支持未来多平台扩展。

> 所有 HTTP API、Webhook、平台适配器与 Agent 能力插件之间的通信均建议或强制使用 JWT（JSON Web Token）认证，保障接口安全与身份可信。详见 deployment/PLATFORM_ADAPTERS.md。

[组件分层与依赖图](../uml/component.md)

> 说明：分层结构严格遵循 DDD，所有协议与接口细节见 INTERACTION_GUIDE.md。infrastructure 层负责与 Discord 平台、HTTP 路由、下游/中游服务的所有实际交互，Publisher 可扩展。

## 2. 领域与应用完整类图

[领域与应用完整类图](../uml/domain_class (discord-capture).md)

## 3. 目录结构（DDD 分层，文件级细化示例）

```plaintext
discord-capture/
├── domain/                        # 领域层：核心业务对象、聚合、服务、事件、仓储接口
│   ├── entities/                  # 领域实体与聚合根
│   │   ├── session.py             # Session 聚合根，管理会话全生命周期
│   │   ├── message.py             # Message 实体，封装单条消息内容
│   │   └── structured_payload.py  # StructuredPayload 抽象及 Task/Checklist/ContentDraft 实现
│   ├── events/                    # 领域事件（Event 抽象及子类）
│   │   ├── base_event.py          # Event 抽象基类，统一事件通用字段
│   │   ├── initiate.py            # Initiate 事件，会话启动事件
│   │   ├── supplement_request.py  # SupplementRequest 事件，请求补充信息
│   │   ├── supplement_response.py # SupplementResponse 事件，补充信息响应
│   │   ├── confirmation.py        # Confirmation 事件，确认操作
│   │   └── publish_result.py      # PublishResult 事件，发布结果反馈
│   ├── services/                  # 领域服务
│   │   ├── session_service.py     # SessionService，会话业务规则与状态流转
│   │   ├── cml_event_relay_service.py # CmlEventRelayService，CML事件流转与分发
│   │   └── message_fetcher_service.py # MessageFetcherService，消息抓取与校验
│   ├── repositories/              # 仓储接口
│   │   ├── session_repository.py  # SessionRepository，会话持久化接口
│   │   ├── structured_payload_repository.py # StructuredPayloadRepository，结构化负载持久化接口
│   │   └── message_repository.py  # MessageRepository，消息持久化接口
│   └── value_objects/             # 值对象
│       └── message_range.py       # MessageRange，消息区间值对象
│
├── application/                   # 应用层：用例/流程编排、DTO、应用服务
│   ├── services/                  # 应用服务
│   │   └── command_service.py     # CommandService，用例编排与流程控制
│   ├── handlers/                  # 命令/事件处理器
│   │   └── session_handler.py     # SessionHandler，处理会话相关命令/事件
│   ├── dtos/                      # 数据传输对象
│   │   └── session_dto.py         # SessionDTO，会话数据传输结构
│   └── strategies/                # 策略分发/注册机制
│       └── payload_strategy.py    # PayloadStrategy，基于 agent 字段的策略分发
│
├── infrastructure/                # 基础设施层：持久化、平台适配、外部集成
│   ├── repositories/              # 仓储实现
│   │   ├── mongo_session_repository.py           # MongoDB SessionRepository 实现
│   │   ├── mongo_structured_payload_repository.py # MongoDB StructuredPayloadRepository 实现
│   │   └── mongo_message_repository.py           # MongoDB MessageRepository 实现
│   ├── platform/                  # 平台适配器
│   │   ├── discord_client.py      # Discord API 封装
│   │   └── discord_webhook.py     # Discord Webhook 适配
│   ├── webhook/                   # Webhook 适配
│   │   └── webhook_handler.py     # WebhookHandler，处理外部 webhook 回调
│   ├── persistence/               # 数据库连接、模型
│   │   └── mongodb.py             # MongoDB 连接与模型定义
│   └── config/                    # 配置与环境变量
│       └── settings.py            # Settings，环境变量与配置加载
│
├── interfaces/                    # 接口层：API、命令入口、webhook
│   ├── api/                       # FastAPI 路由
│   │   └── routes.py              # API 路由定义
│   ├── commands/                  # 命令解析
│   │   └── discord_commands.py    # Discord 命令解析与适配
│   ├── webhook/                   # Webhook 入口
│   │   └── discord_webhook_handler.py # Discord Webhook 入口处理
│   └── schemas/                   # API/消息体 schema
│       └── session_schema.py      # SessionSchema，API 数据结构定义
│
├── shared/                        # 跨层通用工具、CML协议、异常、日志
│   ├── cml/                       # CML 协议定义与解析
│   │   └── cml_protocol.py        # CML 协议结构、序列化/反序列化
│   ├── exceptions.py              # 通用异常定义
│   └── logger.py                  # 日志工具
│
├── main.py                        # 应用入口，依赖注入与启动
└── README.md                      # 项目说明文档
```

### 结构说明与类图映射
- `domain/entities/structured_payload.py`：定义 StructuredPayload 抽象及 Task、Checklist、ContentDraft 实现
- `domain/events/`：所有 Event 及其子类（Initiate、SupplementRequest、SupplementResponse、Confirmation、PublishResult）
- `domain/services/`：SessionService、CmlEventRelayService、MessageFetcherService
- `application/services/command_service.py`：CommandService，编排用例
- `application/strategies/payload_strategy.py`：基于 agent 字段的策略分发机制
- `infrastructure/repositories/`：所有仓储实现
- `interfaces/api/routes.py`：APIRouter 路由入口
- 其余结构与类图、DDD 分层完全对应

### 多平台扩展建议
- 未来如需支持 Slack/Telegram，只需在 `infrastructure/platform/` 和 `interfaces/commands/` 下新增对应适配器和命令解析器，domain/application 层无需变动。
- CML 协议、领域事件、Session 聚合等均保持平台无关。

### 设计原则
- 严格遵循 DDD 分层，保持各层解耦，便于单元测试和平台扩展。
- 领域层不依赖任何外部平台和基础设施，所有平台相关逻辑均在 infrastructure/ 下实现。
- 领域事件、CML 协议、Session 聚合等均为平台无关抽象。

---
如需进一步细化某一层的典型代码结构或类/方法注释示例，请参考本目录结构并结合实际业务需求补充。

## 校验责任分层说明：
- **interfaces**：输入解析、基础格式/平台校验（如命令格式、ID、@user、GUILD_IDS、webhook来源等）
- **application**：业务流程、权限、上下文复杂校验（如区间上限、参与人/责任人、session状态等）
- **domain**：领域不变式、数据一致性校验（如session参与人、事件树、发布后不可补充等）
