# INSTRUCTION

本说明文档为 Miss Spec/Chatmill 的交互入口指引，适用于所有需要对接、开发、测试 Miss Spec/Chatmill IM 端、上游处理器、命令/事件流的开发者和产品。

## 文档用途
- 概述 Miss Spec/Chatmill 在 IM、用户、系统之间的全流程交互、命令、事件、异常处理等规范。
- 帮助开发者、对接方快速理解和实现与 Miss Spec/Chatmill 的集成。

## 主要内容结构
- 交互流程总览
- 事件类型与命令用法（含所有字段、示例、说明）
- 典型完整业务流转示例
- 多人与会人动态变更与溯源
- 业务异常与冲突场景总汇（含常见错误、友好提示语）

## 详细交互协议与示例

[INTERACTION_GUIDE.md](./INTERACTION_GUIDE.md)

如有任何疑问或对接需求，或联系项目维护者。

## 1. 指令格式

- **帮助 help**
  - `/missspec help`
  - 说明：查看所有支持的命令、参数格式和用法说明，推荐新用户或遇到异常时优先使用。

- **发起需求草案 initiate**
  - `/missspec initiate <消息区间/ID列表> [@user1 @user2 ...]`
  - 例：`/missspec initiate 1..13 @alice @bob`
  - 例：`/missspec initiate 11241,11245,11250 @alice @bob`
  - 说明：可在发起时直接圈定与会人，初始 participants 即为被圈用户，未圈人则默认为发起人。

- **补充 supplement_response**
  - `/missspec supplement <session_id> <消息区间/ID列表>`
  - 例：`/missspec supplement sess-001 14..20`
  - 说明：仅与会人可补充，补充内容可为消息区间或具体消息ID。

- **最终确认 confirmation**
  - `/missspec confirm <session_id>`
  - 例：`/missspec confirm sess-001`
  - 说明：仅与会人可确认。

- **与会人管理**
  - `/missspec add_participant @user1 @user2 ...`  增加与会人
  - `/missspec remove_participant @user1 @user2 ...`  移除与会人
  - 例：`/missspec add_participant @alice @bob`
  - 例：`/missspec remove_participant @alice`

- **责任人管理**
  - `/missspec add_assignee @user1 @user2 ...`  增加责任人
  - `/missspec remove_assignee @user1 @user2 ...`  移除责任人

- **手动修改 manual_modify**
  - `/missspec manual {完整task的json}`  全量替换
  - `/missspec manual task.title 新的标题`  精确修改
  - `/missspec manual task.sub_tasks[0].title 新的子任务标题`

## 2. 典型使用场景

1. 团队成员在 Discord 频道讨论需求。
2. 任一成员输入 `/missspec initiate ... [@user]`，Bot 自动抓取消息，发起需求（initiate），并圈定与会人。
3. 如需补充，Bot 会在频道发起 supplement_request（带 session_id），与会人可用 `/missspec supplement` 补充。
4. 频道成员可用 `/missspec add_participant @user` 增加与会人，或 `/missspec remove_participant @user` 移除。
5. 只有与会人可用 `/missspec supplement` 和 `/missspec confirm` 对需求进行补充和确认。
6. 需求发布到下游平台（如 GitHub）后，Bot 会自动回显发布结果。
7. 支持多轮补充、动态与会人变更、责任人管理、手动修改等高级操作。

## 3. 交互流程示意

- 需求发起（可圈人）→ 多轮补充/与会人变更 → 最终确认 → 发布结果回显
- 所有多轮交互均以 session_id 贯穿，Bot 消息中会明确标注 session_id
- 与会人、责任人列表动态可变，Bot 会在消息中同步提示当前状态
- 支持手动补充、确认、责任人管理、任务树手动修改等全流程操作

## 4. 注意事项

- initiate 时如未圈人，participants 默认为发起人；如圈人则以圈定用户为初始 participants。
- 只有在 Miss Spec Bot 在频道内时，指令才会生效。
- session_id/message_id 请直接复制 Bot 消息中的标注，确保补充/确认能正确关联上下文。
- 频道内所有成员均可 initiate 需求，但只有与会人可补充和确认。
- 责任人必须是与会人。
- 如遇指令无响应或报错，请优先参考 INTERACTION_GUIDE.md 的异常场景与提示语，或联系管理员/开发者。

## 5. 相关文档
- [开发者指南 DEV_GUIDE.md](./DEV_GUIDE.md)
- [领域职责说明 DOMAIN.md](./DOMAIN.md)
- [Interaction Guide INTERACTION_GUIDE.md](./INTERACTION_GUIDE.md)

## 用户输入限制说明

- 所有涉及消息区间（如 /missspec initiate 1..13、/missspec supplement sess-001 14..20）的一次性输入区间最大不得超过25条消息，超出将被拒绝并提示"区间不能超过25条"。
- 区间格式必须为 xxx..yyy（如 1..13），不支持 -、.、, 等其他分隔符。
- @用户必须为频道内有效用户，且格式为 @用户名。
- 命令参数顺序、数量、类型必须严格符合 INTERACTION_GUIDE.md 第2章字段表与示例。
- 其他详细限制与异常场景请参见 INTERACTION_GUIDE.md 第5章。