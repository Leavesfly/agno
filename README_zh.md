<div align="center" id="top">
  <a href="https://docs.agno.com">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="https://agno-public.s3.us-east-1.amazonaws.com/assets/logo-dark.svg">
      <source media="(prefers-color-scheme: light)" srcset="https://agno-public.s3.us-east-1.amazonaws.com/assets/logo-light.svg">
      <img src="https://agno-public.s3.us-east-1.amazonaws.com/assets/logo-light.svg" alt="Agno">
    </picture>
  </a>
</div>
<div align="center">
  <a href="https://docs.agno.com">📚 文档</a> &nbsp;|&nbsp;
  <a href="https://docs.agno.com/examples/introduction">💡 示例</a> &nbsp;|&nbsp;
  <a href="https://www.agno.com/?utm_source=github&utm_medium=readme&utm_campaign=agno-github&utm_content=header">🏠 官网</a> &nbsp;|&nbsp;
  <a href="https://github.com/agno-agi/agno/stargazers">🌟 点赞支持</a>
</div>

## 什么是 Agno？

[Agno](https://docs.agno.com) 是一个高性能的多智能体系统运行时环境。使用它可以在您的云端构建、运行和管理安全的多智能体系统。

Agno 为您提供了构建智能体的最快框架，支持会话管理、记忆管理、知识库、人机协作和 MCP 协议。您可以将智能体组合成自主的多智能体团队，或构建基于步骤的智能体工作流，以完全控制复杂的多步骤流程。

仅用 10 行代码，我们就可以构建一个从 HackerNews 获取热门故事并进行总结的智能体。

```python hackernews_agent.py
from agno.agent import Agent
from agno.models.anthropic import Claude
from agno.tools.hackernews import HackerNewsTools

agent = Agent(
    model=Claude(id="claude-sonnet-4-0"),
    tools=[HackerNewsTools()],
    markdown=True,
)
agent.print_response("总结 hackernews 上前5个热门故事", stream=True)
```

但 Agno 的真正优势在于其 [AgentOS](https://docs.agno.com/agent-os/introduction) 运行时：

1. 您将获得一个预构建的 FastAPI 应用程序来运行您的智能体系统，这意味着您可以从第一天就开始构建您的产品。这相比其他解决方案或自己从头构建具有显著优势。
2. 您还将获得一个控制平面，它直接连接到您的 AgentOS 进行测试、监控和管理您的系统。这为您提供了对系统无与伦比的可见性和控制力。
3. 您的 AgentOS 运行在您的云端，您获得完整的数据隐私，因为数据永远不会离开您的系统。这对于注重安全的企业来说非常重要，他们不能将跟踪数据发送到外部服务。

对于构建智能体的组织，Agno 提供了完整的解决方案。您获得构建智能体的最快框架（开发速度和执行速度），一个让您从第一天就能构建产品的预构建 FastAPI 应用程序，以及一个用于管理系统的控制平面。

我们带来了一个没有其他框架提供的创新架构，您的 AgentOS 安全地运行在您的云端，控制平面直接从您的浏览器连接到它。您不需要将数据发送到外部服务或支付保留成本，您获得完整的隐私和控制权。

## 快速开始

如果您是 Agno 的新用户，请遵循我们的[快速开始指南](https://docs.agno.com/introduction/quickstart)来构建您的第一个智能体并使用 AgentOS 运行它。

之后，请查看[示例库](https://docs.agno.com/examples/introduction)并使用 Agno 构建真实世界的应用程序。

## 文档、社区和更多示例

- 文档：<a href="https://docs.agno.com" target="_blank" rel="noopener noreferrer">docs.agno.com</a>
- 示例代码：<a href="https://github.com/agno-agi/agno/tree/main/cookbook" target="_blank" rel="noopener noreferrer">Cookbook</a>
- 社区论坛：<a href="https://community.agno.com/" target="_blank" rel="noopener noreferrer">community.agno.com</a>
- Discord：<a href="https://discord.gg/4MtYHHrgA8" target="_blank" rel="noopener noreferrer">discord</a>

## 配置编程智能体使用 Agno

为了让 LLM 和 AI 助手理解和导航 Agno 的文档，我们提供了一个 [llms.txt](https://docs.agno.com/llms.txt) 或 [llms-full.txt](https://docs.agno.com/llms-full.txt) 文件。

这个文件是为 AI 系统高效解析和引用我们的文档而构建的。

### IDE 集成

在构建 Agno 智能体时，将 Agno 文档作为 IDE 中的源代码是加速开发的好方法。以下是与 Cursor 集成的方法：

1. 在 Cursor 中，转到"Cursor Settings"菜单。
2. 找到"Indexing & Docs"部分。
3. 将 `https://docs.agno.com/llms-full.txt` 添加到文档 URL 列表中。
4. 保存更改。

现在，Cursor 将可以访问 Agno 文档。您可以在其他 IDE（如 VSCode、Windsurf 等）中执行相同操作。

## 核心特性

### 🤖 智能体（Agent）
- **会话管理**：支持持久化会话和历史记录
- **记忆管理**：内置用户记忆和会话摘要功能
- **知识集成**：支持向量数据库和 RAG（检索增强生成）
- **工具调用**：丰富的工具生态系统
- **多模态支持**：图像、音频、视频和文档处理
- **人机协作**：支持确认机制和外部工具执行

### 👥 团队（Team）
团队是多个智能体协作的高级抽象，支持多种协作模式：

- **协作模式**：所有成员共同参与任务
- **协调模式**：通过领导智能体进行任务分配
- **路由模式**：根据任务类型选择特定成员

### 🔄 工作流（Workflow）
工作流提供了步骤化的流程控制能力：

- **顺序执行**：步骤按列表顺序依次执行
- **并行执行**：使用 `Parallel` 组件同时执行多个步骤
- **条件执行**：使用 `Condition` 组件根据条件决定执行
- **循环执行**：使用 `Loop` 组件重复执行步骤
- **动态路由**：使用 `Router` 组件根据输入选择执行路径

### 🏢 AgentOS 运行时环境
- **预构建 FastAPI 应用**：立即可用的 Web 服务
- **控制平面**：系统监控和管理界面
- **数据隐私**：完全私有部署，数据不离开您的系统
- **高性能**：优化的执行引擎和内存管理

## 安装

### 基础安装

```bash
pip install agno
```

### 完整安装（包含所有依赖）

```bash
# 安装所有模型支持
pip install agno[models]

# 安装所有工具支持
pip install agno[tools]

# 安装所有存储支持
pip install agno[storage]

# 安装所有向量数据库支持
pip install agno[vectordbs]

# 安装完整版本
pip install agno[tests]
```

### 特定功能安装

```bash
# OpenAI 模型支持
pip install agno[openai]

# Anthropic 模型支持
pip install agno[anthropic]

# PostgreSQL 存储支持
pip install agno[postgres]

# 知识库功能
pip install agno[knowledge]

# AgentOS 支持
pip install agno[os]
```

## 环境配置

### API 密钥配置

```bash
# OpenAI API 密钥
export OPENAI_API_KEY=your_api_key_here

# Anthropic API 密钥
export ANTHROPIC_API_KEY=your_api_key_here

# 其他 API 密钥
export EXA_API_KEY=your_exa_api_key_here
export MODELS_LAB_API_KEY=your_models_lab_api_key_here
```

### 数据库配置

```bash
# PostgreSQL 数据库
export DATABASE_URL=postgresql+psycopg://user:password@localhost:5432/mydb

# Redis 缓存
export REDIS_URL=redis://localhost:6379/0
```

## 快速示例

### 基础智能体

```python
from agno.agent import Agent
from agno.models.openai import OpenAIChat

# 创建一个简单的智能体
agent = Agent(
    name="我的助手",
    model=OpenAIChat(id="gpt-4o"),
    instructions="你是一个友好且乐于助人的AI助手。"
)

# 与智能体对话
agent.print_response("你好！介绍一下你自己。")
```

### 带工具的智能体

```python
from agno.agent import Agent
from agno.models.openai import OpenAIChat
from agno.tools.calculator import CalculatorTools

agent = Agent(
    name="计算助手",
    model=OpenAIChat(id="gpt-4o"),
    tools=[CalculatorTools()],
    instructions="你是一个数学计算助手。"
)

agent.print_response("25 乘以 4 等于多少？")
```

### 智能体团队

```python
from agno.agent import Agent
from agno.team.team import Team
from agno.models.openai import OpenAIChat
from agno.tools.duckduckgo import DuckDuckGoTools

# 创建研究员智能体
researcher = Agent(
    name="研究员",
    model=OpenAIChat(id="gpt-4o"),
    tools=[DuckDuckGoTools()],
    instructions="你是一位专业的研究员，擅长搜索和分析信息。"
)

# 创建作家智能体
writer = Agent(
    name="作家",
    model=OpenAIChat(id="gpt-4o"),
    instructions="你是一位专业的作家，擅长将研究结果整理成文章。"
)

# 创建团队
team = Team(
    name="内容创作团队",
    members=[researcher, writer],
    model=OpenAIChat(id="gpt-4o"),
    instructions="协作创建高质量的内容。"
)

team.print_response("写一篇关于人工智能最新发展的文章。")
```

### 工作流示例

```python
from agno.workflows import Workflow
from agno.workflows.step import Step

# 创建一个简单的工作流
workflow = Workflow(
    name="内容生成工作流",
    steps=[
        Step(name="研究", agent=researcher),
        Step(name="写作", agent=writer),
        Step(name="审核", agent=reviewer)
    ]
)

# 运行工作流
workflow.run("创建一篇关于机器学习的技术文章")
```

## 性能优势

在 Agno，我们专注于性能。为什么？因为即使是简单的 AI 工作流也可能产生数千个智能体。将其扩展到适度数量的用户，性能就成为瓶颈。Agno 专为构建高性能智能体系统而设计：

- 智能体实例化：平均约 3μs
- 内存占用：平均约 6.5KB

> 在 Apple M4 MacBook Pro 上测试。

虽然智能体的运行时间受推理速度限制，但我们必须尽一切可能最小化执行时间，减少内存使用，并并行化工具调用。这些数字乍一看可能微不足道，但我们的经验表明，即使在相当小的规模下，它们也会累积。

### 实例化时间对比

让我们测量一个带有 1 个工具的智能体启动所需的时间。我们将运行评估 1000 次以获得基线测量。

您应该在自己的机器上运行评估，请不要直接采信这些结果。

```shell
# 设置虚拟环境
./scripts/perf_setup.sh
source .venvs/perfenv/bin/activate
# 或手动安装依赖
# pip install openai agno langgraph langchain_openai

# Agno
python evals/performance/instantiation_with_tool.py

# LangGraph
python evals/performance/other/langgraph_instantiation.py
```

### 内存使用测量

为了测量内存使用，我们使用 `tracemalloc` 库。我们首先通过运行空函数计算基线内存使用，然后运行智能体 1000 次并计算差值。这给出了智能体内存使用的（相对）隔离测量。

我们建议您在自己的机器上运行评估，并深入研究代码以了解其工作原理。如果我们犯了错误，请告诉我们。

## 项目结构

```
agno/
├── libs/                    # 核心库
│   ├── agno/               # 主要库代码
│   │   ├── agent/          # 智能体实现
│   │   ├── team/           # 团队实现
│   │   ├── workflows/      # 工作流实现
│   │   ├── knowledge/      # 知识库
│   │   ├── tools/          # 工具集合
│   │   ├── models/         # 模型集成
│   │   └── os/             # AgentOS 运行时
│   └── agno_infra/         # 基础设施代码
├── cookbook/               # 示例和教程
│   ├── getting_started/    # 入门示例
│   ├── agents/            # 智能体示例
│   ├── teams/             # 团队示例
│   ├── workflows/         # 工作流示例
│   ├── knowledge/         # 知识库示例
│   ├── tools/             # 工具示例
│   ├── models/            # 模型示例
│   └── agent_os/          # AgentOS 示例
├── scripts/               # 脚本和工具
└── README.md              # 项目说明
```

## 部署

### Docker 部署

```bash
# 克隆项目
git clone https://github.com/agno-agi/agno.git
cd agno

# 使用 Docker 运行 PostgreSQL
./cookbook/scripts/run_pgvector.sh

# 使用 Docker 运行 Redis
./cookbook/scripts/run_redis.sh

# 运行示例应用
cd cookbook/demo
python run.py
```

### 生产环境部署

```yaml
# docker-compose.yml
version: '3.8'
services:
  agno-app:
    image: agno/agno:latest
    ports:
      - "7777:7777"
    environment:
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - DATABASE_URL=postgresql://agno:password@postgres:5432/agno
      - REDIS_URL=redis://redis:6379/0
    depends_on:
      - postgres
      - redis

  postgres:
    image: postgres:15
    environment:
      - POSTGRES_DB=agno
      - POSTGRES_USER=agno
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

## 学习路径

### 初学者路径

1. **基础概念理解**
   - 了解智能体、团队和工作流的基本概念
   - 理解 AgentOS 运行时的工作原理
   - 掌握基本的配置和设置

2. **简单示例练习**
   - 运行基础智能体示例
   - 尝试带工具的智能体
   - 探索会话管理功能

3. **进阶功能探索**
   - 知识库集成
   - 多模态输入输出
   - 自定义工具开发

### 有经验开发者路径

1. **深入架构理解**
   - 分析 AgentOS 内部组件的交互
   - 理解异步处理和并发模型
   - 探索 MCP 协议的高级用法

2. **性能优化**
   - 分析性能基准测试结果
   - 优化内存使用和实例化速度
   - 实现自定义缓存策略

3. **企业级部署**
   - 生产环境配置
   - 监控和日志配置
   - 安全和权限管理

## 贡献

我们欢迎贡献，请阅读我们的[贡献指南](https://github.com/agno-agi/agno/blob/v2.0/CONTRIBUTING.md)开始贡献。

## 遥测

Agno 记录智能体使用的模型，以便我们可以优先更新最受欢迎的提供商。您可以通过在环境中设置 `AGNO_TELEMETRY=false` 来禁用此功能。

## 许可证

本项目基于 Mozilla Public License 2.0 许可证开源。

## 支持

-