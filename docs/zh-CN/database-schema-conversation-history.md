# Dify 对话历史数据统计表结构说明文档

## 概述

本文档详细说明 Dify 平台中与对话历史相关的数据库表结构，包括对话、消息、反馈、标注等核心数据表，以及它们之间的关联关系。这些表用于支持对话历史的数据统计，包括次数统计、问答原文记录、分类、用户反馈（点赞/点踩）等功能。

## 核心数据表

### 1. conversations（对话表）

存储应用中的对话会话信息。

#### 字段说明

| 字段名 | 数据类型 | 说明 |
|--------|---------|------|
| id | StringUUID | 对话的唯一标识符（主键） |
| app_id | StringUUID | 所属应用的 ID（外键关联 apps 表） |
| app_model_config_id | StringUUID | 应用模型配置 ID，可为空 |
| model_provider | String(255) | 模型提供商名称，如 openai、anthropic 等 |
| override_model_configs | LongText | 覆盖的模型配置信息（JSON 格式） |
| model_id | String(255) | 使用的模型 ID |
| mode | String(255) | 对话模式，如 chat、completion、workflow 等 |
| name | String(255) | 对话名称 |
| summary | LongText | 对话摘要 |
| inputs | JSON | 对话输入参数（JSON 格式） |
| introduction | LongText | 对话引导语 |
| system_instruction | LongText | 系统指令 |
| system_instruction_tokens | Integer | 系统指令的 token 数量 |
| status | String(255) | 对话状态，如 normal、completed 等 |
| invoke_from | String(255) | 调用来源，记录对话的创建方式 |
| from_source | String(255) | 数据来源，如 api、console、web 等 |
| from_end_user_id | StringUUID | 终端用户 ID（外键关联 end_users 表） |
| from_account_id | StringUUID | 账户 ID（外键关联 accounts 表） |
| read_at | DateTime | 对话被阅读的时间 |
| read_account_id | StringUUID | 阅读对话的账户 ID |
| dialogue_count | Integer | 对话轮次计数（默认值：0） |
| is_deleted | Boolean | 是否已删除（默认值：false） |
| created_at | DateTime | 创建时间（自动生成） |
| updated_at | DateTime | 更新时间（自动更新） |

#### 索引

- `conversation_pkey`: 主键索引（id）
- `conversation_app_from_user_idx`: 复合索引（app_id, from_source, from_end_user_id）

#### 用途

- 统计应用的对话总数和活跃对话数
- 按用户、来源、时间段统计对话数据
- 追踪对话状态和配置信息
- 记录对话轮次（dialogue_count）

---

### 2. messages（消息表）

存储对话中的每条消息记录，包括用户问题和 AI 回答。

#### 字段说明

| 字段名 | 数据类型 | 说明 |
|--------|---------|------|
| id | StringUUID | 消息的唯一标识符（主键） |
| app_id | StringUUID | 所属应用的 ID |
| model_provider | String(255) | 模型提供商名称 |
| model_id | String(255) | 使用的模型 ID |
| override_model_configs | LongText | 覆盖的模型配置（JSON 格式） |
| conversation_id | StringUUID | 所属对话 ID（外键关联 conversations 表） |
| inputs | JSON | 消息输入参数（JSON 格式） |
| query | LongText | 用户提问的原文 |
| message | JSON | 消息内容（JSON 格式，包含消息的详细信息） |
| message_tokens | Integer | 问题消耗的 token 数量（默认值：0） |
| message_unit_price | Numeric(10,4) | 问题的单位价格 |
| message_price_unit | Numeric(10,7) | 问题的价格单位（默认值：0.001） |
| answer | LongText | AI 回答的原文 |
| answer_tokens | Integer | 回答消耗的 token 数量（默认值：0） |
| answer_unit_price | Numeric(10,4) | 回答的单位价格 |
| answer_price_unit | Numeric(10,7) | 回答的价格单位（默认值：0.001） |
| parent_message_id | StringUUID | 父消息 ID，用于消息链追踪 |
| provider_response_latency | Float | 提供商响应延迟时间（秒） |
| total_price | Numeric(10,7) | 总价格 |
| currency | String(255) | 货币类型 |
| status | String(255) | 消息状态，如 normal、error 等（默认值：normal） |
| error | LongText | 错误信息（如果有） |
| message_metadata | LongText | 消息元数据（JSON 格式） |
| invoke_from | String(255) | 调用来源 |
| from_source | String(255) | 数据来源 |
| from_end_user_id | StringUUID | 终端用户 ID |
| from_account_id | StringUUID | 账户 ID |
| agent_based | Boolean | 是否基于 Agent（默认值：false） |
| workflow_run_id | StringUUID | 工作流运行 ID（如果是工作流消息） |
| app_mode | String(255) | 应用模式 |
| created_at | DateTime | 创建时间（自动生成） |
| updated_at | DateTime | 更新时间（自动更新） |

