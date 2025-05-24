# Chatmill Language (CML) 协议与事件类型

## 认证与安全

- 所有 CML 协议事件流（如 initiate、supplement_request、publish 等）在 HTTP/Webhook 传输时，建议通过 Authorization: Bearer <JWT> 头部传递 JWT Token。
- JWT 用于身份校验、接口防护、事件溯源，平台适配器与 Agent 能力插件均应实现 JWT 校验逻辑。
- JWT 配置与密钥管理详见 deployment/PLATFORM_ADAPTERS.md。

## CML 事件类型与字段定义

### 1 initiate
- 说明：用于发起需求草案，可直接指定 participants 字段。
- 用法：
  - `/agent initiate 1..13 @Bob @Carol`
- 目的：将一段讨论正式转化为结构化需求，启动需求捕捉流程。
- 游向：捕获端（IM） → agent能力插件
- 认证：建议通过 HTTP Header 传递 JWT Token，保障事件流安全。
- 字段：
  - type: "initiate"
  - session_id
  - message_id: 会话内对话唯一标识
  - source: { platform, organization_id, project_id, message_ids, participants }
  - operator_id: 操作人用户ID
  - agent: 归属 agent 能力插件（如 missspec、nudge、echo），用于多 agent 策略分发
  - payload: 结构化负载对象（类型为 StructuredPayload，具体结构由 agent 能力插件定义。下方示例为 Miss Spec agent 的 Task 实现，仅为示例）
  - history: 事件历史（字符串数组，表示历史事件的 message_id，如 ["msg-001", "msg-002"]）

| 字段名         | 类型       | 必填  | 格式/约束            | 说明                               |
|-------------|----------|-----|------------------|----------------------------------|
| type        | string   | 是   | 固定值 "initiate"   | 事件类型                             |
| session_id  | string   | 是   | 唯一ID             | 会话唯一标识                           |
| message_id  | string   | 是   | 唯一ID             | 本事件唯一标识                          |
| source      | object   | 是   | 见下表              | 来源平台及消息上下文                       |
| operator_id | string   | 是   | 用户ID             | 操作人 ID                           |
| agent       | string   | 是   | agent 能力插件标识   | 归属 agent 能力插件（如 missspec、nudge、echo），用于多 agent 策略分发 |
| payload     | object   | 是   | 结构由 agent 能力插件定义 | 结构化负载对象（如 Miss Spec 的 Task，仅为示例） |
| history     | string[] | 是   | message_id 数组    | 历史事件ID溯源                         |

**source 字段说明：**

| 字段名          | 类型       | 必填  | 格式/约束       | 说明       |
|--------------|----------|-----|-------------|----------|
| platform     | string   | 是   | "discord" 等 | 来源平台     |
| organization_id | string   | 是   | 唯一ID        | Discord 服务器ID |
| project_id   | string   | 是   | 唯一ID        | 频道ID     |
| message_ids  | int[]    | 是   | 消息ID数组/区间   | 相关消息ID   |
| participants | string[] | 是   | 用户ID数组      | 当前与会人列表  |

**organization_id/project_id 字段为平台无关抽象，映射关系如下：**
- Discord: organization_id = server_id, project_id = channel_id
- Slack: organization_id = workspace_id, project_id = channel_id
- 其他平台以适配器文档为准

**payload 字段说明：**

| 字段名        | 类型           | 必填  | 说明                                         |
|------------|--------------|-----|--------------------------------------------|
| message_ids| int[]/str[]  | 是   | 溯源消息ID，唯一强制字段                          |
| external_id| string/null  | 否   | 外部平台唯一ID（如 issue id、card id），有外部平台映射时填写 |
| ...        | any          | 否   | 其他字段（如 title、description 等）由 agent 能力插件自定义，不在协议强制要求 |

- 类型为 object，结构由 agent 能力插件定义。
- 下方为 Miss Spec agent 的 Task 实现示例，仅为 agent 能力插件实现示例，其他 agent 可有不同结构。
- 示例中的 parent_task、sub_tasks 字段仅为 Miss Spec agent 的实现示例，非协议强制要求。

