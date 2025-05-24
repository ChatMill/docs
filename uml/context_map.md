## 领域边界与上下文地图

- 捕获端（Capture）：负责 IM 平台（如 Discord）消息捕捉、指令解析、上下文管理、CML 事件生成。
- agent端（Agent）：负责语义抽象、智能补充、业务流转、补充请求等。
- 发布端（Publish）：负责结构化需求的发布、同步、回执（如 GitHub、Jira、Notion）。
- 各端通过 CML 协议（JSON/YAML）事件流解耦，所有跨端通信均为结构化事件。
- 捕获端领域层不依赖 agent端/发布端的具体实现，仅依赖协议。

> organization_id/project_id 字段为平台无关抽象，映射关系如下：
> - Discord: organization_id = server_id, project_id = channel_id
> - Slack: organization_id = workspace_id, project_id = channel_id
> - 其他平台以适配器文档为准

```mermaid
flowchart LR
  subgraph 捕获端
    C1[消息捕捉/指令解析]
    C2[Session 聚合]
    C3[CML 事件生成]
  end
  subgraph agent端
    A1[语义抽象/补充请求]
    A2[业务流转/智能处理]
  end
  subgraph 发布端
    P1[结构化发布]
    P2[发布回执]
  end
  C3 -- CML事件 --> A1
  A1 -- CML事件 --> C3
  C3 -- CML事件 --> P1
  P2 -- CML事件 --> C3
``` 