#### 索引

- `message_pkey`: 主键索引（id）
- `message_app_id_idx`: 复合索引（app_id, created_at）
- `message_conversation_id_idx`: 单列索引（conversation_id）
- `message_end_user_idx`: 复合索引（app_id, from_source, from_end_user_id）
- `message_account_idx`: 复合索引（app_id, from_source, from_account_id）
- `message_workflow_run_id_idx`: 复合索引（conversation_id, workflow_run_id）
- `message_created_at_idx`: 单列索引（created_at）
- `message_app_mode_idx`: 单列索引（app_mode）

#### 用途

- 记录完整的问答对话内容（query 和 answer 字段）
- 统计消息数量、token 使用量和成本
- 分析响应时间和性能指标
- 追踪消息状态和错误信息
- 支持按时间、用户、应用等维度的数据统计

---

### 3. message_feedbacks（消息反馈表）

存储用户对消息的反馈信息，包括点赞（like）和点踩（dislike）。

#### 字段说明

| 字段名 | 数据类型 | 说明 |
|--------|---------|------|
| id | StringUUID | 反馈的唯一标识符（主键） |
| app_id | StringUUID | 所属应用的 ID |
| conversation_id | StringUUID | 所属对话 ID |
| message_id | StringUUID | 被反馈的消息 ID |
| rating | String(255) | 反馈评级，值为 "like"（点赞）或 "dislike"（点踩） |
| content | LongText | 反馈的详细内容或评论（可选） |
| from_source | String(255) | 反馈来源 |
| from_end_user_id | StringUUID | 终端用户 ID |
| from_account_id | StringUUID | 账户 ID |
| created_at | DateTime | 创建时间（自动生成） |
| updated_at | DateTime | 更新时间（自动更新） |

#### 索引

- `message_feedback_pkey`: 主键索引（id）
- `message_feedback_app_idx`: 单列索引（app_id）
- `message_feedback_message_idx`: 复合索引（message_id, from_source）
- `message_feedback_conversation_idx`: 复合索引（conversation_id, from_source, rating）

#### 用途

- 统计用户反馈数据（点赞/点踩比例）
- 分析消息质量和用户满意度
- 按应用、对话、时间等维度统计反馈情况
- 识别需要改进的消息类型

---

## 关联数据表

### 4. message_annotations（消息标注表）

存储对消息的人工标注信息，用于优化和改进 AI 回答。

#### 字段说明

| 字段名 | 数据类型 | 说明 |
|--------|---------|------|
| id | StringUUID | 标注的唯一标识符（主键） |
| app_id | StringUUID | 所属应用的 ID |
| conversation_id | StringUUID | 所属对话 ID（外键关联 conversations 表） |
| message_id | StringUUID | 被标注的消息 ID |
| question | LongText | 标注的问题（可选） |
| content | LongText | 标注的回答内容 |
| hit_count | Integer | 标注被命中的次数（默认值：0） |
| account_id | StringUUID | 创建标注的账户 ID |
| created_at | DateTime | 创建时间（自动生成） |
| updated_at | DateTime | 更新时间（自动更新） |

#### 索引

- `message_annotation_pkey`: 主键索引（id）
- `message_annotation_app_idx`: 单列索引（app_id）
- `message_annotation_conversation_idx`: 单列索引（conversation_id）
- `message_annotation_message_idx`: 单列索引（message_id）

#### 用途

- 记录人工标注的优质回答
- 统计标注使用情况（hit_count）
- 用于回答质量优化和模型微调
- 分类和管理标注数据

---

### 5. message_files（消息文件表）

存储与消息关联的文件信息。

#### 字段说明

| 字段名 | 数据类型 | 说明 |
|--------|---------|------|
| id | StringUUID | 文件记录的唯一标识符（主键） |
| message_id | StringUUID | 关联的消息 ID |
| type | String(255) | 文件类型，如 image、document、audio、video 等 |
| transfer_method | String(255) | 传输方法，如 local_file、remote_url、tool_file 等 |
| url | LongText | 文件 URL（可选） |
| belongs_to | String(255) | 归属方，值为 "user" 或 "assistant" |
| upload_file_id | StringUUID | 上传文件 ID |
| created_by_role | String(255) | 创建者角色 |
| created_by | StringUUID | 创建者 ID |
| created_at | DateTime | 创建时间（自动生成） |

