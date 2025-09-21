# 团队API

<cite>
**本文档中引用的文件**  
- [team.py](file://libs/agno/agno/team/team.py)
- [team.py](file://cookbook/agent_os/_teams.py)
- [teams_demo.py](file://cookbook/agent_os/teams_demo.py)
- [team.py](file://libs/agno/agno/api/schemas/team.py)
- [team.py](file://libs/agno/agno/run/team.py)
</cite>

## 目录
1. [简介](#简介)
2. [团队生命周期端点](#团队生命周期端点)
3. [启动团队运行端点](#启动团队运行端点)
4. [团队配置参数](#团队配置参数)
5. [代码示例](#代码示例)
6. [响应结构](#响应结构)

## 简介
团队API提供了一套完整的接口来管理团队的生命周期和操作。该API允许创建、检索、更新和删除团队，并支持启动团队运行以执行协调任务。团队由多个代理组成，可以配置不同的协调模式来实现复杂的协作行为。

**Section sources**
- [team.py](file://libs/agno/agno/team/team.py#L1-L50)

## 团队生命周期端点
团队生命周期端点提供了对团队资源的完整CRUD操作。

### 创建团队 (POST /teams)
创建一个新的团队实例。

**请求头**
- Content-Type: application/json

**请求体Schema**
```json
{
  "members": [
    {
      "id": "string",
      "name": "string",
      "model": "string"
    }
  ],
  "name": "string",
  "description": "string",
  "instructions": "string",
  "model": "string"
}
```

**响应Schema**
```json
{
  "team_id": "string",
  "name": "string",
  "created_at": "string",
  "status": "active"
}
```

### 获取团队列表 (GET /teams)
检索所有团队的列表。

**响应Schema**
```json
[
  {
    "team_id": "string",
    "name": "string",
    "description": "string",
    "member_count": 0,
    "created_at": "string",
    "updated_at": "string"
  }
]
```

### 获取单个团队详情 (GET /teams/{team_id})
获取特定团队的详细信息。

**响应Schema**
```json
{
  "team_id": "string",
  "name": "string",
  "description": "string",
  "instructions": "string",
  "members": [
    {
      "id": "string",
      "name": "string",
      "role": "string"
    }
  ],
  "created_at": "string",
  "updated_at": "string",
  "status": "string"
}
```

### 更新团队配置 (PUT /teams/{team_id})
更新现有团队的配置。

**请求体Schema**
```json
{
  "name": "string",
  "description": "string",
  "instructions": "string",
  "members": [
    {
      "id": "string",
      "name": "string",
      "role": "string"
    }
  ]
}
```

**响应Schema**
```json
{
  "team_id": "string",
  "updated_at": "string",
  "status": "updated"
}
```

### 删除团队 (DELETE /teams/{team_id})
删除指定的团队。

**响应Schema**
```json
{
  "team_id": "string",
  "deleted_at": "string",
  "status": "deleted"
}
```

**Section sources**
- [team.py](file://libs/agno/agno/team/team.py#L100-L200)

## 启动团队运行端点
启动团队运行端点是团队API的核心功能，用于触发团队执行其协调任务。

### 启动团队运行 (POST /teams/{team_id}/run)
启动指定团队的运行。

**请求头**
- Content-Type: application/json
- Authorization: Bearer {token}

**请求体结构**
```json
{
  "data": {
    "input": "string",
    "context": {}
  },
  "session_id": "string",
  "run_config": {
    "stream": false,
    "debug": false,
    "metadata": {}
  }
}
```

**字段说明**
- **data**: 包含运行输入数据的对象
  - input: 用户输入的文本内容
  - context: 附加的上下文信息
- **session_id**: 关联的会话ID，用于保持对话状态
- **run_config**: 运行配置选项
  - stream: 是否流式传输响应
  - debug: 是否启用调试模式
  - metadata: 附加的元数据

**响应Schema**
```json
{
  "run_id": "string",
  "team_id": "string",
  "session_id": "string",
  "status": "running",
  "created_at": "string",
  "output": {}
}
```

**Section sources**
- [team.py](file://libs/agno/agno/team/team.py#L1666-L1700)
- [team.py](file://libs/agno/agno/api/schemas/team.py#L1-L20)

## 团队配置参数
团队配置中的关键参数决定了团队的行为和协作模式。

### 协调模式
团队支持多种协调模式来控制成员间的交互：

**协作模式 (collaborate_mode)**
- 成员可以相互交流和协作
- 信息在成员间共享
- 适用于需要深度协作的任务

**协调模式 (coordinate_mode)**
- 团队领导者协调成员活动
- 成员不直接相互通信
- 适用于需要集中控制的任务

**路由模式 (route_mode)**
- 根据输入自动路由到合适的成员
- 动态分配任务
- 适用于多领域专家系统

### 成员列表
成员列表定义了团队的组成和角色：

```json
"members": [
  {
    "id": "researcher_001",
    "name": "Research Agent",
    "role": "researcher",
    "model": "gpt-4",
    "tools": ["web_search", "knowledge_base"]
  },
  {
    "id": "writer_001",
    "name": "Writer Agent",
    "role": "writer",
    "model": "gpt-4",
    "tools": ["content_generation", "grammar_check"]
  }
]
```

### 共享上下文
共享上下文机制允许团队成员访问共同的信息：

- **session_state**: 会话状态在成员间共享
- **dependencies**: 用户提供的依赖项
- **knowledge**: 共享的知识库
- **memory**: 团队记忆和历史

**Section sources**
- [team.py](file://libs/agno/agno/team/team.py#L50-L100)
- [team.py](file://cookbook/agent_os/_teams.py#L1-L50)

## 代码示例
以下代码示例展示了如何使用API创建一个包含多个成员的协调团队并启动其运行。

### 创建协调团队
```python
from agno.team import Team
from agno.agent import Agent

# 创建团队成员
researcher = Agent(
    name="Research Agent",
    model="gpt-4",
    tools=["web_search", "knowledge_base"],
    instructions="You are a research expert. Find accurate information."
)

writer = Agent(
    name="Content Writer",
    model="gpt-4",
    tools=["content_generation"],
    instructions="You are a professional writer. Create engaging content."
)

# 创建协调团队
coordination_team = Team(
    name="Content Creation Team",
    model="gpt-4",
    members=[researcher, writer],
    description="A team that researches and creates content",
    instructions="Coordinate the research and writing process",
    respond_directly=False,
    delegate_task_to_all_members=False
)
```

### 启动团队运行
```python
# 启动团队运行
run_result = coordination_team.run(
    input="Write an article about renewable energy trends in 2024",
    session_id="session_123",
    run_config={
        "stream": True,
        "debug": False,
        "metadata": {"project": "energy_report"}
    }
)

# 处理流式响应
async for event in coordination_team.arun(
    input="Analyze the latest AI developments",
    stream=True,
    session_id="session_456"
):
    if event.event == "run_content":
        print(f"Response: {event.content}")
    elif event.event == "tool_call_started":
        print(f"Using tool: {event.tool.name}")
```

**Section sources**
- [teams_demo.py](file://cookbook/agent_os/teams_demo.py#L1-L100)
- [team.py](file://libs/agno/agno/team/team.py#L1666-L1859)

## 响应结构
团队API的响应遵循统一的结构，包含运行状态、输出和元数据。

### 运行响应结构
```json
{
  "run_id": "string",
  "team_id": "string",
  "session_id": "string",
  "status": "running|completed|failed|cancelled",
  "created_at": "string",
  "completed_at": "string",
  "output": {
    "content": "string",
    "reasoning": [],
    "tools": [],
    "metrics": {}
  },
  "events": [],
  "metadata": {}
}
```

### 事件流响应
当启用流式传输时，API会发送一系列事件：

- **run_started**: 运行开始事件
- **tool_call_started**: 工具调用开始
- **tool_call_completed**: 工具调用完成
- **run_content**: 运行内容更新
- **run_completed**: 运行完成
- **run_error**: 运行错误

**Section sources**
- [team.py](file://libs/agno/agno/run/team.py#L1-L50)
- [team.py](file://libs/agno/agno/team/team.py#L2060-L2259)