# Agent 能力插件用户指令与交互说明

## 文档用途
- 概述 agent 能力插件在 IM、用户、系统之间的全流程交互、命令、事件、异常处理等规范。
- 帮助开发者、对接方快速理解和实现与 agent/Chatmill 的集成。

## 主要内容结构
- 交互流程总览
- 事件类型与命令用法（含所有字段、示例、说明）
- 典型完整业务流转示例
- 多人与会人动态变更与溯源
- 业务异常与冲突场景总汇（含常见错误、友好提示语）

---

## 1. 指令格式

- **帮助 help**
  - `/agent help`
  - 说明：查看所有支持的命令、参数格式和用法说明，推荐新用户或遇到异常时优先使用。

- **发起需求草案 initiate**
  - `/agent initiate <消息区间/ID列表> [@user1 @user2 ...]`
  - 例：`/agent initiate 1..13 @alice @bob`
  - 例：`/agent initiate 11241,11245,11250 @alice @bob`
  - 说明：可在发起时直接圈定与会人，初始 participants 即为被圈用户，未圈人则默认为发起人。

- **补充 supplement_response**
  - `/agent supplement <session_id> <消息区间/ID列表>`
  - 例：`/agent supplement sess-001 14..20`
  - 说明：仅与会人可补充，补充内容可为消息区间或具体消息ID。

- **最终确认 confirmation**
  - `/agent confirm <session_id>`
  - 例：`/agent confirm sess-001`
  - 说明：仅与会人可确认。

- **与会人管理**
  - `/agent add_participant @user1 @user2 ...`  增加与会人
  - `/agent remove_participant @user1 @user2 ...`  移除与会人
  - 例：`/agent add_participant @alice @bob`
  - 例：`/agent remove_participant @alice`

- **责任人管理**
  - `/agent add_assignee @user1 @user2 ...`  增加责任人
  - `/agent remove_assignee @user1 @user2 ...`  移除责任人

- **结构化负载修改 modify**
  - `/agent modify <session_id> {payload的json}`  修改结构化负载（如任务树、checklist等），payload结构由agent能力插件定义
  - 示例（Miss Spec agent 的 Task 实现，仅为 agent 能力插件实现示例）：
    `/agent modify sess-001 { "missspec_id": "msk-001", "title": "快递柜扫码取件需求", ... }`
  - 其他agent可有不同payload结构

---

## 2. 典型使用场景与流程

1. 团队成员在 IM 频道讨论需求。
2. 任一成员输入 `/agent initiate ... [@user]`，Bot 自动抓取消息，发起需求（initiate），并圈定与会人。
3. 如需补充，Bot 会在频道发起 supplement_request（带 session_id），与会人可用 `/agent supplement` 补充。
4. 频道成员可用 `/agent add_participant @user` 增加与会人，或 `/agent remove_participant @user` 移除。
5. 只有与会人可用 `/agent supplement` 和 `/agent confirm` 对需求进行补充和确认。
6. 需求发布到发布端平台后，Bot 会自动回显发布结果。
7. 支持多轮补充、动态与会人变更、责任人管理、手动修改等高级操作。

---

## 3. 用户输入限制与常见问题

- 所有涉及消息区间（如 /agent initiate 1..13、/agent supplement sess-001 14..20）的一次性输入区间最大不得超过25条消息，超出将被拒绝并提示"区间不能超过25条"。
- 区间格式必须为 xxx..yyy（如 1..13），不支持 -、.、, 等其他分隔符。
- @用户必须为频道内有效用户，且格式为 @用户名。
- 命令参数顺序、数量、类型必须严格符合 INTERACTION_GUIDE.md 第2章字段表与示例。
- 其他详细限制与异常场景请参见 INTERACTION_GUIDE.md 第5章。

---

## 4. 相关文档
- [开发者指南 DEV_GUIDE.md](../developer/ARCHITECTURE.md)
- [领域职责说明 DOMAIN.md](../developer/DOMAIN.md)

# 在命令参数、示例、说明等涉及 session_id、频道、服务器等处补充：
> organization_id/project_id/agent 字段为平台无关抽象，映射关系如下：
> - Discord: organization_id = server_id, project_id = channel_id
> - Slack: organization_id = workspace_id, project_id = channel_id
> - 其他平台以适配器文档为准

agent 字段用于标识本 session 归属的 agent 能力插件（如 missspec、nudge、echo），支持多 agent 扩展和策略分发。