#### 索引

- `message_file_pkey`: 主键索引（id）
- `message_file_message_idx`: 单列索引（message_id）
- `message_file_created_by_idx`: 单列索引（created_by）

#### 用途

- 记录消息中上传或生成的文件
- 统计文件类型和数量
- 追踪文件的归属和来源

---

### 6. message_agent_thoughts（消息 Agent 思考过程表）

存储 Agent 模式下的思考过程和工具调用信息。

#### 字段说明

| 字段名 | 数据类型 | 说明 |
|--------|---------|------|
| id | StringUUID | 记录的唯一标识符（主键） |
| message_id | StringUUID | 关联的消息 ID |
| message_chain_id | StringUUID | 消息链 ID（可选） |
| position | Integer | 在思考链中的位置 |
| thought | LongText | Agent 的思考内容 |
| tool | LongText | 使用的工具名称（多个工具用分号分隔） |
| tool_labels_str | LongText | 工具标签（JSON 格式，默认值：'{}'） |
| tool_meta_str | LongText | 工具元数据（JSON 格式，默认值：'{}'） |
| tool_input | LongText | 工具输入参数（JSON 格式） |
| observation | LongText | 工具执行观察结果（JSON 格式） |
| tool_process_data | LongText | 工具处理过程数据 |
| message | LongText | Agent 消息内容 |
| message_token | Integer | 消息 token 数量 |
| message_unit_price | Numeric | 消息单位价格 |
| message_price_unit | Numeric(10,7) | 消息价格单位（默认值：0.001） |
| message_files | LongText | 消息文件列表（JSON 格式） |
| answer | LongText | Agent 回答内容 |
| answer_token | Integer | 回答 token 数量 |
| answer_unit_price | Numeric | 回答单位价格 |
| answer_price_unit | Numeric(10,7) | 回答价格单位（默认值：0.001） |
| tokens | Integer | 总 token 数量 |
| total_price | Numeric | 总价格 |
| currency | String(255) | 货币类型 |
| latency | Float | 延迟时间（秒） |
| created_by_role | String(255) | 创建者角色 |
| created_by | StringUUID | 创建者 ID |
| created_at | DateTime | 创建时间（自动生成） |

#### 索引

- `message_agent_thought_pkey`: 主键索引（id）
- `message_agent_thought_message_id_idx`: 单列索引（message_id）
- `message_agent_thought_message_chain_id_idx`: 单列索引（message_chain_id）

#### 用途

- 记录 Agent 的完整思考和决策过程
- 统计工具调用次数和类型
- 分析 Agent 性能和成本
- 调试和优化 Agent 行为

---

## 用户相关表

### 7. end_users（终端用户表）

存储应用的终端用户信息。

#### 字段说明

| 字段名 | 数据类型 | 说明 |
|--------|---------|------|
| id | StringUUID | 用户的唯一标识符（主键） |
| tenant_id | StringUUID | 租户 ID |
| app_id | StringUUID | 关联的应用 ID |
| type | String(255) | 用户类型 |
| external_user_id | String(255) | 外部用户 ID（可选） |
| name | String(255) | 用户名称 |
| is_anonymous | Boolean | 是否匿名用户（默认值：true） |
| session_id | String(255) | 会话 ID |
| created_at | DateTime | 创建时间（自动生成） |
| updated_at | DateTime | 更新时间（自动更新） |

#### 索引

- `end_user_pkey`: 主键索引（id）
- `end_user_session_id_idx`: 复合索引（session_id, type）
- `end_user_tenant_session_id_idx`: 复合索引（tenant_id, session_id, type）

#### 用途

- 识别和管理终端用户
- 统计用户活跃度和使用情况
- 关联用户的对话和消息记录

---

### 8. accounts（账户表）

存储平台内部账户信息（管理员、成员等）。

#### 字段说明

此表在 `api/models/account.py` 中定义，包含账户的基本信息、角色、权限等。主要用于识别对话的内部创建者和管理者。

#### 用途

- 区分终端用户和内部账户
- 追踪对话和消息的创建者
- 管理账户权限和角色

---

## 应用相关表

### 9. apps（应用表）

存储 Dify 平台上的应用信息。

#### 字段说明

