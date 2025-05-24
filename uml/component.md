## 组件图

本模块采用 DDD 分层架构，所有外部通信均通过 infrastructure 层适配，支持未来多平台扩展。

```mermaid
flowchart TD
  subgraph 捕获端["捕获端（IM平台）"]
    direction TB
    D_Domain["domain\n(models, events, repositories, services)"]
    D_Application["application\n(commands, services, use_cases)"]
    D_Infrastructure["infrastructure\n(bot.py, http, discord_sdk, persistence, config.py)"]
    D_Interfaces["interfaces\n(commands.py, dto, api)"]
  end

  subgraph 校验节点["Validation Points"]
    V_Interfaces["interfaces: 格式/平台/基础校验"]
    V_Application["application: 业务/权限/流程校验"]
    V_Domain["domain: 不变式/一致性校验"]
  end

  subgraph Agent["Agent能力插件"]
    MissSpec["Miss Spec"]
    Nudge["Nudge"]
    Echo["Echo"]
  end

  subgraph 发布端["发布端（结构化输出平台）"]
    G_Publisher["GitHub Publisher\n(GITHUB_PUBLISHER_BASE_URL)"]
    N_Publisher["Notion Publisher\n(NOTION_PUBLISHER_BASE_URL)"]
  end

  %% 层内依赖
  D_Interfaces --> D_Application
  D_Application --> D_Domain
  D_Infrastructure --> D_Domain
  D_Application --> D_Infrastructure
  D_Interfaces --> D_Infrastructure

  %% 校验依赖
  D_Interfaces --> V_Interfaces
  D_Application --> V_Application
  D_Domain --> V_Domain

  %% 外部接口
  D_Infrastructure -- "HTTP/Webhook\n(supplement, publish_result)" --> MissSpec
  D_Infrastructure -- "HTTP/Webhook\n(supplement, publish_result)" --> Nudge
  D_Infrastructure -- "HTTP/Webhook\n(supplement, publish_result)" --> Echo
  D_Infrastructure -- "HTTP/Webhook\n(publish_result 接收)" --> G_Publisher
  D_Infrastructure -- "HTTP/Webhook\n(publish_result 接收)" --> N_Publisher

  %% IM平台
  IM平台["IM平台"]
  IM平台 -- "消息/指令事件" --> D_Infrastructure

  %% 说明
  classDef ext fill:#e6f7ff,stroke:#333,stroke-width:2px,color:#222,font-weight:bold;
  class IM平台 ext;
```

> 说明：
> 
> - 本图展示了 Chatmill 的三大核心区块：捕获端（IM平台）、Agent（中间层能力插件）、发布端（结构化输出平台），以及它们之间的接口与依赖关系。
> - 捕获端负责从 IM 平台（如 Discord、Slack 等）接收消息和指令，通过接口适配层流入系统。
> - Agent 区块代表所有可插拔的中间层智能处理能力（如 Miss Spec、Nudge 等），通过标准 API 与捕获端通信。
> - 发布端区块代表所有结构化成果的落地平台（如 GitHub、Notion），支持多平台并行扩展。
> - 校验节点分别对应输入格式、业务流程、数据一致性等不同层级的校验职责，确保数据流转安全可靠。
> - 所有外部通信均通过 HTTP/Webhook 适配，便于未来扩展更多 IM 平台、Agent 能力和发布端。
> - 组件间依赖箭头标注了各区块之间、与校验节点、与外部服务的调用关系。
