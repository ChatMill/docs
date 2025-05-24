# Agent 能力插件常见问题（FAQ）

## 1. 指令相关

**Q1：为什么 /agent 指令无响应？**
A：请确认 Bot 已加入频道且具备消息读写、Slash Command 权限，检查 Token 配置和服务是否正常运行。

**Q2：如何指定需求发布到不同平台？**
A：在 initiate 或 publish 阶段通过 `--to <platform>` 参数指定目标平台，如 `/agent publish sess-001 --to github`。

**Q3：session_id、message_id 从哪里获取？**
A：Bot 在消息中会自动标注 session_id/message_id，补充/确认/溯源时请直接复制。

> organization_id/project_id/agent 字段为平台无关抽象，映射关系如下：
> - Discord: organization_id = server_id, project_id = channel_id
> - Slack: organization_id = workspace_id, project_id = channel_id
> - 其他平台以适配器文档为准

## 2. 权限与成员管理

**Q4：谁可以补充、确认需求？**
A：只有与会人（participants）可补充和确认，其他成员需先被添加为与会人。

**Q5：如何添加/移除与会人或责任人？**
A：使用 `/agent add_participant @user`、`/agent remove_participant @user`、`/agent add_assignee @user`、`/agent remove_assignee @user`。

## 3. 平台适配与集成

**Q6：支持哪些平台？如何扩展？**
A：目前支持 IM平台（捕获端）、结构化输出平台（发布端），未来可通过适配器机制扩展 Slack、Jira、Telegram 等。

**Q7：Webhook 配置失败怎么办？**
A：请检查 ALLOWED_PUBLISHER_BASE_URLS、WEBHOOK_SECRET 配置，确保回调来源和签名校验无误。

## 4. 配置与部署

**Q8：环境变量如何配置？**
A：详见 PLATFORM_ADAPTERS.md，所有敏感信息建议通过 .env 文件管理，生产环境务必配置 WEBHOOK_SECRET。

**Q9：如何本地调试 Bot？**
A：可单独运行捕获端 Bot 启动，或用 Postman/curl 测试 HTTP 路由。

## 5. 异常与报错

**Q10：遇到"区间不能超过25条"怎么办？**
A：请缩小消息区间或分批操作，单次最多支持25条消息。

**Q11：提示"仅与会人可操作"或"权限不足"？**
A：请确认自己已被添加为与会人或责任人，或联系管理员。

> organization_id/project_id/agent 字段为平台无关抽象，映射关系如下：
> - Discord: organization_id = server_id, project_id = channel_id
> - Slack: organization_id = workspace_id, project_id = channel_id
> - 其他平台以适配器文档为准

**Q12：命令格式/参数报错？**
A：请严格参照 INSTRUCTION.md 和 INTERACTION_GUIDE.md 的命令格式、参数顺序和类型。

## 6. 其他

**Q13：如何反馈新需求或 Bug？**
A：请在项目 Issue 区提交，或联系维护者。

**Q14：支持多语言/多时区吗？**
A：目前主要支持中文，时区以服务器配置为准，未来可扩展多语言和时区支持。

agent 字段用于标识本 session 归属的 agent 能力插件（如 missspec、nudge、echo），支持多 agent 扩展和策略分发。

---

> 更多详细说明请参见 INSTRUCTION.md、PLATFORM_ADAPTERS.md 等文档。