| 字段名 | 数据类型 | 说明 |
|--------|---------|------|
| id | StringUUID | 应用的唯一标识符（主键） |
| tenant_id | StringUUID | 租户 ID |
| name | String(255) | 应用名称 |
| description | LongText | 应用描述 |
| mode | String(255) | 应用模式，如 chat、completion、workflow、agent-chat 等 |
| icon_type | String(255) | 图标类型，值为 "image" 或 "emoji" |
| icon | String(255) | 图标内容 |
| icon_background | String(255) | 图标背景色 |
| app_model_config_id | StringUUID | 应用模型配置 ID |
| workflow_id | StringUUID | 工作流 ID（如果是工作流应用） |
| status | String(255) | 应用状态（默认值：normal） |
| enable_site | Boolean | 是否启用站点 |
| enable_api | Boolean | 是否启用 API |
| api_rpm | Integer | API 每分钟请求限制（默认值：0） |
| api_rph | Integer | API 每小时请求限制（默认值：0） |
| is_demo | Boolean | 是否演示应用（默认值：false） |
| is_public | Boolean | 是否公开应用（默认值：false） |
| is_universal | Boolean | 是否通用应用（默认值：false） |
| created_by | StringUUID | 创建者 ID |
| created_at | DateTime | 创建时间（自动生成） |
| updated_at | DateTime | 更新时间（自动更新） |

#### 索引

- `app_pkey`: 主键索引（id）
- `app_tenant_id_idx`: 单列索引（tenant_id）

#### 用途

- 组织和管理应用
- 统计应用级别的对话和消息数据
- 关联所有对话历史到具体应用

---

## 数据表关联关系

### 核心关联关系图

```
apps (应用表)
  ├── conversations (对话表) [app_id]
  │     ├── messages (消息表) [conversation_id]
  │     │     ├── message_feedbacks (反馈表) [message_id, conversation_id]
  │     │     ├── message_files (文件表) [message_id]
  │     │     ├── message_agent_thoughts (Agent思考表) [message_id]
  │     │     └── message_annotations (标注表) [message_id]
  │     └── message_annotations (标注表) [conversation_id]
  │
  ├── end_users (终端用户表) [app_id]
  └── accounts (账户表) [创建者关联]
```

### 详细关联说明

1. **应用与对话的关系（一对多）**
   - `apps.id` → `conversations.app_id`
   - 一个应用可以有多个对话

2. **对话与消息的关系（一对多）**
   - `conversations.id` → `messages.conversation_id`
   - 一个对话包含多条消息
   - 外键约束确保数据完整性

3. **消息与反馈的关系（一对多）**
   - `messages.id` → `message_feedbacks.message_id`
   - 一条消息可以有多个反馈记录（但通常每个用户只有一个）

4. **消息与标注的关系（一对多）**
   - `messages.id` → `message_annotations.message_id`
   - 一条消息可以有多个标注版本

5. **消息与文件的关系（一对多）**
   - `messages.id` → `message_files.message_id`
   - 一条消息可以包含多个文件

6. **消息与 Agent 思考的关系（一对多）**
   - `messages.id` → `message_agent_thoughts.message_id`
   - 一条 Agent 消息可以包含多个思考步骤

7. **用户关联关系**
   - `end_users.id` → `conversations.from_end_user_id`
   - `end_users.id` → `messages.from_end_user_id`
   - `accounts.id` → `conversations.from_account_id`
   - `accounts.id` → `messages.from_account_id`
   - 区分终端用户和内部账户创建的对话

---

## 数据统计应用场景

> **注意**: 以下 SQL 示例使用 PostgreSQL 语法。Dify 主要支持 PostgreSQL 数据库。

### 1. 对话次数统计

```sql
-- 统计应用的总对话数
SELECT app_id, COUNT(*) as conversation_count
FROM conversations
WHERE is_deleted = false
GROUP BY app_id;

-- 统计时间段内的对话数
SELECT DATE(created_at) as date, COUNT(*) as count
FROM conversations
WHERE app_id = 'xxx' AND created_at >= '2024-01-01'
GROUP BY DATE(created_at);
```

### 2. 消息问答统计

```sql
-- 统计消息总数
SELECT app_id, COUNT(*) as message_count
FROM messages
WHERE status = 'normal'
GROUP BY app_id;

-- 获取问答原文
SELECT 
    m.id,
    m.query as question,
    m.answer as answer,
    m.created_at
FROM messages m
WHERE m.conversation_id = 'xxx'
ORDER BY m.created_at;
```

### 3. 用户反馈统计

