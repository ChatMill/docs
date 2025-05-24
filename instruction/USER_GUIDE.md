# Chatmill 终端用户操作手册（User Guide）

## 1. 适用对象

本手册面向终端用户、团队成员、产品经理，帮助快速上手 Chatmill 的需求捕捉与协同流程。

## 2. 典型使用场景

### 场景一：团队需求讨论与发布
1. 团队成员在 Discord 频道自由讨论需求。
2. 任一成员输入 `/missspec initiate 1..13 @alice @bob`，Bot 自动抓取消息区间，发起需求草案。
3. Bot 在频道中回显草案卡片，圈定与会人。
4. 与会人可用 `/missspec supplement sess-001 14..20` 补充细节，或 `/missspec add_participant @carol` 动态添加成员。
5. 讨论充分后，任一与会人输入 `/missspec confirm sess-001 --to github`，需求正式发布到 GitHub。
6. Bot 自动回显发布结果，附带链接。

### 场景二：多轮补充与动态协作
1. 需求发起后，Bot 检测到信息不全，自动发起 supplement_request。
2. 与会人多轮补充，Bot 每次回显最新草案。
3. 责任人可用 `/missspec add_assignee @dave` 指定。
4. 需求确认后，自动同步到目标平台。

### 场景三：手动修改与补充
1. 团队可用 `/missspec modify` 命令手动补充或修改需求内容。

## 3. 用户操作流程

1. 进入 Discord 频道，确认 Bot 在线。
2. 输入 `/missspec help` 查看所有支持的命令。
3. 按需发起、补充、确认、管理与会人/责任人。
4. 遇到异常或无响应，优先查阅 FAQ 或联系管理员。

## 4. 常见问题与支持

- 详见 [FAQ.md](./FAQ.md)
- 配置与平台适配见 [PLATFORM_ADAPTERS.md](../deployment/PLATFORM_ADAPTERS.md)

## 5. 进阶用法与建议

- 善用多轮补充、动态成员管理，提升协作效率。
- 充分利用任务树和子任务，便于后续追踪和分工。
- 发布前建议所有与会人确认，减少遗漏。
- 如需扩展新平台或自定义流程，请联系项目维护者。

---

> 本手册为结构归档，后续可持续补充典型案例、操作视频、常见问题等内容。