- 示例（Miss Spec agent 的 Task 实现，仅为示例）：
```json
{
  "type": "initiate",
  "session_id": "sess-001",
  "message_id": "msg-001",
  "source": {
    "platform": "discord",
    "organization_id": "654321",
    "project_id": "123456",
    "message_ids": [3001, ..., 3010],
    "participants": ["u1", "u2", "u3"]
  },
  "operator_id": "u1",
  "agent": "missspec",
  "payload": {
    "missspec_id": "msk-001",
    "external_id": null,
    "title": "快递柜扫码取件需求",
    "description": "用户扫码后自动通知柜员，支持多家快递",
    "message_ids": [3001, ..., 3010],
    "start_time": "2024-06-01T12:00:00Z",
    "end_time": "2024-06-10T18:00:00Z",
    "storypoints": 8,
    "assignees": ["u1", "u2"],
    "priority": "high",
    "parent_task": null,
    "sub_tasks": [
      {
        "missspec_id": "msk-002",
        "external_id": null,
        "title": "扫码流程设计",
        "description": "支持顺丰、京东、菜鸟等快递扫码取件",
        "message_ids": [3001, ..., 3005],
        "start_time": "2024-06-01T12:00:00Z",
        "end_time": "2024-06-05T18:00:00Z",
        "storypoints": 3,
        "assignees": ["u1"],
        "priority": "medium",
        "parent_task": "msk-001",
        "sub_tasks": []
      }
    ],
    "history": ["msg-001"]
  }
}
```

### 2 supplement_request
- 说明：用于请求补充信息。
- 目的：agent能力插件发现需求信息不全，主动向捕获端请求补充。
- 认证：建议通过 HTTP Header 传递 JWT Token，保障事件流安全。
- 游向：agent能力插件 → 捕获端（IM）
- 字段：
  - type: "supplement_request"
  - session_id
  - message_id: 会话内对话唯一标识
  - source: { platform, organization_id, project_id, participants }
  - operator_id: 操作人用户ID
  - question: 需要补充的问题
  - required_fields: 需要补充的字段
  - payload: 结构化负载对象（结构同 initiate）
  - history: 事件历史（字符串数组，表示历史事件的 message_id，如 ["msg-001", "msg-002"]）

| 字段名             | 类型       | 必填  | 格式/约束                    | 说明               |
|-----------------|----------|-----|-----------------------|------------------|
| type            | string   | 是   | 固定值 "supplement_request" | 事件类型             |
| session_id      | string   | 是   | 唯一ID                     | 会话唯一标识           |
| message_id      | string   | 是   | 唯一ID                     | 本事件唯一标识          |
| source          | object   | 是   | 见下表                      | 来源平台及消息上下文       |
| operator_id     | string   | 是   | 用户ID                     | 操作人用户ID |
| question        | string   | 是   | 不超过500字符                 | 需要补充的问题          |
| required_fields | string[] | 是   | 字段名数组                    | 需要补充的字段          |
| payload         | object   | 是   | 见1 payload表               | 结构化负载对象             |
| history         | string[] | 是   | message_id 数组            | 历史事件ID溯源         |

**source 字段说明：**

| 字段名          | 类型       | 必填  | 格式/约束       | 说明       |
|--------------|----------|-----|-------------|----------|
| platform     | string   | 是   | "discord" 等 | 来源平台     |
| organization_id | string   | 是   | 唯一ID        | Discord 服务器ID |
| project_id   | string   | 是   | 唯一ID        | 频道ID     |
| participants | string[] | 是   | 用户ID数组      | 当前与会人列表  |

### 3 supplement_response
- 说明：用于提交补充内容。
- 认证：建议通过 HTTP Header 传递 JWT Token，保障事件流安全。
- 用法：
  - `/agent supplement sess-001 14..20`
- 目的：捕获端用户对补充请求进行响应，补全需求信息。
- 认证：建议通过 HTTP Header 传递 JWT Token，保障事件流安全。
- 游向：捕获端（IM） → agent能力插件
- 字段：
  - type: "supplement_response"
  - session_id
  - message_id: 会话内对话唯一标识
  - source: { platform, organization_id, project_id, participants }
  - operator_id: 操作人用户ID
  - supplement_messages: 本次补充消息ID数组（如 [3011, ..., 3030]，支持区间）
  - payload: 结构化负载对象（结构同 initiate）
  - history: 事件历史（字符串数组，表示历史事件的 message_id，如 ["msg-001", "msg-002"]）

