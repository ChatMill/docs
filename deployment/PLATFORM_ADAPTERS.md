# Chatmill 平台适配说明

## 平台适配架构

Chatmill 采用插件式适配器架构，支持多个捕获端和发布端平台的扩展。每个平台通过独立的适配器模块实现与核心系统的对接。

### 适配器分类

| 适配器类型 | 名称                | 说明           | 状态   |
|------------|---------------------|----------------|--------|
| 捕获端     | Discord Bot         | IM 消息捕获     | 已实现 |
| 捕获端     | Slack Bot           | IM 消息捕获     | 计划中 |
| 捕获端     | Telegram Bot        | IM 消息捕获     | 计划中 |
| 发布端     | GitHub Publisher    | 结构化需求发布   | 已实现 |
| 发布端     | Notion Publisher    | 结构化内容发布   | 计划中 |
| 发布端     | Jira Publisher      | 结构化需求发布   | 计划中 |

## 平台适配要点

### 1. 消息抽象与统一

- 每个平台格式不一样（如 Discord ID、Slack timestamp）
- 使用统一的 message abstraction 进行转换
- 中间语言（CML）统一表达：Section、Title、Tags

### 2. 内容映射

- 平台 API 结构差异（如 Notion 是块，GitHub 是 issue）
- 通过中间语言统一表达
- 支持双向映射：平台原生格式 ↔ CML

### 3. 事件路由

- 基于 platform 字段进行消息路由
- 支持动态切换目标平台
- 多平台并行发布能力

## 已支持平台说明

### Discord（捕获端）

- **集成方式**：Bot + Slash Commands + Reactions
- **主要用途**：主工作场所，需求讨论与捕获

### GitHub（发布端）

- **集成方式**：GitHub API v4 (GraphQL)
- **主要用途**：项目管理，Issue 跟踪
- **输出格式**：
  - Draft Issue
  - Project Card
  - Issue with Labels

### Notion（发布端）

- **集成方式**：Notion API
- **主要用途**：文档管理，知识库
- **输出格式**：
  - Database Row
  - Page with Blocks