```sql
-- 统计点赞/点踩比例
-- 使用窗口函数计算每个应用中各评级类型的百分比
SELECT 
    app_id,
    rating,
    COUNT(*) as count,
    ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (PARTITION BY app_id), 2) as percentage
FROM message_feedbacks
GROUP BY app_id, rating;

-- 统计某个时间段的反馈情况
SELECT 
    DATE(created_at) as date,
    rating,
    COUNT(*) as count
FROM message_feedbacks
WHERE app_id = 'xxx' AND created_at >= '2024-01-01'
GROUP BY DATE(created_at), rating;
```

### 4. 标注分类统计

```sql
-- 统计标注数量和命中次数
SELECT 
    app_id,
    COUNT(*) as annotation_count,
    SUM(hit_count) as total_hits,
    AVG(hit_count) as avg_hits
FROM message_annotations
GROUP BY app_id;

-- 查找热门标注
SELECT 
    id,
    question,
    content,
    hit_count
FROM message_annotations
WHERE app_id = 'xxx'
ORDER BY hit_count DESC
LIMIT 10;
```

### 5. 性能和成本统计

```sql
-- 统计 token 使用量和成本
SELECT 
    app_id,
    COUNT(*) as message_count,
    SUM(message_tokens) as total_input_tokens,
    SUM(answer_tokens) as total_output_tokens,
    SUM(message_tokens + answer_tokens) as total_tokens,
    SUM(total_price) as total_cost
FROM messages
WHERE created_at >= '2024-01-01'
GROUP BY app_id;

-- 统计平均响应时间
SELECT 
    app_id,
    AVG(provider_response_latency) as avg_latency,
    MAX(provider_response_latency) as max_latency,
    MIN(provider_response_latency) as min_latency
FROM messages
WHERE status = 'normal'
GROUP BY app_id;
```

### 6. 用户活跃度统计

```sql
-- 统计活跃用户数（最近7天，可根据需要调整天数）
SELECT 
    app_id,
    COUNT(DISTINCT from_end_user_id) as active_users
FROM conversations
WHERE created_at >= NOW() - INTERVAL '7 days'
GROUP BY app_id;

-- 统计用户对话频率
SELECT 
    from_end_user_id,
    COUNT(*) as conversation_count,
    COUNT(DISTINCT DATE(created_at)) as active_days
FROM conversations
WHERE app_id = 'xxx'
GROUP BY from_end_user_id
ORDER BY conversation_count DESC;
```

---

## 注意事项

1. **数据来源标识（from_source）**
   - 用于区分数据的来源渠道，常见值包括：
     - `api`: 通过 API 调用
     - `console`: 从控制台创建
     - `web`: 从 Web 界面创建
     - `service-api`: 从服务 API 调用

2. **UUID 类型字段**
   - 大多数 ID 字段使用 UUID 格式（StringUUID 类型）
   - 确保全局唯一性和分布式环境下的数据一致性

3. **JSON 字段**
   - `inputs`、`message`、`override_model_configs` 等字段存储 JSON 格式数据
   - 查询时需要使用对应数据库的 JSON 函数

4. **软删除**
   - `conversations` 表使用 `is_deleted` 字段实现软删除
   - 统计时需要注意过滤已删除的记录

5. **时间戳字段**
   - `created_at`: 记录创建时间（自动生成）
   - `updated_at`: 记录更新时间（自动更新）
   - 所有时间字段都使用 DateTime 类型

6. **价格和成本字段**
   - 使用 Numeric 类型存储精确的价格信息
   - `message_unit_price` 和 `answer_unit_price`: 单位价格
   - `message_price_unit` 和 `answer_price_unit`: 价格单位（默认 0.001）
   - `total_price`: 总价格

7. **索引优化**
   - 表中已创建多个索引以优化常见查询
   - 按时间、应用、用户等维度查询时可利用这些索引
   - 进行大规模统计时建议使用适当的索引

---

## 总结

本文档详细说明了 Dify 对话历史数据统计所涉及的核心数据表结构，包括：

- **核心表**：conversations、messages、message_feedbacks
- **关联表**：message_annotations、message_files、message_agent_thoughts
- **辅助表**：end_users、accounts、apps

这些表通过外键关系相互关联，形成完整的对话历史数据体系。利用这些表可以实现丰富的数据统计功能，包括对话次数统计、问答内容分析、用户反馈统计、性能分析、成本追踪等。

在实际应用中，建议：
1. 根据具体需求选择合适的表和字段进行查询
2. 利用已有的索引优化查询性能
3. 注意处理软删除和数据来源标识
4. 合理使用聚合函数进行数据统计分析
