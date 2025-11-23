# Dify 数据库表结构说明文档

## 文档概述

本文档详细描述了 Dify 平台的数据库表结构。Dify 是一个开源的 LLM 应用开发平台，提供了工作流编排、RAG 管道、Agent 能力和模型管理等功能。

### 技术栈
- **后端**: Python Flask + SQLAlchemy ORM
- **数据库**: PostgreSQL (支持其他关系型数据库)
- **架构模式**: 领域驱动设计 (DDD) + Clean Architecture

### 文档说明
- **模型类**: SQLAlchemy ORM 模型类名
- **字段类型**: Python 类型注解和 SQL 数据类型
- **可空性**: 标注字段是否允许为 NULL
- **默认值**: 字段的默认值设置
- **索引**: 数据库索引信息

---

## 目录

- [账户与租户管理](#账户与租户管理)
- [应用与对话管理](#应用与对话管理)
- [数据集与知识库](#数据集与知识库)
- [工作流管理](#工作流管理)
- [模型供应商管理](#模型供应商管理)
- [工具管理](#工具管理)
- [触发器管理](#触发器管理)
- [Web功能](#Web功能)
- [OAuth认证](#OAuth认证)
- [数据源认证](#数据源认证)
- [异步任务](#异步任务)
- [API扩展](#API扩展)
- [附录](#附录)

---


## 账户与租户管理

**说明**: 管理用户账户、租户、账户角色、邀请码等相关数据

**源文件**: `api/models/account.py`

### account_integrates

**模型类**: `AccountIntegrate`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `account_id` | String | StringUUID | 必填 |
| `provider` | String | String | 必填 |
| `open_id` | String | String | 必填 |
| `encrypted_token` | String | String | 必填 |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `updated_at` | DateTime | DateTime | 必填, 默认当前时间 |

---

### account_plugin_permissions

**模型类**: `TenantPluginPermission`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `tenant_id` | String | StringUUID | 必填 |
| `install_permission` | InstallPermission | String | 必填 |
| `debug_permission` | DebugPermission | String | 必填 |

---

### accounts

**模型类**: `Account`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `name` | String | String | 必填 |
| `email` | String | String | 必填 |
| `password` | String (可空) | String | 可空 |
| `password_salt` | String (可空) | String | 可空 |
| `avatar` | String (可空) | String | 可空 |
| `interface_language` | String (可空) | String | 可空 |
| `interface_theme` | String (可空) | String | 可空 |
| `timezone` | String (可空) | String | 可空 |
| `last_login_at` | DateTime (可空) | DateTime | 可空, 默认值: None |
| `last_login_ip` | String (可空) | String | 可空 |
| `last_active_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `status` | String | String | 必填 |
| `initialized_at` | DateTime (可空) | DateTime | 可空, 默认值: None |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `updated_at` | DateTime | DateTime | 必填, 默认当前时间 |

#### 索引

- `account_email_idx`

---

### invitation_codes

**模型类**: `InvitationCode`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | Integer | sa.Integer | 必填 |
| `batch` | String | String | 必填 |
| `code` | String | String | 必填 |
| `status` | String | String | 必填 |
| `used_at` | DateTime (可空) | DateTime | 可空, 默认值: None |
| `used_by_tenant_id` | String (可空) | StringUUID | 可空, 默认值: None |
| `used_by_account_id` | String (可空) | StringUUID | 可空, 默认值: None |
| `deprecated_at` | DateTime (可空) | DateTime | 可空, 默认值: None |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |

#### 索引

- `invitation_codes_batch_idx`
- `invitation_codes_code_idx`

---

### tenant_account_joins

**模型类**: `TenantAccountJoin`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `tenant_id` | String | StringUUID | 必填 |
| `account_id` | String | StringUUID | 必填 |
| `current` | Boolean | sa.Boolean | 必填, 默认值: sa.text("false" |
| `role` | String | String | 必填 |
| `invited_by` | String (可空) | StringUUID | 可空, 默认值: None |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `updated_at` | DateTime | DateTime | 必填, 默认当前时间 |

#### 索引

- `tenant_account_join_account_id_idx`
- `tenant_account_join_tenant_id_idx`

---

### tenant_plugin_auto_upgrade_strategies

**模型类**: `TenantPluginAutoUpgradeStrategy`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `tenant_id` | String | StringUUID | 必填 |
| `strategy_setting` | StrategySetting | String | 必填 |
| `upgrade_mode` | UpgradeMode | String | 必填 |
| `upgrade_time_of_day` | Integer | sa.Integer | 必填, 默认值: 0 |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `updated_at` | DateTime | DateTime | 必填, 默认当前时间 |

---

### tenants

**模型类**: `Tenant`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `name` | String | String | 必填 |
| `encrypt_public_key` | String (可空) | LongText | 可空, 默认值: None |
| `plan` | String | String | 必填 |
| `status` | String | String | 必填 |
| `custom_config` | String (可空) | LongText | 可空, 默认值: None |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `updated_at` | DateTime | DateTime | 必填, 默认当前时间 |

---


## 应用与对话管理

**说明**: 管理应用配置、对话记录、消息、站点设置等

**源文件**: `api/models/model.py`

### api_requests

**模型类**: `ApiRequest`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `tenant_id` | String | StringUUID | 必填 |
| `api_token_id` | String | StringUUID | 必填 |
| `path` | String | String | 必填 |
| `request` | String (可空) | LongText | 可空 |
| `response` | String (可空) | LongText | 可空 |
| `ip` | String | String | 必填 |
| `created_at` | DateTime | sa.DateTime | 必填, 默认当前时间 |

#### 索引

- `api_request_token_idx`

---

### api_tokens

**模型类**: `ApiToken`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `token` | String | String | 必填 |
| `id` | Unknown | StringUUID | 必填, 自动生成UUID |
| `app_id` | Unknown | StringUUID | 可空 |
| `tenant_id` | Unknown | StringUUID | 可空 |
| `type` | Unknown | String | 必填 |
| `last_used_at` | Unknown | sa | 可空 |
| `created_at` | Unknown | sa | 必填, 默认当前时间 |

#### 索引

- `api_token_app_id_type_idx`
- `api_token_token_idx`
- `api_token_tenant_idx`

---

### app_annotation_hit_histories

**模型类**: `AppAnnotationHitHistory`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `annotation_id` | String | StringUUID | 必填 |
| `id` | Unknown | StringUUID | 必填, 自动生成UUID |
| `app_id` | Unknown | StringUUID | 必填 |
| `source` | Unknown | LongText | 必填 |
| `question` | Unknown | LongText | 必填 |
| `account_id` | Unknown | StringUUID | 必填 |
| `created_at` | Unknown | sa | 必填, 默认当前时间 |
| `score` | Unknown | Float | 必填, 默认值: sa.text("0" |
| `message_id` | Unknown | StringUUID | 必填 |
| `annotation_question` | Unknown | LongText | 必填 |
| `annotation_content` | Unknown | LongText | 必填 |

#### 索引

- `app_annotation_hit_histories_app_idx`
- `app_annotation_hit_histories_account_idx`
- `app_annotation_hit_histories_annotation_idx`
- `app_annotation_hit_histories_message_idx`

---

### app_annotation_settings

**模型类**: `AppAnnotationSetting`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | Unknown | StringUUID | 必填, 自动生成UUID |
| `app_id` | Unknown | StringUUID | 必填 |
| `score_threshold` | Unknown | Float | 必填, 默认值: sa.text("0" |
| `collection_binding_id` | Unknown | StringUUID | 必填 |
| `created_user_id` | Unknown | StringUUID | 必填 |
| `created_at` | Unknown | sa | 必填, 默认当前时间 |
| `updated_user_id` | Unknown | StringUUID | 必填 |
| `updated_at` | Unknown | sa | 必填, 默认当前时间 |

#### 索引

- `app_annotation_settings_app_idx`

---

### app_mcp_servers

**模型类**: `AppMCPServer`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `tenant_id` | String | StringUUID | 必填 |
| `app_id` | String | StringUUID | 必填 |
| `name` | String | String | 必填 |
| `description` | String | String | 必填 |
| `server_code` | String | String | 必填 |
| `status` | String | String | 必填 |
| `parameters` | String | LongText | 必填 |
| `created_at` | DateTime | sa.DateTime | 必填, 默认当前时间 |
| `updated_at` | DateTime | sa.DateTime | 必填, 默认当前时间 |

---

### app_model_configs

**模型类**: `AppModelConfig`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | Unknown | StringUUID | 必填, 自动生成UUID |
| `app_id` | Unknown | StringUUID | 必填 |
| `provider` | Unknown | String | 必填 |
| `model_id` | Unknown | String | 必填 |
| `configs` | Unknown | sa | 可空 |
| `created_by` | Unknown | StringUUID | 可空 |
| `created_at` | Unknown | sa | 必填, 默认当前时间 |
| `updated_by` | Unknown | StringUUID | 可空 |
| `updated_at` | Unknown | sa | 必填, 默认当前时间 |
| `opening_statement` | Unknown | LongText | 必填 |
| `suggested_questions` | Unknown | LongText | 必填 |
| `suggested_questions_after_answer` | Unknown | LongText | 必填 |
| `speech_to_text` | Unknown | LongText | 必填 |
| `text_to_speech` | Unknown | LongText | 必填 |
| `more_like_this` | Unknown | LongText | 必填 |
| `model` | Unknown | LongText | 必填 |
| `user_input_form` | Unknown | LongText | 必填 |
| `dataset_query_variable` | Unknown | String | 必填 |
| `pre_prompt` | Unknown | LongText | 必填 |
| `agent_mode` | Unknown | LongText | 必填 |
| `sensitive_word_avoidance` | Unknown | LongText | 必填 |
| `retriever_resource` | Unknown | LongText | 必填 |
| `prompt_type` | Unknown | String | 必填 |
| `chat_prompt_config` | Unknown | LongText | 必填 |
| `completion_prompt_config` | Unknown | LongText | 必填 |
| `dataset_configs` | Unknown | LongText | 必填 |
| `external_data_tools` | Unknown | LongText | 必填 |
| `file_upload` | Unknown | LongText | 必填 |

#### 索引

- `app_app_id_idx`

---

### apps

**模型类**: `App`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `tenant_id` | String | StringUUID | 必填 |
| `name` | String | String | 必填 |
| `description` | String | LongText | 必填, 默认值: sa.text("''" |
| `mode` | String | String | 必填 |
| `icon_type` | String (可空) | String | 可空 |
| `icon_background` | String (可空) | String | 可空 |
| `status` | String | String | 必填 |
| `enable_site` | Boolean | sa.Boolean | 必填 |
| `enable_api` | Boolean | sa.Boolean | 必填 |
| `api_rpm` | Integer | sa.Integer | 必填, 默认值: sa.text("0" |
| `api_rph` | Integer | sa.Integer | 必填, 默认值: sa.text("0" |
| `is_demo` | Boolean | sa.Boolean | 必填, 默认值: sa.text("false" |
| `is_public` | Boolean | sa.Boolean | 必填, 默认值: sa.text("false" |
| `is_universal` | Boolean | sa.Boolean | 必填, 默认值: sa.text("false" |
| `updated_at` | DateTime | sa.DateTime | 必填, 默认当前时间 |
| `use_icon_as_answer_icon` | Boolean | sa.Boolean | 必填, 默认值: sa.text("false" |
| `icon` | Unknown | String | 必填 |
| `app_model_config_id` | Unknown | StringUUID | 可空 |
| `workflow_id` | Unknown | StringUUID | 可空 |
| `tracing` | Unknown | LongText | 可空 |
| `created_by` | Unknown | StringUUID | 可空 |
| `created_at` | Unknown | sa | 必填, 默认当前时间 |
| `updated_by` | Unknown | StringUUID | 可空 |

#### 索引

- `app_tenant_id_idx`

---

### conversations

**模型类**: `Conversation`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `mode` | String | String | 必填 |
| `name` | String | String | 必填 |
| `system_instruction_tokens` | Integer | sa.Integer | 必填, 默认值: sa.text("0" |
| `status` | String | String | 必填 |
| `from_source` | String | String | 必填 |
| `dialogue_count` | Integer | default=0 | 必填, 默认值: 0 |
| `is_deleted` | Boolean | sa.Boolean | 必填, 默认值: sa.text("false" |
| `app_id` | Unknown | StringUUID | 必填 |
| `app_model_config_id` | Unknown | StringUUID | 可空 |
| `model_provider` | Unknown | String | 必填 |
| `override_model_configs` | Unknown | LongText | 必填 |
| `model_id` | Unknown | String | 必填 |
| `summary` | Unknown | LongText | 必填 |
| `introduction` | Unknown | LongText | 必填 |
| `system_instruction` | Unknown | LongText | 必填 |
| `invoke_from` | Unknown | String | 必填 |
| `from_end_user_id` | Unknown | StringUUID | 必填 |
| `from_account_id` | Unknown | StringUUID | 必填 |
| `read_at` | Unknown | sa | 必填 |
| `read_account_id` | Unknown | StringUUID | 必填 |
| `created_at` | Unknown | sa | 必填, 默认当前时间 |
| `updated_at` | Unknown | sa | 必填, 默认当前时间 |

#### 索引

- `conversation_app_from_user_idx`

---

### dataset_retriever_resources

**模型类**: `DatasetRetrieverResource`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `position` | Integer | sa.Integer | 必填 |
| `score` | Float (可空) | sa.Float | 可空 |
| `hit_count` | Integer (可空) | sa.Integer | 可空 |
| `word_count` | Integer (可空) | sa.Integer | 可空 |
| `segment_position` | Integer (可空) | sa.Integer | 可空 |
| `id` | Unknown | StringUUID | 必填, 自动生成UUID |
| `message_id` | Unknown | StringUUID | 必填 |
| `dataset_id` | Unknown | StringUUID | 必填 |
| `dataset_name` | Unknown | LongText | 必填 |
| `document_id` | Unknown | StringUUID | 可空 |
| `document_name` | Unknown | LongText | 必填 |
| `data_source_type` | Unknown | LongText | 可空 |
| `segment_id` | Unknown | StringUUID | 可空 |
| `content` | Unknown | LongText | 必填 |
| `index_node_hash` | Unknown | LongText | 可空 |
| `retriever_from` | Unknown | LongText | 必填 |
| `created_by` | Unknown | StringUUID | 必填 |
| `created_at` | Unknown | sa | 必填, 默认当前时间 |

#### 索引

- `dataset_retriever_resource_message_id_idx`

---

### dify_setups

**模型类**: `DifySetup`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `version` | String | String | 必填 |
| `setup_at` | DateTime | sa.DateTime | 必填, 默认当前时间 |

---

### end_users

**模型类**: `EndUser`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `tenant_id` | String | StringUUID | 必填 |
| `type` | String | String | 必填 |
| `_is_anonymous` | Boolean | "is_anonymous" | 必填, 默认值: sa.text("true" |
| `session_id` | String | String | 必填 |
| `app_id` | Unknown | StringUUID | 可空 |
| `external_user_id` | Unknown | String | 必填 |
| `name` | Unknown | String | 必填 |
| `created_at` | Unknown | sa | 必填, 默认当前时间 |
| `updated_at` | Unknown | sa | 必填, 默认当前时间 |

#### 索引

- `end_user_session_id_idx`
- `end_user_tenant_session_id_idx`

---

### installed_apps

**模型类**: `InstalledApp`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `tenant_id` | String | StringUUID | 必填 |
| `app_id` | String | StringUUID | 必填 |
| `app_owner_tenant_id` | String | StringUUID | 必填 |
| `position` | Integer | sa.Integer | 必填, 默认值: 0 |
| `is_pinned` | Boolean | sa.Boolean | 必填, 默认值: sa.text("false" |
| `last_used_at` | DateTime (可空) | sa.DateTime | 可空, 默认值: None |
| `created_at` | DateTime | sa.DateTime | 必填, 默认当前时间 |

#### 索引

- `installed_app_tenant_id_idx`
- `installed_app_app_id_idx`

---

### message_agent_thoughts

**模型类**: `MessageAgentThought`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `position` | Integer | sa.Integer | 必填 |
| `message_token` | Integer (可空) | sa.Integer | 可空 |
| `answer_token` | Integer (可空) | sa.Integer | 可空 |
| `tokens` | Integer (可空) | sa.Integer | 可空 |
| `latency` | Float (可空) | sa.Float | 可空 |
| `id` | Unknown | StringUUID | 必填, 自动生成UUID |
| `message_id` | Unknown | StringUUID | 必填 |
| `message_chain_id` | Unknown | StringUUID | 可空 |
| `thought` | Unknown | LongText | 可空 |
| `tool` | Unknown | LongText | 可空 |
| `tool_labels_str` | Unknown | LongText | 必填, 默认值: sa.text("'{}'" |
| `tool_meta_str` | Unknown | LongText | 必填, 默认值: sa.text("'{}'" |
| `tool_input` | Unknown | LongText | 可空 |
| `observation` | Unknown | LongText | 可空 |
| `plugin_id` | Unknown | StringUUID | 可空 |
| `tool_process_data` | Unknown | LongText | 可空 |
| `message` | Unknown | LongText | 可空 |
| `message_unit_price` | Unknown | sa | 可空 |
| `message_price_unit` | Unknown | sa | 必填 |
| `message_files` | Unknown | LongText | 可空 |
| `answer` | Unknown | LongText | 可空 |
| `answer_unit_price` | Unknown | sa | 可空 |
| `answer_price_unit` | Unknown | sa | 必填 |
| `total_price` | Unknown | sa | 可空 |
| `currency` | Unknown | String | 必填 |
| `created_by_role` | Unknown | String | 必填 |
| `created_by` | Unknown | StringUUID | 必填 |
| `created_at` | Unknown | sa | 必填, 默认当前时间 |

#### 索引

- `message_agent_thought_message_id_idx`
- `message_agent_thought_message_chain_id_idx`

---

### message_annotations

**模型类**: `MessageAnnotation`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `app_id` | String | StringUUID | 必填 |
| `conversation_id` | String (可空) | StringUUID | 可空 |
| `message_id` | String (可空) | StringUUID | 可空 |
| `hit_count` | Integer | sa.Integer | 必填, 默认值: sa.text("0" |
| `question` | Unknown | LongText | 可空 |
| `content` | Unknown | LongText | 必填 |
| `account_id` | Unknown | StringUUID | 必填 |
| `created_at` | Unknown | sa | 必填, 默认当前时间 |
| `updated_at` | Unknown | sa | 必填, 默认当前时间 |

#### 索引

- `message_annotation_app_idx`
- `message_annotation_conversation_idx`
- `message_annotation_message_idx`

---

### message_chains

**模型类**: `MessageChain`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `message_id` | String | StringUUID | 必填 |
| `type` | String | String | 必填 |
| `input` | String (可空) | LongText | 可空 |
| `output` | String (可空) | LongText | 可空 |
| `created_at` | DateTime | sa.DateTime | 必填, 默认当前时间 |

#### 索引

- `message_chain_message_id_idx`

---

### message_feedbacks

**模型类**: `MessageFeedback`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `app_id` | String | StringUUID | 必填 |
| `conversation_id` | String | StringUUID | 必填 |
| `message_id` | String | StringUUID | 必填 |
| `rating` | String | String | 必填 |
| `content` | String (可空) | LongText | 可空 |
| `from_source` | String | String | 必填 |
| `from_end_user_id` | String (可空) | StringUUID | 可空 |
| `from_account_id` | String (可空) | StringUUID | 可空 |
| `created_at` | DateTime | sa.DateTime | 必填, 默认当前时间 |
| `updated_at` | DateTime | sa.DateTime | 必填, 默认当前时间 |

#### 索引

- `message_feedback_app_idx`
- `message_feedback_message_idx`
- `message_feedback_conversation_idx`

---

### message_files

**模型类**: `MessageFile`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `message_id` | String | StringUUID | 必填 |
| `type` | String | String | 必填 |
| `transfer_method` | String | String | 必填 |
| `url` | String (可空) | LongText | 可空 |
| `belongs_to` | String (可空) | String | 可空 |
| `upload_file_id` | String (可空) | StringUUID | 可空 |
| `created_by_role` | String | String | 必填 |
| `created_by` | String | StringUUID | 必填 |
| `created_at` | DateTime | sa.DateTime | 必填, 默认当前时间 |

#### 索引

- `message_file_message_idx`
- `message_file_created_by_idx`

---

### messages

**模型类**: `Message`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `app_id` | String | StringUUID | 必填 |
| `model_provider` | String (可空) | String | 可空 |
| `model_id` | String (可空) | String | 可空 |
| `override_model_configs` | String (可空) | LongText | 可空 |
| `conversation_id` | String | StringUUID | 必填 |
| `query` | String | LongText | 必填 |
| `message_tokens` | Integer | sa.Integer | 必填, 默认值: sa.text("0" |
| `message_unit_price` | Decimal | sa.Numeric | 必填 |
| `message_price_unit` | Decimal | sa.Numeric | 必填 |
| `answer` | String | LongText | 必填 |
| `answer_tokens` | Integer | sa.Integer | 必填, 默认值: sa.text("0" |
| `answer_unit_price` | Decimal | sa.Numeric | 必填 |
| `answer_price_unit` | Decimal | sa.Numeric | 必填 |
| `parent_message_id` | String (可空) | StringUUID | 可空 |
| `provider_response_latency` | Float | sa.Float | 必填, 默认值: sa.text("0" |
| `total_price` | Decimal | None | sa.Numeric | 可空 |
| `currency` | String | String | 必填 |
| `status` | String | String | 必填 |
| `error` | String (可空) | LongText | 可空 |
| `message_metadata` | String (可空) | LongText | 可空 |
| `invoke_from` | String (可空) | String | 可空 |
| `from_source` | String | String | 必填 |
| `from_end_user_id` | String (可空) | StringUUID | 可空 |
| `from_account_id` | String (可空) | StringUUID | 可空 |
| `created_at` | DateTime | sa.DateTime | 必填, 默认当前时间 |
| `updated_at` | DateTime | sa.DateTime | 必填, 默认当前时间 |
| `agent_based` | Boolean | sa.Boolean | 必填, 默认值: sa.text("false" |
| `workflow_run_id` | String (可空) | StringUUID | 可空 |
| `app_mode` | String (可空) | String | 可空 |

---

### oauth_provider_apps

**模型类**: `OAuthProviderApp`

**功能**: Globally shared OAuth provider app information.
    Only for Dify Cloud.

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | Unknown | StringUUID | 必填, 默认值: lambda: str(uuidv7( |
| `app_icon` | Unknown | String | 必填 |
| `app_label` | Unknown | sa | 必填, 默认值: "{}" |
| `client_id` | Unknown | String | 必填 |
| `client_secret` | Unknown | String | 必填 |
| `redirect_uris` | Unknown | sa | 必填, 默认值: "[]" |
| `scope` | Unknown | String | 必填 |
| `created_at` | Unknown | sa | 必填, 默认当前时间 |

#### 索引

- `oauth_provider_app_client_id_idx`

---

### operation_logs

**模型类**: `OperationLog`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `action` | String | String | 必填 |
| `created_ip` | String | String | 必填 |
| `id` | Unknown | StringUUID | 必填, 自动生成UUID |
| `tenant_id` | Unknown | StringUUID | 必填 |
| `account_id` | Unknown | StringUUID | 必填 |
| `content` | Unknown | sa | 必填 |
| `created_at` | Unknown | sa | 必填, 默认当前时间 |
| `updated_at` | Unknown | sa | 必填, 默认当前时间 |

#### 索引

- `operation_log_account_action_idx`

---

### recommended_apps

**模型类**: `RecommendedApp`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `copyright` | String | String | 必填 |
| `privacy_policy` | String | String | 必填 |
| `custom_disclaimer` | String | LongText | 必填, 默认值: "" |
| `category` | String | String | 必填 |
| `position` | Integer | sa.Integer | 必填, 默认值: 0 |
| `is_listed` | Boolean | sa.Boolean | 必填, 默认值: True |
| `install_count` | Integer | sa.Integer | 必填, 默认值: 0 |
| `id` | Unknown | StringUUID | 必填, 自动生成UUID |
| `app_id` | Unknown | StringUUID | 必填 |
| `description` | Unknown | sa | 必填 |
| `language` | Unknown | String | 必填 |
| `created_at` | Unknown | sa | 必填, 默认当前时间 |
| `updated_at` | Unknown | sa | 必填, 默认当前时间 |

#### 索引

- `recommended_app_app_id_idx`
- `recommended_app_is_listed_idx`

---

### sites

**模型类**: `Site`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `title` | String | String | 必填 |
| `default_language` | String | String | 必填 |
| `chat_color_theme_inverted` | Boolean | sa.Boolean | 必填, 默认值: sa.text("false" |
| `show_workflow_steps` | Boolean | sa.Boolean | 必填, 默认值: sa.text("true" |
| `use_icon_as_answer_icon` | Boolean | sa.Boolean | 必填, 默认值: sa.text("false" |
| `_custom_disclaimer` | String | "custom_disclaimer" | 必填, 默认值: "" |
| `customize_token_strategy` | String | String | 必填 |
| `prompt_public` | Boolean | sa.Boolean | 必填, 默认值: sa.text("false" |
| `id` | Unknown | StringUUID | 必填, 自动生成UUID |
| `app_id` | Unknown | StringUUID | 必填 |
| `icon_type` | Unknown | String | 必填 |
| `icon` | Unknown | String | 必填 |
| `icon_background` | Unknown | String | 必填 |
| `description` | Unknown | LongText | 必填 |
| `chat_color_theme` | Unknown | String | 必填 |
| `copyright` | Unknown | String | 必填 |
| `privacy_policy` | Unknown | String | 必填 |
| `customize_domain` | Unknown | String | 必填 |
| `status` | Unknown | String | 必填 |
| `created_by` | Unknown | StringUUID | 可空 |
| `created_at` | Unknown | sa | 必填, 默认当前时间 |
| `updated_by` | Unknown | StringUUID | 可空 |
| `updated_at` | Unknown | sa | 必填, 默认当前时间 |
| `code` | Unknown | String | 必填 |

#### 索引

- `site_app_id_idx`
- `site_code_idx`

---

### tag_bindings

**模型类**: `TagBinding`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `tenant_id` | String (可空) | StringUUID | 可空 |
| `tag_id` | String (可空) | StringUUID | 可空 |
| `target_id` | String (可空) | StringUUID | 可空 |
| `created_by` | String | StringUUID | 必填 |
| `created_at` | DateTime | sa.DateTime | 必填, 默认当前时间 |

#### 索引

- `tag_bind_target_id_idx`
- `tag_bind_tag_id_idx`

---

### tags

**模型类**: `Tag`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `name` | String | String | 必填 |
| `id` | Unknown | StringUUID | 必填, 自动生成UUID |
| `tenant_id` | Unknown | StringUUID | 可空 |
| `type` | Unknown | String | 必填 |
| `created_by` | Unknown | StringUUID | 必填 |
| `created_at` | Unknown | sa | 必填, 默认当前时间 |

#### 索引

- `tag_type_idx`
- `tag_name_idx`

---

### trace_app_config

**模型类**: `TraceAppConfig`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `app_id` | String | StringUUID | 必填 |
| `tracing_provider` | String (可空) | String | 可空 |
| `tracing_config` | JSON (可空) | sa.JSON | 可空 |
| `created_at` | DateTime | sa.DateTime | 必填, 默认当前时间 |
| `updated_at` | DateTime | sa.DateTime | 必填, 默认当前时间 |
| `is_active` | Boolean | sa.Boolean | 必填, 默认值: sa.text("true" |

#### 索引

- `trace_app_config_app_id_idx`

---

### upload_files

**模型类**: `UploadFile`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `tenant_id` | String | StringUUID | 必填 |
| `storage_type` | String | String | 必填 |
| `key` | String | String | 必填 |
| `name` | String | String | 必填 |
| `size` | Integer | sa.Integer | 必填 |
| `extension` | String | String | 必填 |
| `mime_type` | String | String | 必填 |
| `created_by_role` | String | String | 必填 |
| `created_by` | String | StringUUID | 必填 |
| `created_at` | DateTime | sa.DateTime | 必填, 默认当前时间 |
| `used` | Boolean | sa.Boolean | 必填, 默认值: sa.text("false" |
| `used_by` | String (可空) | StringUUID | 可空 |
| `used_at` | DateTime (可空) | sa.DateTime | 可空 |
| `hash` | String (可空) | String | 可空 |
| `source_url` | String | LongText | 必填, 默认值: "" |

#### 索引

- `upload_file_tenant_idx`

---


## 数据集与知识库

**说明**: 管理数据集、文档、文档片段、嵌入向量、外部知识库等

**源文件**: `api/models/dataset.py`

### app_dataset_joins

**模型类**: `AppDatasetJoin`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `app_id` | String | StringUUID | 必填 |
| `dataset_id` | String | StringUUID | 必填 |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |

#### 索引

- `app_dataset_join_app_dataset_idx`

---

### child_chunks

**模型类**: `ChildChunk`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `position` | Integer | sa.Integer | 必填 |
| `word_count` | Integer | sa.Integer | 必填 |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `updated_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `indexing_at` | DateTime (可空) | DateTime | 可空 |
| `completed_at` | DateTime (可空) | DateTime | 可空 |
| `id` | Unknown | StringUUID | 必填, 自动生成UUID |
| `tenant_id` | Unknown | StringUUID | 必填 |
| `dataset_id` | Unknown | StringUUID | 必填 |
| `document_id` | Unknown | StringUUID | 必填 |
| `segment_id` | Unknown | StringUUID | 必填 |
| `content` | Unknown | LongText | 必填 |
| `index_node_id` | Unknown | String | 必填 |
| `index_node_hash` | Unknown | String | 必填 |
| `type` | Unknown | String | 必填 |
| `created_by` | Unknown | StringUUID | 必填 |
| `updated_by` | Unknown | StringUUID | 可空 |
| `error` | Unknown | LongText | 可空 |

#### 索引

- `child_chunk_dataset_id_idx`
- `child_chunks_node_idx`
- `child_chunks_segment_idx`

---

### dataset_auto_disable_logs

**模型类**: `DatasetAutoDisableLog`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `notified` | Boolean | sa.Boolean | 必填, 默认值: sa.text("false" |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `id` | Unknown | StringUUID | 必填, 自动生成UUID |
| `tenant_id` | Unknown | StringUUID | 必填 |
| `dataset_id` | Unknown | StringUUID | 必填 |
| `document_id` | Unknown | StringUUID | 必填 |

#### 索引

- `dataset_auto_disable_log_tenant_idx`
- `dataset_auto_disable_log_dataset_idx`
- `dataset_auto_disable_log_created_atx`

---

### dataset_collection_bindings

**模型类**: `DatasetCollectionBinding`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `provider_name` | String | String | 必填 |
| `model_name` | String | String | 必填 |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `id` | Unknown | StringUUID | 必填, 自动生成UUID |
| `type` | Unknown | String | 必填 |
| `collection_name` | Unknown | String | 必填 |

#### 索引

- `provider_model_name_idx`

---

### dataset_keyword_tables

**模型类**: `DatasetKeywordTable`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `dataset_id` | String | StringUUID | 必填 |
| `keyword_table` | String | LongText | 必填 |
| `data_source_type` | String | String | 必填 |

#### 索引

- `dataset_keyword_table_dataset_id_idx`

---

### dataset_metadata_bindings

**模型类**: `DatasetMetadataBinding`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `id` | Unknown | StringUUID | 必填, 自动生成UUID |
| `tenant_id` | Unknown | StringUUID | 必填 |
| `dataset_id` | Unknown | StringUUID | 必填 |
| `metadata_id` | Unknown | StringUUID | 必填 |
| `document_id` | Unknown | StringUUID | 必填 |
| `created_by` | Unknown | StringUUID | 必填 |

#### 索引

- `dataset_metadata_binding_tenant_idx`
- `dataset_metadata_binding_dataset_idx`
- `dataset_metadata_binding_metadata_idx`
- `dataset_metadata_binding_document_idx`

---

### dataset_metadatas

**模型类**: `DatasetMetadata`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `type` | String | String | 必填 |
| `name` | String | String | 必填 |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `updated_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `id` | Unknown | StringUUID | 必填, 自动生成UUID |
| `tenant_id` | Unknown | StringUUID | 必填 |
| `dataset_id` | Unknown | StringUUID | 必填 |
| `created_by` | Unknown | StringUUID | 必填 |
| `updated_by` | Unknown | StringUUID | 可空 |

#### 索引

- `dataset_metadata_tenant_idx`
- `dataset_metadata_dataset_idx`

---

### dataset_permissions

**模型类**: `DatasetPermission`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `dataset_id` | String | StringUUID | 必填 |
| `account_id` | String | StringUUID | 必填 |
| `tenant_id` | String | StringUUID | 必填 |
| `has_permission` | Boolean | sa.Boolean | 必填, 默认值: sa.text("true" |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |

#### 索引

- `idx_dataset_permissions_dataset_id`
- `idx_dataset_permissions_account_id`
- `idx_dataset_permissions_tenant_id`

---

### dataset_process_rules

**模型类**: `DatasetProcessRule`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `id` | Unknown | StringUUID | 必填, 自动生成UUID |
| `dataset_id` | Unknown | StringUUID | 必填 |
| `mode` | Unknown | String | 必填 |
| `rules` | Unknown | LongText | 可空 |
| `created_by` | Unknown | StringUUID | 必填 |

#### 索引

- `dataset_process_rule_dataset_id_idx`

---

### dataset_queries

**模型类**: `DatasetQuery`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `dataset_id` | String | StringUUID | 必填 |
| `content` | String | LongText | 必填 |
| `source` | String | String | 必填 |
| `source_app_id` | String (可空) | StringUUID | 可空 |
| `created_by_role` | String | String | 必填 |
| `created_by` | String | StringUUID | 必填 |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |

#### 索引

- `dataset_query_dataset_id_idx`

---

### datasets

**模型类**: `Dataset`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `tenant_id` | String | StringUUID | 必填 |
| `name` | String | String | 必填 |
| `provider` | String | String | 必填 |
| `permission` | String | String | 必填 |
| `indexing_technique` | String (可空) | String | 可空 |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `description` | Unknown | LongText | 可空 |
| `data_source_type` | Unknown | String | 必填 |
| `index_struct` | Unknown | LongText | 可空 |
| `created_by` | Unknown | StringUUID | 必填 |
| `updated_by` | Unknown | StringUUID | 可空 |
| `updated_at` | Unknown | sa | 必填, 默认当前时间 |
| `embedding_model` | Unknown | sa | 必填 |
| `embedding_model_provider` | Unknown | sa | 必填 |
| `keyword_number` | Unknown | sa | 可空, 默认值: sa.text("10" |
| `collection_binding_id` | Unknown | StringUUID | 可空 |
| `retrieval_model` | Unknown | AdjustedJSON | 可空 |
| `built_in_field_enabled` | Unknown | sa | 必填, 默认值: sa.text("false" |
| `icon_info` | Unknown | AdjustedJSON | 可空 |
| `runtime_mode` | Unknown | sa | 必填 |
| `pipeline_id` | Unknown | StringUUID | 可空 |
| `chunk_structure` | Unknown | sa | 必填 |
| `enable_api` | Unknown | sa | 必填, 默认值: sa.text("true" |

#### 索引

- `dataset_tenant_idx`

---

### document_pipeline_execution_logs

**模型类**: `DocumentPipelineExecutionLog`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 默认值: lambda: str(uuidv7( |
| `pipeline_id` | String | StringUUID | 必填 |
| `document_id` | String | StringUUID | 必填 |
| `datasource_type` | String | sa.String | 必填 |
| `datasource_info` | String | LongText | 必填 |
| `datasource_node_id` | String | sa.String | 必填 |
| `input_data` | JSON | sa.JSON | 必填 |
| `created_by` | String (可空) | StringUUID | 可空 |
| `created_at` | DateTime | sa.DateTime | 必填, 默认当前时间 |

#### 索引

- `document_pipeline_execution_logs_document_id_idx`

---

### document_segments

**模型类**: `DocumentSegment`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `hit_count` | Integer | sa.Integer | 必填, 默认值: 0 |
| `enabled` | Boolean | sa.Boolean | 必填, 默认值: sa.text("true" |
| `disabled_at` | DateTime (可空) | DateTime | 可空 |
| `status` | String | String | 必填 |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `updated_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `indexing_at` | DateTime (可空) | DateTime | 可空 |
| `completed_at` | DateTime (可空) | DateTime | 可空 |
| `stopped_at` | DateTime (可空) | DateTime | 可空 |
| `id` | Unknown | StringUUID | 必填, 自动生成UUID |
| `tenant_id` | Unknown | StringUUID | 必填 |
| `dataset_id` | Unknown | StringUUID | 必填 |
| `document_id` | Unknown | StringUUID | 必填 |
| `content` | Unknown | LongText | 必填 |
| `answer` | Unknown | LongText | 可空 |
| `keywords` | Unknown | sa | 可空 |
| `index_node_id` | Unknown | String | 必填 |
| `index_node_hash` | Unknown | String | 必填 |
| `disabled_by` | Unknown | StringUUID | 可空 |
| `created_by` | Unknown | StringUUID | 必填 |
| `updated_by` | Unknown | StringUUID | 可空 |
| `error` | Unknown | LongText | 可空 |

#### 索引

- `document_segment_dataset_id_idx`
- `document_segment_document_id_idx`
- `document_segment_tenant_dataset_idx`
- `document_segment_tenant_document_idx`
- `document_segment_node_dataset_idx`
- `document_segment_tenant_idx`

---

### documents

**模型类**: `Document`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `position` | Integer | sa.Integer | 必填 |
| `data_source_type` | String | String | 必填 |
| `batch` | String | String | 必填 |
| `name` | String | String | 必填 |
| `created_from` | String | String | 必填 |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `processing_started_at` | DateTime (可空) | DateTime | 可空 |
| `word_count` | Integer (可空) | sa.Integer | 可空 |
| `parsing_completed_at` | DateTime (可空) | DateTime | 可空 |
| `cleaning_completed_at` | DateTime (可空) | DateTime | 可空 |
| `splitting_completed_at` | DateTime (可空) | DateTime | 可空 |
| `tokens` | Integer (可空) | sa.Integer | 可空 |
| `indexing_latency` | Float (可空) | sa.Float | 可空 |
| `completed_at` | DateTime (可空) | DateTime | 可空 |
| `is_paused` | Boolean (可空) | sa.Boolean | 可空, 默认值: sa.text("false" |
| `paused_at` | DateTime (可空) | DateTime | 可空 |
| `stopped_at` | DateTime (可空) | DateTime | 可空 |
| `enabled` | Boolean | sa.Boolean | 必填, 默认值: sa.text("true" |
| `disabled_at` | DateTime (可空) | DateTime | 可空 |
| `archived` | Boolean | sa.Boolean | 必填, 默认值: sa.text("false" |
| `archived_at` | DateTime (可空) | DateTime | 可空 |
| `updated_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `id` | Unknown | StringUUID | 必填, 自动生成UUID |
| `tenant_id` | Unknown | StringUUID | 必填 |
| `dataset_id` | Unknown | StringUUID | 必填 |
| `data_source_info` | Unknown | LongText | 可空 |
| `dataset_process_rule_id` | Unknown | StringUUID | 可空 |
| `created_by` | Unknown | StringUUID | 必填 |
| `created_api_request_id` | Unknown | StringUUID | 可空 |
| `file_id` | Unknown | LongText | 可空 |
| `paused_by` | Unknown | StringUUID | 可空 |
| `error` | Unknown | LongText | 可空 |
| `indexing_status` | Unknown | String | 必填 |
| `disabled_by` | Unknown | StringUUID | 可空 |
| `archived_reason` | Unknown | String | 必填 |
| `archived_by` | Unknown | StringUUID | 可空 |
| `doc_type` | Unknown | String | 必填 |
| `doc_metadata` | Unknown | AdjustedJSON | 可空 |
| `doc_form` | Unknown | String | 必填 |
| `doc_language` | Unknown | String | 必填 |

#### 索引

- `document_dataset_id_idx`
- `document_is_paused_idx`
- `document_tenant_idx`

---

### embeddings

**模型类**: `Embedding`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `id` | Unknown | StringUUID | 必填, 自动生成UUID |
| `model_name` | Unknown | String | 必填 |
| `hash` | Unknown | String | 必填 |
| `embedding` | Unknown | BinaryData | 必填 |
| `provider_name` | Unknown | String | 必填 |

#### 索引

- `created_at_idx`

---

### external_knowledge_apis

**模型类**: `ExternalKnowledgeApis`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `name` | String | String | 必填 |
| `description` | String | String | 必填 |
| `tenant_id` | String | StringUUID | 必填 |
| `settings` | String (可空) | LongText | 可空 |
| `created_by` | String | StringUUID | 必填 |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `updated_by` | String (可空) | StringUUID | 可空 |
| `updated_at` | DateTime | DateTime | 必填, 默认当前时间 |

#### 索引

- `external_knowledge_apis_tenant_idx`
- `external_knowledge_apis_name_idx`

---

### external_knowledge_bindings

**模型类**: `ExternalKnowledgeBindings`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `tenant_id` | String | StringUUID | 必填 |
| `external_knowledge_api_id` | String | StringUUID | 必填 |
| `dataset_id` | String | StringUUID | 必填 |
| `external_knowledge_id` | String | String | 必填 |
| `created_by` | String | StringUUID | 必填 |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `updated_by` | String (可空) | StringUUID | 可空, 默认值: None |
| `updated_at` | DateTime | DateTime | 必填, 默认当前时间 |

#### 索引

- `external_knowledge_bindings_tenant_idx`
- `external_knowledge_bindings_dataset_idx`
- `external_knowledge_bindings_external_knowledge_idx`
- `external_knowledge_bindings_external_knowledge_api_idx`

---

### pipeline_built_in_templates

**模型类**: `PipelineBuiltInTemplate`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 默认值: lambda: str(uuidv7( |
| `name` | String | sa.String | 必填 |
| `description` | String | LongText | 必填 |
| `chunk_structure` | String | sa.String | 必填 |
| `icon` | JSON | sa.JSON | 必填 |
| `yaml_content` | String | LongText | 必填 |
| `copyright` | String | sa.String | 必填 |
| `privacy_policy` | String | sa.String | 必填 |
| `position` | Integer | sa.Integer | 必填 |
| `install_count` | Integer | sa.Integer | 必填 |
| `language` | String | sa.String | 必填 |
| `created_at` | DateTime | sa.DateTime | 必填, 默认当前时间 |
| `updated_at` | DateTime | sa.DateTime | 必填, 默认当前时间 |

---

### pipeline_customized_templates

**模型类**: `PipelineCustomizedTemplate`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 默认值: lambda: str(uuidv7( |
| `tenant_id` | String | StringUUID | 必填 |
| `name` | String | sa.String | 必填 |
| `description` | String | LongText | 必填 |
| `chunk_structure` | String | sa.String | 必填 |
| `icon` | JSON | sa.JSON | 必填 |
| `position` | Integer | sa.Integer | 必填 |
| `yaml_content` | String | LongText | 必填 |
| `install_count` | Integer | sa.Integer | 必填 |
| `language` | String | sa.String | 必填 |
| `created_by` | String | StringUUID | 必填 |
| `updated_by` | String (可空) | StringUUID | 可空, 默认值: None |
| `created_at` | DateTime | sa.DateTime | 必填, 默认当前时间 |
| `updated_at` | DateTime | sa.DateTime | 必填, 默认当前时间 |

#### 索引

- `pipeline_customized_template_tenant_idx`

---

### pipeline_recommended_plugins

**模型类**: `PipelineRecommendedPlugin`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 默认值: lambda: str(uuidv7( |
| `plugin_id` | String | LongText | 必填 |
| `provider_name` | String | LongText | 必填 |
| `position` | Integer | sa.Integer | 必填, 默认值: 0 |
| `active` | Boolean | sa.Boolean | 必填, 默认值: True |
| `created_at` | DateTime | sa.DateTime | 必填, 默认当前时间 |
| `updated_at` | DateTime | sa.DateTime | 必填, 默认当前时间 |

---

### pipelines

**模型类**: `Pipeline`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `tenant_id` | String | StringUUID | 必填 |
| `id` | Unknown | StringUUID | 必填, 默认值: lambda: str(uuidv7( |
| `name` | Unknown | sa | 必填 |
| `description` | Unknown | LongText | 必填, 默认值: sa.text("''" |
| `workflow_id` | Unknown | StringUUID | 可空 |
| `is_public` | Unknown | sa | 必填, 默认值: sa.text("false" |
| `is_published` | Unknown | sa | 必填, 默认值: sa.text("false" |
| `created_by` | Unknown | StringUUID | 可空 |
| `created_at` | Unknown | sa | 必填, 默认当前时间 |
| `updated_by` | Unknown | StringUUID | 可空 |
| `updated_at` | Unknown | sa | 必填, 默认当前时间 |

---

### rate_limit_logs

**模型类**: `RateLimitLog`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `tenant_id` | String | StringUUID | 必填 |
| `subscription_plan` | String | String | 必填 |
| `operation` | String | String | 必填 |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |

#### 索引

- `rate_limit_log_tenant_idx`
- `rate_limit_log_operation_idx`

---

### tidb_auth_bindings

**模型类**: `TidbAuthBinding`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `tenant_id` | String (可空) | StringUUID | 可空 |
| `cluster_id` | String | String | 必填 |
| `cluster_name` | String | String | 必填 |
| `active` | Boolean | sa.Boolean | 必填, 默认值: sa.text("false" |
| `status` | String | sa.String | 必填 |
| `account` | String | String | 必填 |
| `password` | String | String | 必填 |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |

#### 索引

- `tidb_auth_bindings_tenant_idx`
- `tidb_auth_bindings_active_idx`
- `tidb_auth_bindings_created_at_idx`
- `tidb_auth_bindings_status_idx`

---

### whitelists

**模型类**: `Whitelist`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `tenant_id` | String (可空) | StringUUID | 可空 |
| `category` | String | String | 必填 |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |

#### 索引

- `whitelists_tenant_idx`

---


## 工作流管理

**说明**: 管理工作流定义、工作流运行、节点执行记录等

**源文件**: `api/models/workflow.py`

### workflow_app_logs

**模型类**: `WorkflowAppLog`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `tenant_id` | String | StringUUID | 必填 |
| `app_id` | String | StringUUID | 必填 |
| `workflow_id` | String | StringUUID | 必填 |
| `workflow_run_id` | String | StringUUID | 必填 |
| `created_from` | String | String | 必填 |
| `created_by_role` | String | String | 必填 |
| `created_by` | String | StringUUID | 必填 |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |

#### 索引

- `workflow_app_log_app_idx`
- `workflow_app_log_workflow_run_id_idx`

---

### workflow_conversation_variables

**模型类**: `ConversationVariable`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填 |
| `conversation_id` | String | StringUUID | 必填 |
| `app_id` | String | StringUUID | 必填 |
| `data` | String | LongText | 必填 |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `updated_at` | DateTime | DateTime | 必填, 默认当前时间 |

---

### workflow_draft_variable_files

**模型类**: `WorkflowDraftVariableFile`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 默认值: lambda: str(uuidv7( |
| `created_at` | DateTime | DateTime | 必填, 默认值: naive_utc_now |
| `tenant_id` | String | StringUUID | 必填 |
| `app_id` | String | StringUUID | 必填 |
| `user_id` | String | StringUUID | 必填 |
| `upload_file_id` | String | StringUUID | 必填 |
| `size` | Integer (可空) | sa.BigInteger | 可空 |
| `length` | Integer (可空) | sa.Integer | 可空 |
| `value_type` | SegmentType | EnumText | 必填 |

---

### workflow_draft_variables

**模型类**: `WorkflowDraftVariable`

**功能**: `WorkflowDraftVariable` record variables and outputs generated during
    debugging workflow or chatflow.

    IMPORTANT: This model maintains multiple invariant rules that must be preserved.
    Do not instantiate this class directly with the constructor.

    Instead, use the factory methods (`new_conversation_variable`, `new_sys_variable`,
    `new_node_variable`) defined below to ensure all invariants are properly maintained.

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `created_at` | DateTime | DateTime | 必填, 默认值: naive_utc_now |
| `updated_at` | DateTime | DateTime | 必填, 默认值: naive_utc_now |
| `app_id` | String | StringUUID | 必填 |
| `last_edited_at` | DateTime (可空) | DateTime | 可空, 默认值: None |
| `node_id` | String | sa.String | 必填 |
| `name` | String | sa.String | 必填 |
| `description` | String | sa.String | 必填 |
| `selector` | String | sa.String | 必填 |
| `value_type` | SegmentType | EnumText | 必填 |
| `value` | String | LongText | 必填 |
| `visible` | Boolean | sa.Boolean | 必填, 默认值: True |
| `editable` | Boolean | sa.Boolean | 必填, 默认值: False |
| `node_execution_id` | String (可空) | StringUUID | 可空, 默认值: None |
| `file_id` | String (可空) | StringUUID | 可空, 默认值: None |
| `is_default_value` | Boolean | sa.Boolean | 必填, 默认值: False |

---

### workflow_node_execution_offload

**模型类**: `WorkflowNodeExecutionOffload`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `created_at` | DateTime | DateTime | 必填, 默认值: naive_utc_now |
| `tenant_id` | String | StringUUID | 必填 |
| `app_id` | String | StringUUID | 必填 |
| `node_execution_id` | String (可空) | StringUUID | 可空 |
| `type_` | ExecutionOffLoadType | EnumText | 必填 |
| `file_id` | String | StringUUID | 必填 |

---

### workflow_pauses

**模型类**: `WorkflowPause`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `workflow_id` | String | StringUUID | 必填 |
| `workflow_run_id` | String | StringUUID | 必填 |
| `resumed_at` | DateTime (可空) | sa.DateTime | 可空 |
| `state_object_key` | String | String | 必填 |

---

### workflows

**模型类**: `Workflow`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `tenant_id` | String | StringUUID | 必填 |
| `app_id` | String | StringUUID | 必填 |
| `type` | String | String | 必填 |
| `version` | String | String | 必填 |
| `marked_name` | String | String | 必填 |
| `marked_comment` | String | String | 必填 |
| `graph` | String | LongText | 必填 |
| `_features` | String | "features" | 必填 |
| `created_by` | String | StringUUID | 必填 |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `updated_by` | String (可空) | StringUUID | 可空 |
| `updated_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `_environment_variables` | String | "environment_variables" | 必填, 默认值: "{}" |
| `_conversation_variables` | String | "conversation_variables" | 必填, 默认值: "{}" |
| `_rag_pipeline_variables` | String | "rag_pipeline_variables" | 必填, 默认值: "{}" |

#### 索引

- `workflow_version_idx`

---


## 模型供应商管理

**说明**: 管理AI模型供应商、模型配置、凭证等

**源文件**: `api/models/provider.py`

### load_balancing_model_configs

**模型类**: `LoadBalancingModelConfig`

**功能**: Configurations for load balancing models.

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `tenant_id` | String | StringUUID | 必填 |
| `provider_name` | String | String | 必填 |
| `model_name` | String | String | 必填 |
| `model_type` | String | String | 必填 |
| `name` | String | String | 必填 |
| `encrypted_config` | String (可空) | LongText | 可空 |
| `credential_id` | String (可空) | StringUUID | 可空 |
| `credential_source_type` | String (可空) | String | 可空 |
| `enabled` | Boolean | sa.Boolean | 必填, 默认值: text("true" |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `updated_at` | DateTime | DateTime | 必填, 默认当前时间 |

#### 索引

- `load_balancing_model_config_tenant_provider_model_idx`

---

### provider_credentials

**模型类**: `ProviderCredential`

**功能**: Provider credential - stores multiple named credentials for each provider

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 默认值: lambda: str(uuidv7( |
| `tenant_id` | String | StringUUID | 必填 |
| `provider_name` | String | String | 必填 |
| `credential_name` | String | String | 必填 |
| `encrypted_config` | String | LongText | 必填 |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `updated_at` | DateTime | DateTime | 必填, 默认当前时间 |

#### 索引

- `provider_credential_tenant_provider_idx`

---

### provider_model_credentials

**模型类**: `ProviderModelCredential`

**功能**: Provider model credential - stores multiple named credentials for each provider model

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 默认值: lambda: str(uuidv7( |
| `tenant_id` | String | StringUUID | 必填 |
| `provider_name` | String | String | 必填 |
| `model_name` | String | String | 必填 |
| `model_type` | String | String | 必填 |
| `credential_name` | String | String | 必填 |
| `encrypted_config` | String | LongText | 必填 |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `updated_at` | DateTime | DateTime | 必填, 默认当前时间 |

---

### provider_model_settings

**模型类**: `ProviderModelSetting`

**功能**: Provider model settings for record the model enabled status and load balancing status.

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `tenant_id` | String | StringUUID | 必填 |
| `provider_name` | String | String | 必填 |
| `model_name` | String | String | 必填 |
| `model_type` | String | String | 必填 |
| `enabled` | Boolean | sa.Boolean | 必填, 默认值: text("true" |
| `load_balancing_enabled` | Boolean | sa.Boolean | 必填, 默认值: text("false" |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `updated_at` | DateTime | DateTime | 必填, 默认当前时间 |

#### 索引

- `provider_model_setting_tenant_provider_model_idx`

---

### provider_models

**模型类**: `ProviderModel`

**功能**: Provider model representing the API provider_models and their configurations.

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `tenant_id` | String | StringUUID | 必填 |
| `provider_name` | String | String | 必填 |
| `model_name` | String | String | 必填 |
| `model_type` | String | String | 必填 |
| `credential_id` | String (可空) | StringUUID | 可空, 默认值: None |
| `is_valid` | Boolean | sa.Boolean | 必填, 默认值: text("false" |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `updated_at` | DateTime | DateTime | 必填, 默认当前时间 |

#### 索引

- `provider_model_tenant_id_provider_idx`

---

### provider_orders

**模型类**: `ProviderOrder`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `tenant_id` | String | StringUUID | 必填 |
| `provider_name` | String | String | 必填 |
| `account_id` | String | StringUUID | 必填 |
| `payment_product_id` | String | String | 必填 |
| `payment_id` | String (可空) | String | 可空 |
| `transaction_id` | String (可空) | String | 可空 |
| `quantity` | Integer | sa.Integer | 必填, 默认值: text("1" |
| `currency` | String (可空) | String | 可空 |
| `total_amount` | Integer (可空) | sa.Integer | 可空 |
| `payment_status` | String | String | 必填 |
| `paid_at` | DateTime (可空) | DateTime | 可空 |
| `pay_failed_at` | DateTime (可空) | DateTime | 可空 |
| `refunded_at` | DateTime (可空) | DateTime | 可空 |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `updated_at` | DateTime | DateTime | 必填, 默认当前时间 |

#### 索引

- `provider_order_tenant_provider_idx`

---

### providers

**模型类**: `Provider`

**功能**: Provider model representing the API providers and their configurations.

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 默认值: lambda: str(uuidv7( |
| `tenant_id` | String | StringUUID | 必填 |
| `provider_name` | String | String | 必填 |
| `provider_type` | String | String | 必填 |
| `is_valid` | Boolean | sa.Boolean | 必填, 默认值: text("false" |
| `last_used` | DateTime (可空) | DateTime | 可空 |
| `credential_id` | String (可空) | StringUUID | 可空, 默认值: None |
| `quota_type` | String (可空) | String | 可空 |
| `quota_limit` | Integer (可空) | sa.BigInteger | 可空, 默认值: None |
| `quota_used` | Integer | sa.BigInteger | 必填, 默认值: 0 |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `updated_at` | DateTime | DateTime | 必填, 默认当前时间 |

#### 索引

- `provider_tenant_id_provider_idx`

---

### tenant_default_models

**模型类**: `TenantDefaultModel`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `tenant_id` | String | StringUUID | 必填 |
| `provider_name` | String | String | 必填 |
| `model_name` | String | String | 必填 |
| `model_type` | String | String | 必填 |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `updated_at` | DateTime | DateTime | 必填, 默认当前时间 |

#### 索引

- `tenant_default_model_tenant_id_provider_type_idx`

---

### tenant_preferred_model_providers

**模型类**: `TenantPreferredModelProvider`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `tenant_id` | String | StringUUID | 必填 |
| `provider_name` | String | String | 必填 |
| `preferred_provider_type` | String | String | 必填 |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `updated_at` | DateTime | DateTime | 必填, 默认当前时间 |

#### 索引

- `tenant_preferred_model_provider_tenant_provider_idx`

---


## 工具管理

**说明**: 管理各类工具提供商、工具调用记录、工具文件等

**源文件**: `api/models/tools.py`

### tool_api_providers

**模型类**: `ApiToolProvider`

**功能**: The table stores the api providers.

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `name` | String | String | 必填 |
| `icon` | String | String | 必填 |
| `schema` | String | LongText | 必填 |
| `schema_type_str` | String | String | 必填 |
| `user_id` | String | StringUUID | 必填 |
| `tenant_id` | String | StringUUID | 必填 |
| `description` | String | LongText | 必填 |
| `tools_str` | String | LongText | 必填 |
| `credentials_str` | String | LongText | 必填 |
| `privacy_policy` | String (可空) | String | 可空 |
| `custom_disclaimer` | String | LongText | 必填, 默认值: "" |
| `created_at` | DateTime | sa.DateTime | 必填, 默认当前时间 |
| `updated_at` | DateTime | sa.DateTime | 必填, 默认当前时间 |

---

### tool_builtin_providers

**模型类**: `BuiltinToolProvider`

**功能**: This table stores the tool provider information for built-in tools for each tenant.

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `name` | String | String | 必填 |
| `tenant_id` | String (可空) | StringUUID | 可空 |
| `user_id` | String | StringUUID | 必填 |
| `provider` | String | String | 必填 |
| `encrypted_credentials` | String (可空) | LongText | 可空, 默认值: None |
| `created_at` | DateTime | sa.DateTime | 必填, 默认当前时间 |
| `updated_at` | DateTime | sa.DateTime | 必填, 默认当前时间 |
| `is_default` | Boolean | sa.Boolean | 必填, 默认值: sa.text("false" |
| `credential_type` | String | String | 必填 |
| `expires_at` | Integer | sa.BigInteger | 必填, 默认值: sa.text("-1" |

---

### tool_conversation_variables

**模型类**: `ToolConversationVariables`

**功能**: store the conversation variables from tool invoke

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `user_id` | String | StringUUID | 必填 |
| `tenant_id` | String | StringUUID | 必填 |
| `conversation_id` | String | StringUUID | 必填 |
| `variables_str` | String | LongText | 必填 |
| `created_at` | DateTime | sa.DateTime | 必填, 默认当前时间 |
| `updated_at` | DateTime | sa.DateTime | 必填, 默认当前时间 |

#### 索引

- `user_id_idx`
- `conversation_id_idx`

---

### tool_files

**模型类**: `ToolFile`

**功能**: This table stores file metadata generated in workflows,
    not only files created by agent.

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `user_id` | String | StringUUID | 必填 |
| `tenant_id` | String | StringUUID | 必填 |
| `conversation_id` | String (可空) | StringUUID | 可空 |
| `file_key` | String | String | 必填 |
| `mimetype` | String | String | 必填 |
| `original_url` | String (可空) | String | 可空 |
| `name` | String | String | 必填 |
| `size` | Integer | sa.Integer | 必填, 默认值: -1 |

#### 索引

- `tool_file_conversation_id_idx`

---

### tool_label_bindings

**模型类**: `ToolLabelBinding`

**功能**: The table stores the labels for tools.

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `tool_id` | String | String | 必填 |
| `tool_type` | String | String | 必填 |
| `label_name` | String | String | 必填 |

---

### tool_mcp_providers

**模型类**: `MCPToolProvider`

**功能**: The table stores the mcp providers.

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `name` | String | String | 必填 |
| `server_identifier` | String | String | 必填 |
| `server_url` | String | LongText | 必填 |
| `server_url_hash` | String | String | 必填 |
| `icon` | String (可空) | String | 可空 |
| `tenant_id` | String | StringUUID | 必填 |
| `user_id` | String | StringUUID | 必填 |
| `encrypted_credentials` | String (可空) | LongText | 可空, 默认值: None |
| `authed` | Boolean | sa.Boolean | 必填, 默认值: False |
| `tools` | String | LongText | 必填, 默认值: "[]" |
| `created_at` | DateTime | sa.DateTime | 必填, 默认当前时间 |
| `updated_at` | DateTime | sa.DateTime | 必填, 默认当前时间 |
| `timeout` | Float | sa.Float | 必填, 默认值: sa.text("30" |
| `sse_read_timeout` | Float | sa.Float | 必填, 默认值: sa.text("300" |
| `encrypted_headers` | String (可空) | LongText | 可空, 默认值: None |

---

### tool_model_invokes

**模型类**: `ToolModelInvoke`

**功能**: store the invoke logs from tool invoke

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `user_id` | String | StringUUID | 必填 |
| `tenant_id` | String | StringUUID | 必填 |
| `provider` | String | String | 必填 |
| `tool_type` | String | String | 必填 |
| `tool_name` | String | String | 必填 |
| `model_parameters` | String | LongText | 必填 |
| `prompt_messages` | String | LongText | 必填 |
| `model_response` | String | LongText | 必填 |
| `prompt_tokens` | Integer | sa.Integer | 必填, 默认值: sa.text("0" |
| `answer_tokens` | Integer | sa.Integer | 必填, 默认值: sa.text("0" |
| `answer_unit_price` | Decimal | sa.Numeric | 必填 |
| `answer_price_unit` | Decimal | sa.Numeric | 必填 |
| `provider_response_latency` | Float | sa.Float | 必填, 默认值: sa.text("0" |
| `total_price` | Decimal | None | sa.Numeric | 可空 |
| `currency` | String | String | 必填 |
| `created_at` | DateTime | sa.DateTime | 必填, 默认当前时间 |
| `updated_at` | DateTime | sa.DateTime | 必填, 默认当前时间 |

---

### tool_oauth_system_clients

**模型类**: `ToolOAuthSystemClient`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `plugin_id` | String | String | 必填 |
| `provider` | String | String | 必填 |
| `encrypted_oauth_params` | String | LongText | 必填 |

---

### tool_oauth_tenant_clients

**模型类**: `ToolOAuthTenantClient`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `tenant_id` | String | StringUUID | 必填 |
| `plugin_id` | String | String | 必填 |
| `provider` | String | String | 必填 |
| `enabled` | Boolean | sa.Boolean | 必填, 默认值: sa.text("true" |
| `encrypted_oauth_params` | String | LongText | 必填 |

---

### tool_published_apps

**模型类**: `DeprecatedPublishedAppTool`

**功能**: The table stores the apps published as a tool for each person.

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `app_id` | String | StringUUID | 必填 |
| `user_id` | String | StringUUID | 必填 |
| `description` | String | LongText | 必填 |
| `llm_description` | String | LongText | 必填 |
| `query_description` | String | LongText | 必填 |
| `query_name` | String | String | 必填 |
| `tool_name` | String | String | 必填 |
| `author` | String | String | 必填 |
| `created_at` | DateTime | sa.DateTime | 必填, 默认当前时间 |
| `updated_at` | DateTime | sa.DateTime | 必填, 默认当前时间 |

---

### tool_workflow_providers

**模型类**: `WorkflowToolProvider`

**功能**: The table stores the workflow providers.

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `name` | String | String | 必填 |
| `label` | String | String | 必填 |
| `icon` | String | String | 必填 |
| `app_id` | String | StringUUID | 必填 |
| `version` | String | String | 必填 |
| `user_id` | String | StringUUID | 必填 |
| `tenant_id` | String | StringUUID | 必填 |
| `description` | String | LongText | 必填 |
| `parameter_configuration` | String | LongText | 必填, 默认值: "[]" |
| `privacy_policy` | String (可空) | String | 可空 |
| `created_at` | DateTime | sa.DateTime | 必填, 默认当前时间 |
| `updated_at` | DateTime | sa.DateTime | 必填, 默认当前时间 |

---


## 触发器管理

**说明**: 管理触发器订阅、Webhook、定时任务等

**源文件**: `api/models/trigger.py`

### app_triggers

**模型类**: `AppTrigger`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 默认值: lambda: str(uuidv7( |
| `tenant_id` | String | StringUUID | 必填 |
| `app_id` | String | StringUUID | 必填 |
| `node_id` | String (可空) | String | 可空 |
| `trigger_type` | String | EnumText | 必填 |
| `title` | String | String | 必填 |
| `provider_name` | String | String | 必填 |
| `status` | String | EnumText | 必填 |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `updated_at` | DateTime | DateTime | 必填, 默认值: naive_utc_now( |

#### 索引

- `app_trigger_tenant_app_idx`

---

### trigger_oauth_system_clients

**模型类**: `TriggerOAuthSystemClient`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `plugin_id` | String | String | 必填 |
| `provider` | String | String | 必填 |
| `encrypted_oauth_params` | String | LongText | 必填 |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `updated_at` | DateTime | DateTime | 必填, 默认当前时间 |

---

### trigger_oauth_tenant_clients

**模型类**: `TriggerOAuthTenantClient`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `tenant_id` | String | StringUUID | 必填 |
| `plugin_id` | String | String | 必填 |
| `provider` | String | String | 必填 |
| `enabled` | Boolean | sa.Boolean | 必填, 默认值: sa.text("true" |
| `encrypted_oauth_params` | String | LongText | 必填 |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `updated_at` | DateTime | DateTime | 必填, 默认当前时间 |

---

### trigger_subscriptions

**模型类**: `TriggerSubscription`

**功能**: Trigger provider model for managing credentials
    Supports multiple credential instances per provider

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `name` | String | String | 必填 |
| `tenant_id` | String | StringUUID | 必填 |
| `user_id` | String | StringUUID | 必填 |
| `provider_id` | String | String | 必填 |
| `endpoint_id` | String | String | 必填 |
| `credential_type` | String | String | 必填 |
| `credential_expires_at` | Integer | Integer | 必填, 默认值: -1 |
| `expires_at` | Integer | Integer | 必填, 默认值: -1 |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `updated_at` | DateTime | DateTime | 必填, 默认当前时间 |

---

### workflow_plugin_triggers

**模型类**: `WorkflowPluginTrigger`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `app_id` | String | StringUUID | 必填 |
| `node_id` | String | String | 必填 |
| `tenant_id` | String | StringUUID | 必填 |
| `provider_id` | String | String | 必填 |
| `event_name` | String | String | 必填 |
| `subscription_id` | String | String | 必填 |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `updated_at` | DateTime | DateTime | 必填, 默认当前时间 |

#### 索引

- `workflow_plugin_trigger_tenant_subscription_idx`

---

### workflow_schedule_plans

**模型类**: `WorkflowSchedulePlan`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 默认值: lambda: str(uuidv7( |
| `app_id` | String | StringUUID | 必填 |
| `node_id` | String | String | 必填 |
| `tenant_id` | String | StringUUID | 必填 |
| `cron_expression` | String | String | 必填 |
| `timezone` | String | String | 必填 |
| `next_run_at` | DateTime (可空) | DateTime | 可空 |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `updated_at` | DateTime | DateTime | 必填, 默认当前时间 |

#### 索引

- `workflow_schedule_plan_next_idx`

---

### workflow_webhook_triggers

**模型类**: `WorkflowWebhookTrigger`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 默认值: lambda: str(uuidv7( |
| `app_id` | String | StringUUID | 必填 |
| `node_id` | String | String | 必填 |
| `tenant_id` | String | StringUUID | 必填 |
| `webhook_id` | String | String | 必填 |
| `created_by` | String | StringUUID | 必填 |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `updated_at` | DateTime | DateTime | 必填, 默认当前时间 |

#### 索引

- `workflow_webhook_trigger_tenant_idx`

---


## Web功能

**说明**: 管理保存的消息、固定的对话等Web相关功能

**源文件**: `api/models/web.py`

### pinned_conversations

**模型类**: `PinnedConversation`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `app_id` | String | StringUUID | 必填 |
| `conversation_id` | String | StringUUID | 必填 |
| `created_by_role` | String | String | 必填 |
| `created_by` | String | StringUUID | 必填 |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |

#### 索引

- `pinned_conversation_conversation_idx`

---

### saved_messages

**模型类**: `SavedMessage`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `app_id` | String | StringUUID | 必填 |
| `message_id` | String | StringUUID | 必填 |
| `created_by_role` | String | String | 必填 |
| `created_by` | String | StringUUID | 必填 |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |

#### 索引

- `saved_message_message_idx`

---


## OAuth认证

**说明**: 管理数据源OAuth配置

**源文件**: `api/models/oauth.py`

### datasource_oauth_params

**模型类**: `DatasourceOauthParamConfig`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 默认值: lambda: str(uuidv7( |
| `plugin_id` | String | sa.String | 必填 |
| `provider` | String | sa.String | 必填 |
| `system_credentials` | JSON | AdjustedJSON | 必填 |

---

### datasource_oauth_tenant_params

**模型类**: `DatasourceOauthTenantParamConfig`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 默认值: lambda: str(uuidv7( |
| `tenant_id` | String | StringUUID | 必填 |
| `provider` | String | sa.String | 必填 |
| `plugin_id` | String | sa.String | 必填 |
| `client_params` | JSON | AdjustedJSON | 必填 |
| `enabled` | Boolean | sa.Boolean | 必填, 默认值: False |
| `created_at` | DateTime | sa.DateTime | 必填, 默认当前时间 |
| `updated_at` | DateTime | sa.DateTime | 必填, 默认当前时间 |

---

### datasource_providers

**模型类**: `DatasourceProvider`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 默认值: lambda: str(uuidv7( |
| `tenant_id` | String | StringUUID | 必填 |
| `name` | String | sa.String | 必填 |
| `provider` | String | sa.String | 必填 |
| `plugin_id` | String | sa.String | 必填 |
| `auth_type` | String | sa.String | 必填 |
| `encrypted_credentials` | JSON | AdjustedJSON | 必填 |
| `avatar_url` | String | LongText | 可空, 默认值: "default" |
| `is_default` | Boolean | sa.Boolean | 必填, 默认值: sa.text("false" |
| `expires_at` | Integer | sa.Integer | 必填, 默认值: "-1" |
| `created_at` | DateTime | sa.DateTime | 必填, 默认当前时间 |
| `updated_at` | DateTime | sa.DateTime | 必填, 默认当前时间 |

#### 索引

- `datasource_provider_auth_type_provider_idx`

---


## 数据源认证

**说明**: 管理数据源OAuth和API Key认证绑定

**源文件**: `api/models/source.py`

### data_source_api_key_auth_bindings

**模型类**: `DataSourceApiKeyAuthBinding`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `tenant_id` | String | StringUUID | 必填 |
| `category` | String | String | 必填 |
| `provider` | String | String | 必填 |
| `credentials` | String (可空) | LongText | 可空, 默认值: None |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `updated_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `disabled` | Boolean | sa.Boolean | 可空, 默认值: sa.text("false" |

#### 索引

- `data_source_api_key_auth_binding_tenant_id_idx`
- `data_source_api_key_auth_binding_provider_idx`

---

### data_source_oauth_bindings

**模型类**: `DataSourceOauthBinding`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `tenant_id` | String | StringUUID | 必填 |
| `access_token` | String | String | 必填 |
| `provider` | String | String | 必填 |
| `source_info` | JSON | AdjustedJSON | 必填 |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `updated_at` | DateTime | DateTime | 必填, 默认当前时间 |
| `disabled` | Boolean | sa.Boolean | 可空, 默认值: sa.text("false" |

#### 索引

- `source_binding_tenant_id_idx`

---


## 异步任务

**说明**: 管理Celery异步任务

**源文件**: `api/models/task.py`

### celery_taskmeta

**模型类**: `CeleryTask`

**功能**: Task result/status.

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | Integer | sa.Integer | 必填 |
| `task_id` | String | String | 必填 |
| `status` | String | String | 必填 |
| `result` | bytes | None | BinaryData | 可空, 默认值: None |
| `date_done` | DateTime (可空) | DateTime | 可空, 默认值: naive_utc_now |
| `traceback` | String (可空) | LongText | 可空, 默认值: None |
| `name` | String (可空) | String | 可空 |
| `args` | bytes | None | BinaryData | 可空, 默认值: None |
| `kwargs` | bytes | None | BinaryData | 可空, 默认值: None |
| `worker` | String (可空) | String | 可空 |
| `retries` | Integer (可空) | sa.Integer | 可空, 默认值: None |
| `queue` | String (可空) | String | 可空 |

---

### celery_tasksetmeta

**模型类**: `CeleryTaskSet`

**功能**: TaskSet result.

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | Integer | sa.Integer | 必填 |
| `taskset_id` | String | String | 必填 |
| `result` | bytes | None | BinaryData | 可空, 默认值: None |
| `date_done` | DateTime (可空) | DateTime | 可空, 默认值: naive_utc_now |

---


## API扩展

**说明**: 管理基于API的扩展点

**源文件**: `api/models/api_based_extension.py`

### api_based_extensions

**模型类**: `APIBasedExtension`

#### 字段说明

| 字段名 | 类型 | SQL类型 | 说明 |
|--------|------|---------|------|
| `id` | String | StringUUID | 必填, 自动生成UUID |
| `tenant_id` | String | StringUUID | 必填 |
| `name` | String | String | 必填 |
| `api_endpoint` | String | String | 必填 |
| `api_key` | String | LongText | 必填 |
| `created_at` | DateTime | DateTime | 必填, 默认当前时间 |

#### 索引

- `api_based_extension_tenant_idx`

---


## 附录

### 常见数据类型说明

| 类型 | 说明 |
|------|------|
| StringUUID | UUID 字符串类型，用于主键和外键 |
| String(N) | 可变长度字符串，最大长度 N |
| LongText | 长文本类型，用于存储大段文本 |
| DateTime | 日期时间类型 |
| Integer | 整数类型 |
| Boolean | 布尔类型 (true/false) |
| AdjustedJSON | JSON 类型，用于存储结构化数据 |
| BinaryData | 二进制数据类型 |
| EnumText | 枚举文本类型 |

### 通用字段约定

大多数表都包含以下通用字段：

- `id`: 主键，UUID 类型
- `created_at`: 创建时间，自动设置为当前时间戳
- `updated_at`: 更新时间，自动更新为当前时间戳
- `tenant_id`: 租户ID，用于多租户数据隔离
- `created_by`: 创建者ID
- `updated_by`: 更新者ID

### 关系说明

#### 核心关系
- **Account (账户)** ←→ **Tenant (租户)**: 多对多关系，通过 `tenant_account_joins` 关联
- **Tenant (租户)** ←→ **App (应用)**: 一对多关系
- **App (应用)** ←→ **Conversation (对话)**: 一对多关系
- **Conversation (对话)** ←→ **Message (消息)**: 一对多关系
- **Dataset (数据集)** ←→ **Document (文档)**: 一对多关系
- **Document (文档)** ←→ **DocumentSegment (文档片段)**: 一对多关系
- **App (应用)** ←→ **Workflow (工作流)**: 一对一关系

### 数据模型设计原则

1. **多租户隔离**: 通过 `tenant_id` 字段实现数据隔离
2. **软删除**: 部分表使用 `archived` 字段实现软删除
3. **审计追踪**: 记录创建者、创建时间、更新者、更新时间
4. **版本控制**: 工作流和应用配置支持版本管理
5. **状态管理**: 使用枚举类型管理实体状态
6. **加密存储**: 敏感信息（如凭证）使用加密存储

### 索引策略

- 主键索引：所有表的 `id` 字段
- 外键索引：关联字段如 `tenant_id`, `app_id`, `user_id` 等
- 查询优化索引：频繁查询字段如 `email`, `name` 等
- 复合索引：多字段组合查询场景

---

**文档版本**: 1.0  
**生成时间**: 2025-11-23  
**适用版本**: Dify Latest