| 字段名                 | 类型       | 必填  | 格式/约束                     | 说明               |
|---------------------|----------|-----|---------------------------|------------------|
| type                | string   | 是   | 固定值 "supplement_response" | 事件类型             |
| session_id          | string   | 是   | 唯一ID                      | 会话唯一标识           |
| message_id          | string   | 是   | 唯一ID                      | 本事件唯一标识          |
| source              | object   | 是   | 见下表                       | 来源平台及消息上下文       |
| operator_id         | string   | 是   | 用户ID                      | 操作人用户ID |
| supplement_messages | int[]    | 是   | 消息ID数组/区间（区间最大25条）   | 本次补充消息ID         |
| payload             | object   | 是   | 见1 payload表                | 结构化负载对象             |
| history             | string[] | 是   | message_id 数组             | 历史事件ID溯源         |

**source 字段说明：**

| 字段名          | 类型       | 必填  | 格式/约束       | 说明       |
|--------------|----------|-----|-------------|----------|
| platform     | string   | 是   | "discord" 等 | 来源平台     |
| organization_id | string   | 是   | 唯一ID        | Discord 服务器ID |
| project_id   | string   | 是   | 唯一ID        | 频道ID     |
| participants | string[] | 是   | 用户ID数组      | 当前与会人列表  |

### 4 publish
- 认证：建议通过 HTTP Header 传递 JWT Token，保障事件流安全。
- 说明：用于最终发布结构化负载。
- 用法：
  - `/agent publish sess-001 --to github`
- 目的：与会人对结构化负载内容达成一致，确认可进入发布端发布。
- 认证：建议通过 HTTP Header 传递 JWT Token，保障事件流安全。
- 游向：捕获端（IM） → agent能力插件
- 字段：
  - type: "publish"
  - session_id
  - message_id: 会话内对话唯一标识
  - source: { platform, organization_id, project_id, participants }
  - operator_id: 操作人用户ID
  - platform: 目标发布平台（如 github、jira、notion，必填）
  - payload: 结构化负载对象（结构同 initiate）
  - history: 事件历史（字符串数组，表示历史事件的 message_id，如 ["msg-001", "msg-002"]）

| 字段名         | 类型       | 必填  | 格式/约束              | 说明               |
|-------------|----------|-----|--------------------|------------------|
| type        | string   | 是   | 固定值 "publish" | 事件类型             |
| session_id  | string   | 是   | 唯一ID               | 会话唯一标识           |
| message_id  | string   | 是   | 唯一ID               | 本事件唯一标识          |
| source      | object   | 是   | 见下表                | 来源平台及消息上下文       |
| operator_id | string   | 是   | 用户ID               | 操作人用户ID |
| platform    | string   | 是   | "github"/"jira"/"notion"等 | 目标发布平台           |
| payload     | object   | 是   | 见1 payload表         | 结构化负载对象             |
| history     | string[] | 是   | message_id 数组      | 历史事件ID溯源         |

### 5 publish_result
- 认证：建议通过 HTTP Header 传递 JWT Token，保障事件流安全。
- 说明：用于发布端发布结果回传，result 字段带下游 id 及 message_ids，便于溯源。
- 目的：发布端（如 GitHub）将发布结果同步回捕获端，便于溯源和回显。
- 认证：建议通过 HTTP Header 传递 JWT Token，保障事件流安全。
- 游向：发布端 → 捕获端（IM）
- 字段：
  - type: "publish_result"
  - session_id
  - message_id: 会话内对话唯一标识
  - source: { platform, organization_id, project_id, participants }
  - platform: 目标发布平台（如 github、jira、notion，必填）
  - payload: 结构化负载对象（结构同 initiate）
  - result: 发布结果对象（仅含发布状态、平台、url、message等）
  - history: 事件历史（字符串数组，表示历史事件的 message_id，如 ["msg-001", "msg-002"]）

| 字段名        | 类型       | 必填  | 格式/约束                | 说明         |
|------------|----------|-----|----------------------|------------|
| type       | string   | 是   | 固定值 "publish_result" | 事件类型       |
| session_id | string   | 是   | 唯一ID                 | 会话唯一标识     |
| message_id | string   | 是   | 唯一ID                 | 本事件唯一标识    |
| source     | object   | 是   | 见下表                  | 来源平台及消息上下文 |
| platform   | string   | 是   | "github"/"jira"/"notion"等 | 目标发布平台     |
| payload    | object   | 是   | 见1 payload表           | 结构化负载对象       |
| result     | object   | 是   | 见下表                  | 发布结果对象     |
| history    | string[] | 是   | message_id 数组        | 历史事件ID溯源   |

**result 字段说明：**

