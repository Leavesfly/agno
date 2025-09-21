# AgentOS RESTful API 参考文档

<cite>
**本文档中引用的文件**
- [router.py](file://libs/agno/agno/os/router.py)
- [app.py](file://libs/agno/agno/os/app.py)
- [auth.py](file://libs/agno/agno/os/auth.py)
- [schema.py](file://libs/agno/agno/os/schema.py)
- [settings.py](file://libs/agno/agno/api/settings.py)
- [api.py](file://libs/agno/agno/api/api.py)
- [routes.py](file://libs/agno/agno/api/routes.py)
- [run.py](file://cookbook/demo/run.py)
- [config.yaml](file://cookbook/demo/config.yaml)
</cite>

## 目录
1. [简介](#简介)
2. [认证系统](#认证系统)
3. [API 端点概览](#api-端点概览)
4. [核心资源 API](#核心资源-api)
5. [事件流和 WebSocket](#事件流和-websocket)
6. [错误处理](#错误处理)
7. [速率限制](#速率限制)
8. [客户端实现指南](#客户端实现指南)
9. [版本控制和兼容性](#版本控制和兼容性)
10. [故障排除](#故障排除)

## 简介

AgentOS 提供了一个完整的 RESTful API 系统，用于管理智能体、团队和工作流程。该 API 基于 FastAPI 构建，支持实时事件流、多模态媒体处理和高级认证功能。

### 主要特性

- **RESTful 设计**: 符合 REST 原则的 HTTP 接口
- **实时流式响应**: 支持 Server-Sent Events (SSE)
- **WebSocket 连接**: 实时事件推送和交互
- **多模态支持**: 图像、音频、视频和文档处理
- **认证安全**: API 密钥和 WebSocket 认证
- **错误处理**: 完整的状态码和错误响应
- **可扩展性**: 支持自定义路由和中间件

## 认证系统

### API 密钥认证

AgentOS 使用基于 API 密钥的认证系统。需要在 HTTP 请求头中包含 `Authorization` 字段。

#### 配置

```python
# 在 AgentOS 设置中启用安全
from agno.os.settings import AgnoAPISettings

settings = AgnoAPISettings(
    os_security_key="your-secret-key-here"
)
```

#### HTTP 请求头

```http
Authorization: Bearer your-secret-key-here
```

#### Python 客户端示例

```python
import requests

headers = {
    "Authorization": "Bearer your-secret-key-here",
    "Content-Type": "application/json"
}

response = requests.get("http://localhost:7777/config", headers=headers)
```

#### curl 示例

```bash
curl -H "Authorization: Bearer your-secret-key-here" \
     -H "Content-Type: application/json" \
     http://localhost:7777/config
```

### WebSocket 认证

WebSocket 连接需要单独的认证过程：

```javascript
// 建立 WebSocket 连接
const ws = new WebSocket('ws://localhost:7777/workflows/ws');

ws.onopen = () => {
    // 发送认证消息
    ws.send(JSON.stringify({
        action: 'authenticate',
        token: 'your-secret-key-here'
    }));
};

ws.onmessage = (event) => {
    const message = JSON.parse(event.data);
    if (message.event === 'authenticated') {
        console.log('WebSocket authenticated');
        // 现在可以发送其他命令
    }
};
```

**节来源**
- [auth.py](file://libs/agno/agno/os/auth.py#L1-L58)

## API 端点概览

### 核心端点

| 方法 | 路径 | 描述 |
|------|------|------|
| GET | `/config` | 获取 OS 配置信息 |
| GET | `/health` | 检查服务健康状态 |
| GET | `/models` | 获取可用模型列表 |

### 智能体端点

| 方法 | 路径 | 描述 |
|------|------|------|
| GET | `/agents` | 列出所有智能体 |
| GET | `/agents/{agent_id}` | 获取特定智能体详情 |
| POST | `/agents/{agent_id}/runs` | 执行智能体运行 |
| POST | `/agents/{agent_id}/runs/{run_id}/cancel` | 取消智能体运行 |
| POST | `/agents/{agent_id}/runs/{run_id}/continue` | 继续智能体运行 |

### 团队端点

| 方法 | 路径 | 描述 |
|------|------|------|
| GET | `/teams` | 列出所有团队 |
| GET | `/teams/{team_id}` | 获取特定团队详情 |
| POST | `/teams/{team_id}/runs` | 执行团队协作 |
| POST | `/teams/{team_id}/runs/{run_id}/cancel` | 取消团队运行 |

### 工作流程端点

| 方法 | 路径 | 描述 |
|------|------|------|
| GET | `/workflows` | 列出所有工作流程 |
| GET | `/workflows/{workflow_id}` | 获取特定工作流程详情 |
| POST | `/workflows/{workflow_id}/runs` | 执行工作流程 |
| POST | `/workflows/{workflow_id}/runs/{run_id}/cancel` | 取消工作流程运行 |

### 会话管理端点

| 方法 | 路径 | 描述 |
|------|------|------|
| GET | `/sessions` | 列出会话 |
| GET | `/sessions/{session_id}` | 获取会话详情 |
| DELETE | `/sessions` | 删除会话 |

### 内存管理端点

| 方法 | 路径 | 描述 |
|------|------|------|
| GET | `/memories` | 列出内存 |
| POST | `/memories` | 创建内存 |
| GET | `/memories/{memory_id}` | 获取内存详情 |
| PUT | `/memories/{memory_id}` | 更新内存 |
| DELETE | `/memories/{memory_id}` | 删除内存 |

**节来源**
- [router.py](file://libs/agno/agno/os/router.py#L483-L1513)

## 核心资源 API

### 配置端点

#### 获取 OS 配置

```http
GET /config
```

**响应格式**:

```json
{
    "os_id": "demo",
    "description": "Example AgentOS configuration",
    "available_models": ["gpt-4", "claude-3"],
    "databases": ["db-123"],
    "agents": [
        {
            "id": "main-agent",
            "name": "Main Agent",
            "db_id": "db-123"
        }
    ],
    "teams": [],
    "workflows": [],
    "interfaces": []
}
```

**节来源**
- [router.py](file://libs/agno/agno/os/router.py#L483-L580)

### 智能体 API

#### 创建智能体运行

```http
POST /agents/{agent_id}/runs
```

**请求参数**:

| 参数 | 类型 | 必需 | 描述 |
|------|------|------|------|
| message | string | 是 | 用户输入消息 |
| stream | boolean | 否 | 是否启用流式响应，默认true |
| session_id | string | 否 | 会话ID，为空时自动生成 |
| user_id | string | 否 | 用户ID |
| files | file[] | 否 | 多媒体文件 |

**支持的文件类型**:
- 图像: PNG, JPEG, JPG, WebP
- 音频: WAV, MP3, MPEG
- 视频: MP4, WebM, FLV, QuickTime等
- 文档: PDF, CSV, DOCX, TXT, JSON

**响应格式**:

```json
{
    "run_id": "uuid-string",
    "content": "智能体响应内容",
    "created_at": 1757348314
}
```

**节来源**
- [router.py](file://libs/agno/agno/os/router.py#L600-L750)

#### 取消智能体运行

```http
POST /agents/{agent_id}/runs/{run_id}/cancel
```

**响应**: 成功时返回空对象 `{}`

#### 继续智能体运行

```http
POST /agents/{agent_id}/runs/{run_id}/continue
```

**请求参数**:

| 参数 | 类型 | 必需 | 描述 |
|------|------|------|------|
| tools | string | 是 | 包含工具执行结果的JSON字符串 |

**工具数据格式**:

```json
[
    {
        "tool_name": "tool_name",
        "tool_input": {...},
        "tool_output": {...},
        "tool_status": "success|failed"
    }
]
```

**节来源**
- [router.py](file://libs/agno/agno/os/router.py#L750-L850)

### 团队 API

#### 创建团队运行

```http
POST /teams/{team_id}/runs
```

**请求参数**:

| 参数 | 类型 | 必需 | 描述 |
|------|------|------|------|
| message | string | 是 | 用户输入消息 |
| stream | boolean | 否 | 是否启用流式响应，默认true |
| monitor | boolean | 否 | 是否监控团队运行，默认true |
| session_id | string | 否 | 会话ID |
| user_id | string | 否 | 用户ID |
| files | file[] | 否 | 多媒体文件 |

**响应格式**:

```json
{
    "run_id": "uuid-string",
    "content": "团队协作响应",
    "created_at": 1757348314,
    "team_members": [
        {
            "member_id": "agent-123",
            "response": "成员响应内容"
        }
    ]
}
```

**节来源**
- [router.py](file://libs/agno/agno/os/router.py#L850-L1000)

### 工作流程 API

#### 创建工作流程运行

```http
POST /workflows/{workflow_id}/runs
```

**请求参数**:

| 参数 | 类型 | 必需 | 描述 |
|------|------|------|------|
| message | string | 是 | 输入数据或消息 |
| stream | boolean | 否 | 是否启用流式响应，默认true |
| session_id | string | 否 | 会话ID |
| user_id | string | 否 | 用户ID |

**输入数据格式**:

```json
{
    "step_name": "step_value",
    "another_step": "another_value"
}
```

**响应格式**:

```json
{
    "run_id": "uuid-string",
    "content": "工作流程最终输出",
    "created_at": 1757348314,
    "steps": [
        {
            "step_id": "step-1",
            "status": "completed",
            "output": "步骤输出"
        }
    ]
}
```

**节来源**
- [router.py](file://libs/agno/agno/os/router.py#L1300-L1450)

## 事件流和 WebSocket

### WebSocket 连接

AgentOS 提供实时事件流通过 WebSocket 连接：

```javascript
const ws = new WebSocket('ws://localhost:7777/workflows/ws');

ws.onopen = () => {
    console.log('WebSocket connected');
    
    // 如果启用了安全认证
    if (needsAuthentication) {
        ws.send(JSON.stringify({
            action: 'authenticate',
            token: 'your-secret-key-here'
        }));
    }
};

ws.onmessage = (event) => {
    const message = JSON.parse(event.data);
    console.log('Received event:', message.event);
    console.log('Data:', message.data);
};
```

### 事件类型

#### SSE 事件

当使用流式响应时，服务器会发送以下事件：

```javascript
// 事件格式
event: EventType
data: {"content": "事件数据", "run_id": "uuid"}

// 示例事件
event: RunStarted
data: {"content": "开始执行...", "run_id": "uuid-123"}

event: RunContent
data: {"content": "正在处理...", "run_id": "uuid-123"}

event: RunFinished
data: {"content": "完成！", "run_id": "uuid-123"}
```

#### WebSocket 事件

WebSocket 连接支持以下动作：

| 动作 | 描述 |
|------|------|
| authenticate | 认证连接 |
| ping | 心跳检测 |
| start-workflow | 开始工作流程 |

**节来源**
- [router.py](file://libs/agno/agno/os/router.py#L150-L250)

## 错误处理

### HTTP 状态码

| 状态码 | 描述 | 响应格式 |
|--------|------|----------|
| 200 | 成功 | JSON 响应 |
| 400 | 请求错误 | `BadRequestResponse` |
| 401 | 未授权 | `UnauthenticatedResponse` |
| 404 | 资源不存在 | `NotFoundResponse` |
| 422 | 验证错误 | `ValidationErrorResponse` |
| 500 | 内部服务器错误 | `InternalServerErrorResponse` |

### 错误响应格式

```json
{
    "detail": "错误描述信息",
    "error_code": "错误代码"
}
```

### 常见错误场景

#### 文件类型不支持

```json
{
    "detail": "Unsupported file type",
    "error_code": "UNSUPPORTED_FILE_TYPE"
}
```

#### 智能体未找到

```json
{
    "detail": "Agent not found",
    "error_code": "AGENT_NOT_FOUND"
}
```

#### 工作流程执行失败

```json
{
    "detail": "Error running workflow: step failed",
    "error_code": "WORKFLOW_EXECUTION_ERROR"
}
```

**节来源**
- [schema.py](file://libs/agno/agno/os/schema.py#L20-L80)

## 速率限制

### 默认限制

AgentOS 不强制实施速率限制，但建议客户端实现适当的重试逻辑：

```python
import time
import random

def exponential_backoff_retry(func, max_retries=3):
    """指数退避重试机制"""
    for i in range(max_retries):
        try:
            return func()
        except Exception as e:
            if i == max_retries - 1:
                raise e
            
            # 指数退避延迟
            delay = (2 ** i) + random.uniform(0, 1)
            time.sleep(delay)
```

### 最佳实践

1. **批量操作**: 尽量减少单个请求的数量
2. **并发控制**: 控制同时进行的请求数量
3. **缓存策略**: 缓存频繁查询的结果
4. **优雅降级**: 在服务不可用时提供备用方案

## 客户端实现指南

### Python 客户端

```python
import requests
import json
from typing import Dict, List, Optional

class AgentOSClient:
    def __init__(self, base_url: str, api_key: str):
        self.base_url = base_url.rstrip('/')
        self.headers = {
            "Authorization": f"Bearer {api_key}",
            "Content-Type": "application/json"
        }
    
    def get_config(self) -> Dict:
        """获取配置"""
        response = requests.get(f"{self.base_url}/config", headers=self.headers)
        response.raise_for_status()
        return response.json()
    
    def create_agent_run(self, agent_id: str, message: str, 
                         stream: bool = True, session_id: Optional[str] = None) -> Dict:
        """创建智能体运行"""
        data = {
            "message": message,
            "stream": stream,
            "session_id": session_id
        }
        
        response = requests.post(
            f"{self.base_url}/agents/{agent_id}/runs",
            headers=self.headers,
            data=data
        )
        response.raise_for_status()
        return response.json()
    
    def stream_agent_run(self, agent_id: str, message: str) -> str:
        """流式执行智能体运行"""
        response = requests.post(
            f"{self.base_url}/agents/{agent_id}/runs",
            headers=self.headers,
            data={
                "message": message,
                "stream": "true"
            },
            stream=True
        )
        
        content = ""
        for line in response.iter_lines():
            if line.startswith(b'data: '):
                try:
                    data = json.loads(line.decode('utf-8')[6:])
                    content += data.get('content', '')
                except:
                    pass
        
        return content
```

### JavaScript 客户端

```javascript
class AgentOSClient {
    constructor(baseUrl, apiKey) {
        this.baseUrl = baseUrl.replace(/\/$/, '');
        this.headers = {
            'Authorization': `Bearer ${apiKey}`,
            'Content-Type': 'application/json'
        };
    }
    
    async getConfig() {
        const response = await fetch(`${this.baseUrl}/config`, {
            headers: this.headers
        });
        if (!response.ok) throw new Error(await response.text());
        return await response.json();
    }
    
    async createAgentRun(agentId, message, options = {}) {
        const data = new FormData();
        data.append('message', message);
        data.append('stream', options.stream !== false ? 'true' : 'false');
        if (options.sessionId) data.append('session_id', options.sessionId);
        
        const response = await fetch(`${this.baseUrl}/agents/${agentId}/runs`, {
            method: 'POST',
            headers: {
                'Authorization': this.headers.Authorization
            },
            body: data
        });
        
        if (!response.ok) throw new Error(await response.text());
        return await response.json();
    }
    
    streamAgentRun(agentId, message, onMessage) {
        const ws = new WebSocket(`${this.baseUrl.replace('http', 'ws')}/workflows/ws`);
        
        ws.onopen = () => {
            ws.send(JSON.stringify({
                action: 'start-workflow',
                workflow_id: agentId,
                message: message
            }));
        };
        
        ws.onmessage = (event) => {
            const data = JSON.parse(event.data);
            onMessage(data);
        };
        
        return ws;
    }
}
```

### cURL 示例

```bash
# 获取配置
curl -H "Authorization: Bearer your-key" \
     -H "Content-Type: application/json" \
     http://localhost:7777/config

# 创建智能体运行
curl -X POST \
     -H "Authorization: Bearer your-key" \
     -F "message=你好，世界！" \
     -F "stream=true" \
     http://localhost:7777/agents/main-agent/runs

# 流式响应（需要处理 SSE）
curl -H "Authorization: Bearer your-key" \
     -H "Accept: text/event-stream" \
     http://localhost:7777/agents/main-agent/runs
```

## 版本控制和兼容性

### API 版本策略

AgentOS 使用语义化版本控制：

- **主版本**: 重大架构变更，可能破坏兼容性
- **次版本**: 新功能添加，向后兼容
- **修订版本**: 错误修复，向后兼容

### 向后兼容性

1. **字段保留**: 移除字段时会标记为废弃
2. **默认值**: 新字段提供合理的默认值
3. **迁移指南**: 重大变更时提供迁移文档

### 兼容性检查

```python
import requests

def check_api_compatibility(base_url: str, min_version: str = "1.0.0") -> bool:
    """检查 API 兼容性"""
    try:
        response = requests.get(f"{base_url}/config")
        if response.status_code == 200:
            config = response.json()
            # 检查必要的字段是否存在
            required_fields = ['os_id', 'agents']
            return all(field in config for field in required_fields)
    except Exception:
        return False
    return True
```

## 故障排除

### 常见问题

#### 1. 认证失败

**症状**: 返回 401 状态码

**解决方案**:
```bash
# 检查 API 密钥是否正确
export AGNO_API_KEY="your-correct-key"

# 验证连接
curl -H "Authorization: Bearer $AGNO_API_KEY" \
     http://localhost:7777/config
```

#### 2. 文件上传失败

**症状**: 返回 400 状态码，提示文件类型不支持

**解决方案**:
- 确认文件格式在支持列表中
- 检查文件大小限制
- 验证文件编码

#### 3. WebSocket 连接被拒绝

**症状**: WebSocket 连接立即断开

**解决方案**:
```javascript
// 添加重连逻辑
let reconnectAttempts = 0;
const maxReconnectAttempts = 5;

function connectWebSocket() {
    const ws = new WebSocket('ws://localhost:7777/workflows/ws');
    
    ws.onclose = () => {
        if (reconnectAttempts < maxReconnectAttempts) {
            setTimeout(() => {
                reconnectAttempts++;
                connectWebSocket();
            }, 1000 * reconnectAttempts);
        }
    };
}
```

#### 4. 流式响应中断

**症状**: SSE 连接意外断开

**解决方案**:
```python
def handle_stream_response(response):
    """处理流式响应的断线重连"""
    try:
        for line in response.iter_lines():
            if line:
                yield line
    except requests.exceptions.ConnectionError:
        # 实现重连逻辑
        time.sleep(5)
        # 重新建立连接...
```

### 调试技巧

#### 启用详细日志

```python
import logging

# 设置日志级别
logging.basicConfig(level=logging.DEBUG)
logger = logging.getLogger('agno')
logger.setLevel(logging.DEBUG)
```

#### 检查网络连接

```bash
# 测试基本连接
curl -I http://localhost:7777/health

# 检查防火墙设置
telnet localhost 7777

# 查看端口监听状态
netstat -an | grep 7777
```

#### 验证配置

```python
# 检查 AgentOS 配置
from agno.os import AgentOS

agent_os = AgentOS(
    agents=[your_agent],
    teams=[your_team],
    workflows=[your_workflow]
)

app = agent_os.get_app()
print("Available routes:")
for route in app.routes:
    print(f"- {route.path}")
```

**节来源**
- [app.py](file://libs/agno/agno/os/app.py#L267-L389)