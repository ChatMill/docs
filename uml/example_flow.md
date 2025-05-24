## 典型流程图示例

下图用 Mermaid sequenceDiagram 展示了 Miss Spec agent 在 Discord (IM平台) + GitHub (发布端) 场景下，多与会人动态变更的典型协作流程。payload 结构为 Miss Spec agent 的 Task 实现：

```mermaid
sequenceDiagram
    participant User as Channel成员（Alice/Bob/Carol/Dave）
    participant IM as Discord (IM平台)
    participant Capture as 捕获端处理器
    participant Agent as Miss Spec (Agent能力插件)
    participant Publisher as GitHub (发布端)
    participant Target as GitHub Issue

    %% 1. Alice 发起需求，圈 Bob
    User->>IM: /missspec initiate 1..13 @Bob
    IM->>Capture: initiate (msg-001, participants: Alice, Bob, message_ids: 1..13)
    Capture->>Agent: initiate

    %% 2. Alice 加 Carol
    User->>IM: /missspec add_participant Carol
    IM->>Capture: add_participant (msg-002, participants: Alice, Bob, Carol)

    %% 3. Bob 加 Dave
    User->>IM: /missspec add_participant Dave
    IM->>Capture: add_participant (msg-003, participants: Alice, Bob, Carol, Dave)

    %% 4. Miss Spec 请求补充
    Agent->>Capture: supplement_request (msg-004, question: 请补充子任务, participants: Alice, Bob, Carol, Dave)
    Capture->>IM: supplement_request
    IM->>User: 补充请求

    %% 5. Alice 补充内容
    User->>IM: /missspec supplement (扫码流程设计, 柜员通知机制)
    IM->>Capture: supplement_response (msg-005, sub_tasks: [扫码流程设计, 柜员通知机制], participants: Alice, Bob, Carol, Dave)
    Capture->>Agent: supplement_response

    %% 6. Bob 移除 Carol
    User->>IM: /missspec remove_participant Carol
    IM->>Capture: remove_participant (msg-006, participants: Alice, Bob, Dave)

    %% 7. Dave 确认
    User->>IM: /missspec publish
    IM->>Capture: publish (msg-007, participants: Alice, Bob, Dave)
    Capture->>Publisher: publish（直接触发发布端发布）
    Publisher->>Target: 创建/同步 issue
    Target-->>Publisher: 发布结果 (msg-010, sub_tasks: [扫码流程设计（修正版）, 柜员通知机制], id: msk-001)
    Publisher->>Capture: publish_result
    Capture->>IM: publish_result
    IM->>User: 发布结果回显
    Note over Publisher,Capture: 发布结果由发布端直接回传捕获端
```

> 说明：本例为 Miss Spec agent 在 Discord (IM平台) + GitHub (发布端) 场景下的动态与会人变更协作流程，payload 结构为 Miss Spec agent 的 Task 实现。 