| 字段名   | 类型     | 必填 | 说明                 |
|----------|----------|------|----------------------|
| status   | string   | 是   | 发布状态（如 success/failed）|
| platform | string   | 是   | 发布平台（如 github、jira）  |
| url      | string   | 否   | 发布对象的链接（如 issue/pr）|
| message  | string   | 否   | 发布结果说明/错误信息        |
| id       | string   | 否   | 下游平台对象ID（如 issue id）|

### 6 add_participant
- 说明：用于将用户加入当前 session 的与会人列表。
- 用法：
  - `/agent add_participant @Carol @Dave`
- 目的：动态增加协作的参与人。
- 游向：捕获端（IM）本地存储
- 字段：
  - type: "add_participant"
  - session_id
  - message_id: 会话内对话唯一标识
  - source: { platform, organization_id, project_id, participants }
  - operator_id: 操作人用户ID
  - participants: 变更后完整与会人列表
  - target_participants: 本次要加入的目标用户列表（必选，支持单个或多个）
  - history: 事件历史（字符串数组，表示历史事件的 message_id，如 ["msg-001", "msg-002"]）

### 7 remove_participant
- 说明：用于将用户移出当前 session 的与会人列表。
- 用法：
  - `/agent remove_participant @Carol`
- 目的：动态移除协作的参与人。
- 游向：捕获端（IM）本地存储
- 字段：
  - type: "remove_participant"
  - session_id
  - message_id: 会话内对话唯一标识
  - source: { platform, organization_id, project_id, participants }
  - operator_id: 操作人用户ID
  - participants: 变更后完整与会人列表
  - target_participants: 本次要移除的目标用户列表（必选，支持单个或多个）
  - history: 事件历史（字符串数组，表示历史事件的 message_id，如 ["msg-001", "msg-002"]）

### 8 modify
- 说明：用于手动修改结构化负载树结构或内容，支持全量替换和路径精确修改两种模式。
- 用法：
  - `/agent modify {完整payload的json}`
  - `/agent modify payload.title 新的标题`
  - `/agent modify payload.sub_tasks[0].title 新的子结构标题`
- 目的：允许用户通过命令手动修正结构化负载树的任意字段或整体内容。
- 游向：捕获端（IM） → 本地存储/agent能力插件（如需同步）
- 字段：
  - type: "modify"
  - session_id
  - message_id: 会话内对话唯一标识
  - source: { platform, organization_id, project_id, participants }
  - operator_id: 操作人用户ID
  - mode: "full"（全量替换）或 "patch"（路径精确修改）
  - full_payload: 全量替换时的新 payload JSON（mode=full 时必填）
  - path: 精确修改的 JSON 路径（如 "payload.title"，mode=patch 时必填）
  - value: 要修改成的新值（mode=patch 时必填）
  - history: 事件历史（字符串数组，表示历史事件的 message_id，如 ["msg-001", "msg-002"]）

### 9 add_assignee（特定 agent 扩展事件，仅 Miss Spec agent 等实现需要）
- 说明：用于将用户加入当前 session 的 assignees（责任人）列表，主要用于结构化负载（如 Miss Spec agent 的 Task）责任人管理，非协议通用事件。
- 用法：
  - `/agent add_assignee @Bob @Carol`
- 目的：动态增加结构化负载或子结构的责任人。
- 游向：捕获端（IM）本地存储
- 字段：
  - type: "add_assignee"
  - session_id
  - message_id: 会话内对话唯一标识
  - source: { platform, organization_id, project_id, participants }
  - operator_id: 操作人用户ID
  - assignees: 变更后完整责任人列表
  - target_assignees: 本次要加入的责任人列表（必选，支持单个或多个）
  - history: 事件历史（字符串数组，表示历史事件的 message_id，如 ["msg-001", "msg-002"]）

### 10 remove_assignee（特定 agent 扩展事件，仅 Miss Spec agent 等实现需要）
- 说明：用于将用户移出当前 session 的 assignees（责任人）列表，主要用于结构化负载（如 Miss Spec agent 的 Task）责任人管理，非协议通用事件。
- 用法：
  - `/agent remove_assignee @Carol`
- 目的：动态移除结构化负载或子结构的责任人。
- 游向：捕获端（IM）本地存储
- 字段：
  - type: "remove_assignee"
  - session_id
  - message_id: 会话内对话唯一标识
  - source: { platform, organization_id, project_id, participants }
  - operator_id: 操作人用户ID
  - assignees: 变更后完整责任人列表
  - target_assignees: 本次要移除的责任人列表（必选，支持单个或多个）
  - history: 事件历史（字符串数组，表示历史事件的 message_id，如 ["msg-001", "msg-002"]）
