# 会话API

<cite>
**本文档引用的文件**
- [session.py](file://libs/agno/agno/os/routers/session/session.py)
- [agent.py](file://libs/agno/agno/agent/agent.py)
- [02_persistent_session_history.py](file://cookbook/agents/session/02_persistent_session_history.py)
- [06_rename_session.py](file://cookbook/agents/session/06_rename_session.py)
</cite>

## 目录
1. [简介](#简介)
2. [核心端点](#核心端点)
3. [获取会话列表](#获取会话列表)
4. [获取会话详情](#获取会话详情)
5. [更新会话元数据](#更新会话元数据)
6. [删除会话](#删除会话)
7. [获取会话消息历史](#获取会话消息历史)
8. [使用示例](#使用示例)

## 简介
会话API提供了一套完整的端点来管理用户会话，这对于维护对话历史和上下文至关重要。该API允许创建、检索、更新和删除会话，并支持获取会话的详细消息历史。会话代表了用户与系统之间的对话历史和执行上下文，可用于持久化对话状态和恢复之前的交互。

**Section sources**
- [session.py](file://libs/agno/agno/os/routers/session/session.py#L0-L552)

## 核心端点
会话API提供以下核心端点来管理用户会话：

- **POST /sessions**: 创建新会话（在代码中通过数据库操作隐式创建）
- **GET /sessions**: 获取会话列表，支持分页、过滤和排序
- **GET /sessions/{session_id}**: 获取特定会话的详细信息
- **PUT /sessions/{session_id}/rename**: 更新会话元数据（如重命名）
- **DELETE /sessions/{session_id}**: 删除指定会话
- **GET /sessions/{session_id}/runs**: 获取会话的运行历史（消息历史）

这些端点通过FastAPI实现，具有完整的OpenAPI文档，支持认证、错误处理和响应验证。

**Section sources**
- [session.py](file://libs/agno/agno/os/routers/session/session.py#L47-L551)

## 获取会话列表
`GET /sessions` 端点用于检索会话列表，支持分页、过滤和排序功能。

### 请求信息
- **HTTP方法**: GET
- **URL路径**: `/sessions`
- **操作ID**: get_sessions
- **摘要**: 列出会话

### 查询参数
| 参数 | 类型 | 必需 | 默认值 | 描述 |
|------|------|------|--------|------|
| type | SessionType | 否 | AGENT | 会话类型（agent、team或workflow） |
| component_id | string | 否 | 无 | 按组件ID过滤会话（agent/team/workflow ID） |
| user_id | string | 否 | 无 | 按用户ID过滤会话 |
| session_name | string | 否 | 无 | 按名称过滤会话（部分匹配） |
| limit | integer | 否 | 20 | 每页返回的会话数量 |
| page | integer | 否 | 1 | 分页的页码 |
| sort_by | string | 否 | created_at | 排序字段 |
| sort_order | SortOrder | 否 | desc | 排序顺序（asc或desc） |
| db_id | string | 否 | 无 | 查询会话的数据库ID |

### 响应
- **200 OK**: 成功检索到会话
  - 响应模型: `PaginatedResponse[SessionSchema]`
  - 内容示例:
    ```json
    {
      "data": [
        {
          "session_id": "6f6cfbfd-9643-479a-ae47-b8f32eb4d710",
          "session_name": "What tools do you have?",
          "session_state": {},
          "created_at": "2025-09-05T16:02:09Z",
          "updated_at": "2025-09-05T16:02:09Z"
        }
      ],
      "meta": {
        "page": 1,
        "limit": 20,
        "total_count": 1,
        "total_pages": 1
      }
    }
    ```
- **400 Bad Request**: 会话类型或过滤参数无效
- **422 Unprocessable Entity**: 查询参数验证错误

**Section sources**
- [session.py](file://libs/agno/agno/os/routers/session/session.py#L47-L125)

## 获取会话详情
`GET /sessions/{session_id}` 端点用于获取特定会话的详细信息，包括元数据、配置和运行历史。

### 请求信息
- **HTTP方法**: GET
- **URL路径**: `/sessions/{session_id}`
- **操作ID**: get_session_by_id
- **摘要**: 按ID获取会话

### 路径参数
| 参数 | 类型 | 必需 | 描述 |
|------|------|------|------|
| session_id | string | 是 | 要检索的会话ID |

### 查询参数
| 参数 | 类型 | 必需 | 默认值 | 描述 |
|------|------|------|--------|------|
| type | SessionType | 否 | AGENT | 会话类型（agent、team或workflow） |
| db_id | string | 否 | 无 | 查询会话的数据库ID |

### 响应
- **200 OK**: 成功检索到会话详情
  - 响应模型: `Union[AgentSessionDetailSchema, TeamSessionDetailSchema, WorkflowSessionDetailSchema]`
  - 响应模式根据会话类型而变化（agent、team或workflow）
- **404 Not Found**: 会话未找到
- **422 Unprocessable Entity**: 会话类型无效

**Section sources**
- [session.py](file://libs/agno/agno/os/routers/session/session.py#L126-L230)

## 更新会话元数据
`POST /sessions/{session_id}/rename` 端点用于更新会话的元数据，主要是重命名会话。

### 请求信息
- **HTTP方法**: POST
- **URL路径**: `/sessions/{session_id}/rename`
- **操作ID**: rename_session
- **摘要**: 重命名会话

### 路径参数
| 参数 | 类型 | 必需 | 描述 |
|------|------|------|------|
| session_id | string | 是 | 要重命名的会话ID |

### 查询参数
| 参数 | 类型 | 必需 | 默认值 | 描述 |
|------|------|------|--------|------|
| type | SessionType | 否 | AGENT | 会话类型（agent、team或workflow） |
| db_id | string | 否 | 无 | 用于重命名操作的数据库ID |

### 请求体
- **session_name**: 要为会话设置的新名称（嵌入式请求体）

### 响应
- **200 OK**: 会话重命名成功
  - 响应模型: `Union[AgentSessionDetailSchema, TeamSessionDetailSchema, WorkflowSessionDetailSchema]`
  - 返回更新后的会话详情
- **400 Bad Request**: 会话名称无效
- **404 Not Found**: 会话未找到
- **422 Unprocessable Entity**: 会话类型或验证错误

**Section sources**
- [session.py](file://libs/agno/agno/os/routers/session/session.py#L423-L543)

## 删除会话
会话API提供了删除单个或多个会话的端点。

### 删除单个会话
`DELETE /sessions/{session_id}` 端点用于永久删除特定会话及其所有关联的运行。

#### 请求信息
- **HTTP方法**: DELETE
- **URL路径**: `/sessions/{session_id}`
- **操作ID**: delete_session
- **摘要**: 删除会话

#### 路径参数
| 参数 | 类型 | 必需 | 描述 |
|------|------|------|------|
| session_id | string | 是 | 要删除的会话ID |

#### 查询参数
| 参数 | 类型 | 必需 | 默认值 | 描述 |
|------|------|------|--------|------|
| db_id | string | 否 | 无 | 用于删除的数据库ID |

#### 响应
- **204 No Content**: 会话删除成功
- **500 Internal Server Error**: 删除会话失败

### 删除多个会话
`DELETE /sessions` 端点用于在单个操作中删除多个会话。

#### 请求信息
- **HTTP方法**: DELETE
- **URL路径**: `/sessions`
- **操作ID**: delete_sessions
- **摘要**: 删除多个会话

#### 请求体
- **DeleteSessionRequest**: 包含要删除的会话ID列表和会话类型列表

#### 查询参数
| 参数 | 类型 | 必需 | 默认值 | 描述 |
|------|------|------|--------|------|
| type | SessionType | 否 | AGENT | 默认会话类型过滤器 |
| db_id | string | 否 | 无 | 用于删除的数据库ID |

#### 响应
- **204 No Content**: 会话删除成功
- **400 Bad Request**: 请求无效（会话ID和类型长度不匹配）
- **500 Internal Server Error**: 删除会话失败

**Section sources**
- [session.py](file://libs/agno/agno/os/routers/session/session.py#L388-L422)

## 获取会话消息历史
`GET /sessions/{session_id}/runs` 端点用于获取特定会话的所有运行（执行）历史，这代表了会话中的单个交互或执行。

### 请求信息
- **HTTP方法**: GET
- **URL路径**: `/sessions/{session_id}/runs`
- **操作ID**: get_session_runs
- **摘要**: 获取会话运行

### 路径参数
| 参数 | 类型 | 必需 | 描述 |
|------|------|------|------|
| session_id | string | 是 | 要获取运行的会话ID |

### 查询参数
| 参数 | 类型 | 必需 | 默认值 | 描述 |
|------|------|------|--------|------|
| type | SessionType | 否 | AGENT | 会话类型（agent、team或workflow） |
| db_id | string | 否 | 无 | 查询运行的数据库ID |

### 响应
- **200 OK**: 成功检索到会话运行
  - 响应模型: `List[Union[RunSchema, TeamRunSchema, WorkflowRunSchema]]`
  - 响应包含会话中所有运行的列表，每个运行包含输入、响应、指标和消息
- **404 Not Found**: 会话未找到或没有运行
- **422 Unprocessable Entity**: 会话类型无效

### 消息历史结构
每个运行中的`messages`字段包含会话的完整消息历史，包括：
- **system消息**: 系统指令和附加信息
- **user消息**: 用户输入
- **assistant消息**: 助手响应
- 每条消息包含角色、内容、创建时间和相关指标

**Section sources**
- [session.py](file://libs/agno/agno/os/routers/session/session.py#L231-L387)
- [agent.py](file://libs/agno/agno/agent/agent.py#L4574-L4603)

## 使用示例
以下示例展示了如何使用会话API来加载用户的对话历史，实现对话的持久化。

### 创建会话并启用历史记录
```python
from agno.agent.agent import Agent
from agno.db.postgres import PostgresDb
from agno.models.openai import OpenAIChat

db_url = "postgresql+psycopg://ai:ai@localhost:5532/ai"
db = PostgresDb(db_url=db_url, session_table="sessions")

agent = Agent(
    model=OpenAIChat(id="gpt-4o-mini"),
    db=db,
    session_id="session_storage",
    add_history_to_context=True,
    num_history_runs=2,
)

agent.print_response("Tell me a new interesting fact about space")
```

### 重命名会话
```python
from agno.agent.agent import Agent
from agno.db.postgres import PostgresDb
from agno.models.openai import OpenAIChat

db_url = "postgresql+psycopg://ai:ai@localhost:5532/ai"
db = PostgresDb(db_url=db_url, session_table="sessions")

agent = Agent(
    model=OpenAIChat(id="gpt-4o-mini"),
    db=db,
    session_id="chat_history",
    instructions="You are a helpful assistant that can answer questions about space and oceans.",
    add_history_to_context=True,
)

agent.print_response("Tell me a new interesting fact about space")
agent.set_session_name(session_name="Interesting Space Facts")

session = agent.get_session(session_id=agent.session_id)
print(session.session_data.get("session_name"))

agent.set_session_name(autogenerate=True)
```

这些示例展示了如何配置代理以持久化会话和历史记录，以及如何通过API重命名会话以更好地组织和管理对话。

**Section sources**
- [02_persistent_session_history.py](file://cookbook/agents/session/02_persistent_session_history.py#L0-L23)
- [06_rename_session.py](file://cookbook/agents/session/06_rename_session.py#L0-L26)