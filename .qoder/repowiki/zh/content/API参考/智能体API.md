# 智能体API

<cite>
**本文档中引用的文件**  
- [agent.py](file://libs/agno/agno/agent/agent.py)
- [router.py](file://libs/agno/agno/os/router.py)
- [schema.py](file://libs/agno/agno/os/schema.py)
</cite>

## 目录
1. [简介](#简介)
2. [智能体生命周期管理端点](#智能体生命周期管理端点)
3. [核心参数说明](#核心参数说明)
4. [API调用示例](#api调用示例)
5. [错误处理](#错误处理)

## 简介
智能体API提供了一套完整的RESTful接口，用于管理和操作智能体系统。该API支持智能体的创建、查询、更新和删除等全生命周期管理操作。API基于FastAPI框架构建，提供了详细的OpenAPI文档和示例，便于开发者快速集成和使用。所有端点都实现了标准的HTTP响应码和错误处理机制，确保了接口的可靠性和一致性。

## 智能体生命周期管理端点

### 创建智能体 (POST /agents)
创建一个新的智能体实例。

**HTTP方法**: POST  
**URL路径**: `/agents`  
**请求头**:  
- `Authorization: Bearer <token>`  
- `Content-Type: application/json`

**请求体JSON Schema (AgentCreateSchema)**:
```json
{
  "id": "string",
  "name": "string",
  "model": {
    "name": "string",
    "model": "string",
    "provider": "string"
  },
  "tools": {},
  "sessions": {},
  "knowledge": {},
  "memory": {},
  "reasoning": {},
  "system_message": {}
}
```

**响应体Schema (AgentResponseSchema)**:
```json
{
  "id": "string",
  "name": "string",
  "db_id": "string",
  "model": {
    "name": "string",
    "model": "string",
    "provider": "string"
  },
  "tools": {},
  "sessions": {},
  "knowledge": {},
  "memory": {},
  "reasoning": {},
  "system_message": {}
}
```

**字段说明**:
- `id`: 智能体的唯一标识符
- `name`: 智能体的名称
- `model`: 模型配置，包括模型名称、ID和提供商
- `tools`: 工具集配置
- `sessions`: 会话管理配置
- `knowledge`: 知识库配置
- `memory`: 记忆管理配置
- `reasoning`: 推理能力配置
- `system_message`: 系统消息配置

**Section sources**
- [router.py](file://libs/agno/agno/os/router.py#L887-L915)
- [schema.py](file://libs/agno/agno/os/schema.py#L200-L300)

### 获取智能体列表 (GET /agents)
获取所有已配置的智能体列表。

**HTTP方法**: GET  
**URL路径**: `/agents`  
**请求头**:  
- `Authorization: Bearer <token>`

**响应体Schema (List[AgentResponse])**:
```json
[
  {
    "id": "string",
    "name": "string",
    "db_id": "string",
    "model": {
      "name": "string",
      "model": "string",
      "provider": "string"
    },
    "tools": null,
    "sessions": {
      "session_table": "string"
    },
    "knowledge": {
      "knowledge_table": "string"
    },
    "system_message": {
      "markdown": true,
      "add_datetime_to_context": true
    }
  }
]
```

**字段说明**:
- 返回一个包含所有智能体信息的数组
- 每个智能体包含其元数据、模型配置、工具集、会话、知识、记忆和推理设置

**Section sources**
- [router.py](file://libs/agno/agno/os/router.py#L887-L915)
- [schema.py](file://libs/agno/agno/os/schema.py#L200-L300)

### 获取单个智能体详情 (GET /agents/{agent_id})
获取特定智能体的详细配置信息。

**HTTP方法**: GET  
**URL路径**: `/agents/{agent_id}`  
**请求头**:  
- `Authorization: Bearer <token>`

**路径参数**:
- `agent_id`: 智能体的唯一标识符

**响应体Schema (AgentResponse)**:
```json
{
  "id": "string",
  "name": "string",
  "db_id": "string",
  "model": {
    "name": "string",
    "model": "string",
    "provider": "string"
  },
  "tools": null,
  "sessions": {
    "session_table": "string"
  },
  "knowledge": {
    "knowledge_table": "string"
  },
  "system_message": {
    "markdown": true,
    "add_datetime_to_context": true
  }
}
```

**字段说明**:
- 返回指定智能体的完整配置信息
- 包含模型配置、工具、会话、知识、记忆、推理能力等详细信息

**Section sources**
- [router.py](file://libs/agno/agno/os/router.py#L963-L981)
- [schema.py](file://libs/agno/agno/os/schema.py#L200-L300)

### 更新智能体配置 (PUT /agents/{agent_id})
更新特定智能体的配置。

**HTTP方法**: PUT  
**URL路径**: `/agents/{agent_id}`  
**请求头**:  
- `Authorization: Bearer <token>`  
- `Content-Type: application/json`

**路径参数**:
- `agent_id`: 智能体的唯一标识符

**请求体JSON Schema**:
```json
{
  "name": "string",
  "model": {
    "name": "string",
    "model": "string",
    "provider": "string"
  },
  "tools": {},
  "sessions": {},
  "knowledge": {},
  "memory": {},
  "reasoning": {},
  "system_message": {}
}
```

**响应体Schema (AgentResponse)**:
```json
{
  "id": "string",
  "name": "string",
  "db_id": "string",
  "model": {
    "name": "string",
    "model": "string",
    "provider": "string"
  },
  "tools": {},
  "sessions": {},
  "knowledge": {},
  "memory": {},
  "reasoning": {},
  "system_message": {}
}
```

**字段说明**:
- 更新指定智能体的配置
- 可以更新名称、模型、工具集、会话、知识、记忆、推理和系统消息等配置

**Section sources**
- [router.py](file://libs/agno/agno/os/router.py#L963-L981)
- [schema.py](file://libs/agno/agno/os/schema.py#L200-L300)

### 删除智能体 (DELETE /agents/{agent_id})
删除特定智能体。

**HTTP方法**: DELETE  
**URL路径**: `/agents/{agent_id}`  
**请求头**:  
- `Authorization: Bearer <token>`

**路径参数**:
- `agent_id`: 智能体的唯一标识符

**响应状态码**:
- `204 No Content`: 智能体删除成功
- `404 Not Found`: 智能体不存在

**Section sources**
- [router.py](file://libs/agno/agno/os/router.py#L963-L981)

## 核心参数说明

### 模型选择
智能体的模型配置决定了其核心能力。支持多种模型提供商，包括OpenAI、Anthropic等。

**参数**:
- `name`: 模型名称
- `model`: 模型ID
- `provider`: 模型提供商

**示例**:
```json
"model": {
  "name": "OpenAIChat",
  "model": "gpt-4o",
  "provider": "OpenAI"
}
```

### 指令
指令用于指导智能体的行为和响应方式。

**参数**:
- `instructions`: 指令列表或字符串
- `expected_output`: 预期输出格式
- `additional_context`: 额外上下文

**示例**:
```json
"instructions": [
  "使用表格显示数据",
  "在响应中包含来源",
  "回答前先搜索知识库"
],
"expected_output": "## 引人入胜的文章标题\n\n### 概述\n{简要介绍文章和用户应阅读此报告的原因}"
```

### 工具集
工具集扩展了智能体的功能，使其能够执行特定任务。

**参数**:
- `tools`: 工具配置对象
- `tool_call_limit`: 工具调用限制
- `tool_choice`: 工具选择策略

**支持的工具类型**:
- `DuckDuckGoTools`: 网络搜索
- `YFinanceTools`: 金融数据
- `DalleTools`: 图像生成
- `ExaTools`: 研究工具
- `YouTubeTools`: YouTube视频分析

### 记忆设置
记忆设置使智能体能够记住用户信息和会话历史。

**参数**:
- `enable_user_memories`: 启用用户记忆
- `enable_agentic_memory`: 启用智能体记忆
- `add_memories_to_context`: 将记忆添加到上下文

**会话管理**:
- `add_history_to_context`: 将历史添加到上下文
- `num_history_runs`: 历史运行次数
- `enable_session_summaries`: 启用会话摘要

**Section sources**
- [agent.py](file://libs/agno/agno/agent/agent.py#L100-L500)
- [schema.py](file://libs/agno/agno/os/schema.py#L200-L300)

## API调用示例

### 使用Python requests库
```python
import requests
import json

# 创建智能体
url = "http://localhost:8000/agents"
headers = {
    "Authorization": "Bearer your-token",
    "Content-Type": "application/json"
}
data = {
    "id": "web-agent",
    "name": "Web Agent",
    "model": {
        "name": "OpenAIChat",
        "model": "gpt-4o",
        "provider": "OpenAI"
    },
    "tools": None,
    "sessions": {"session_table": "agno_sessions"},
    "knowledge": {"knowledge_table": "main_knowledge"},
    "system_message": {"markdown": True, "add_datetime_to_context": True}
}

response = requests.post(url, headers=headers, json=data)
print(response.json())
```

### 使用cURL
```bash
# 获取智能体列表
curl -X GET "http://localhost:8000/agents" \
  -H "Authorization: Bearer your-token" \
  -H "Content-Type: application/json"

# 创建智能体
curl -X POST "http://localhost:8000/agents" \
  -H "Authorization: Bearer your-token" \
  -H "Content-Type: application/json" \
  -d '{
    "id": "web-agent",
    "name": "Web Agent",
    "model": {
      "name": "OpenAIChat",
      "model": "gpt-4o",
      "provider": "OpenAI"
    },
    "tools": null,
    "sessions": {"session_table": "agno_sessions"},
    "knowledge": {"knowledge_table": "main_knowledge"},
    "system_message": {"markdown": true, "add_datetime_to_context": true}
  }'
```

**Section sources**
- [basic.py](file://cookbook/demo/agents/basic.py)
- [agent_with_tools.py](file://cookbook/examples/agents/agent_with_tools.py)

## 错误处理
API实现了标准的错误处理机制，返回详细的错误信息。

**常见错误响应**:
- `400 Bad Request`: 请求无效
- `401 Unauthorized`: 未授权访问
- `404 Not Found`: 资源未找到
- `422 Validation Error`: 验证错误
- `500 Internal Server Error`: 服务器内部错误

**错误响应Schema**:
```json
{
  "detail": "错误描述",
  "error_code": "错误代码"
}
```

**Section sources**
- [schema.py](file://libs/agno/agno/os/schema.py#L50-L100)
- [router.py](file://libs/agno/agno/os/router.py#L454-L481)