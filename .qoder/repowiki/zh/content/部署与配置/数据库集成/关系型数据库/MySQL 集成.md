# MySQL 集成

<cite>
**本文档中引用的文件**
- [mysql_for_agent.py](file://cookbook/db/mysql/mysql_for_agent.py)
- [mysql_for_team.py](file://cookbook/db/mysql/mysql_for_team.py)
- [mysql.py](file://libs/agno/agno/db/mysql/mysql.py)
- [schemas.py](file://libs/agno/agno/db/mysql/schemas.py)
- [utils.py](file://libs/agno/agno/db/mysql/utils.py)
- [base.py](file://libs/agno/agno/db/base.py)
- [memory.py](file://libs/agno/agno/db/schemas/memory.py)
- [evals.py](file://libs/agno/agno/db/schemas/evals.py)
- [knowledge.py](file://libs/agno/agno/db/schemas/knowledge.py)
- [README.md](file://cookbook/db/mysql/README.md)
- [run_mysql.sh](file://cookbook/scripts/run_mysql.sh)
</cite>

## 目录
1. [简介](#简介)
2. [项目结构](#项目结构)
3. [核心组件](#核心组件)
4. [架构概览](#架构概览)
5. [详细组件分析](#详细组件分析)
6. [依赖关系分析](#依赖关系分析)
7. [性能考虑](#性能考虑)
8. [故障排除指南](#故障排除指南)
9. [结论](#结论)

## 简介

MySQL 是 Agno 智能体框架中支持的数据持久化解决方案之一。通过 MySQL 数据库，开发者可以实现智能体的状态持久化、会话管理、记忆存储和交互历史记录等功能。本文档详细介绍了如何在 Agno 环境中配置和使用 MySQL 数据库，包括连接配置、表结构设计、存储引擎选择以及针对智能体应用的优化策略。

MySQL 集成为 Agno 提供了一个可靠的关系型数据库解决方案，特别适合需要事务一致性保证的应用场景。它支持复杂的查询操作、索引优化和连接池管理，能够满足智能体系统对数据持久化的各种需求。

## 项目结构

MySQL 集成功能在 Agno 项目中的组织结构如下：

```mermaid
graph TB
subgraph "Cookbook 示例"
A[mysql_for_agent.py]
B[mysql_for_team.py]
C[README.md]
end
subgraph "核心实现"
D[mysql.py]
E[schemas.py]
F[utils.py]
G[__init__.py]
end
subgraph "基础架构"
H[base.py]
I[memory.py]
J[evals.py]
K[knowledge.py]
end
subgraph "脚本工具"
L[run_mysql.sh]
M[run_mysql.bat]
end
A --> D
B --> D
D --> E
D --> F
D --> H
E --> I
E --> J
E --> K
L --> N[MySQL Docker 容器]
M --> N
```

**图表来源**
- [mysql_for_agent.py](file://cookbook/db/mysql/mysql_for_agent.py#L1-L18)
- [mysql_for_team.py](file://cookbook/db/mysql/mysql_for_team.py#L1-L60)
- [mysql.py](file://libs/agno/agno/db/mysql/mysql.py#L1-L50)

**章节来源**
- [mysql_for_agent.py](file://cookbook/db/mysql/mysql_for_agent.py#L1-L18)
- [mysql_for_team.py](file://cookbook/db/mysql/mysql_for_team.py#L1-L60)
- [README.md](file://cookbook/db/mysql/README.md#L1-L29)

## 核心组件

### MySQLDb 类

`MySQLDb` 类是 MySQL 集成的核心组件，继承自 `BaseDb` 抽象基类，提供了完整的数据库操作接口。

```python
class MySQLDb(BaseDb):
    def __init__(
        self,
        db_engine: Optional[Engine] = None,
        db_schema: Optional[str] = None,
        db_url: Optional[str] = None,
        session_table: Optional[str] = None,
        memory_table: Optional[str] = None,
        metrics_table: Optional[str] = None,
        eval_table: Optional[str] = None,
        knowledge_table: Optional[str] = None,
        id: Optional[str] = None,
    ):
```

该类支持多种初始化方式：
- 使用 `db_url` 参数直接指定连接字符串
- 使用 `db_engine` 参数传入现有的 SQLAlchemy 引擎
- 支持自定义表名和数据库模式

### 连接字符串语法

MySQL 连接字符串遵循标准格式：

```python
db_url = "mysql+pymysql://username:password@hostname:port/database"
```

支持的驱动程序：
- `pymysql`：Python 的纯 Python MySQL 驱动
- `mysqlconnector`：官方 MySQL Connector/Python
- `mysqldb`：MySQLdb 驱动

**章节来源**
- [mysql.py](file://libs/agno/agno/db/mysql/mysql.py#L35-L100)
- [mysql_for_agent.py](file://cookbook/db/mysql/mysql_for_agent.py#L6-L8)

## 架构概览

MySQL 集成采用分层架构设计，确保了良好的可扩展性和维护性：

```mermaid
graph TD
subgraph "应用层"
A[Agent]
B[Team]
C[Workflow]
end
subgraph "数据库抽象层"
D[MySQLDb]
E[BaseDb]
end
subgraph "数据访问层"
F[Session Management]
G[Memory Storage]
H[Metrics Collection]
I[Evaluation Storage]
J[Knowledge Base]
end
subgraph "MySQL 实现层"
K[SQLAlchemy ORM]
L[Connection Pool]
M[Transaction Management]
end
subgraph "MySQL 数据库"
N[Session Tables]
O[Memory Tables]
P[Metrics Tables]
Q[Eval Tables]
R[Knowledge Tables]
end
A --> D
B --> D
C --> D
D --> E
D --> F
D --> G
D --> H
D --> I
D --> J
F --> K
G --> K
H --> K
I --> K
J --> K
K --> L
K --> M
L --> N
L --> O
L --> P
L --> Q
L --> R
```

**图表来源**
- [mysql.py](file://libs/agno/agno/db/mysql/mysql.py#L35-L150)
- [base.py](file://libs/agno/agno/db/base.py#L15-L50)

## 详细组件分析

### 表结构设计

MySQL 集成支持五种主要表类型，每种都有专门的模式设计：

#### 会话表 (Sessions)

```mermaid
erDiagram
SESSIONS {
string session_id PK
string session_type
string agent_id FK
string team_id FK
string workflow_id FK
string user_id
json session_data
json agent_data
json team_data
json workflow_data
json metadata
json runs
json summary
bigint created_at
bigint updated_at
}
USERS {
string user_id PK
string username
string email
timestamp created_at
}
AGENTS {
string agent_id PK
string name
string model
timestamp created_at
}
TEAMS {
string team_id PK
string name
string model
timestamp created_at
}
WORKFLOWS {
string workflow_id PK
string name
string type
timestamp created_at
}
SESSIONS ||--|| USERS : belongs_to
SESSIONS }|--|| AGENTS : associated_with
SESSIONS }|--|| TEAMS : associated_with
SESSIONS }|--|| WORKFLOWS : associated_with
```

**图表来源**
- [schemas.py](file://libs/agno/agno/db/mysql/schemas.py#L10-L30)

#### 记忆表 (Memories)

```mermaid
erDiagram
MEMORIES {
string memory_id PK
json memory
text input
string agent_id FK
string team_id FK
string user_id FK
json topics
bigint updated_at
}
TOPICS {
string topic PK
string category
timestamp created_at
}
MEMORIES ||--o{ TOPICS : contains
```

**图表来源**
- [schemas.py](file://libs/agno/agno/db/mysql/schemas.py#L32-L45)

#### 指标表 (Metrics)

```mermaid
erDiagram
METRICS {
string id PK
bigint agent_runs_count
bigint team_runs_count
bigint workflow_runs_count
bigint agent_sessions_count
bigint team_sessions_count
bigint workflow_sessions_count
bigint users_count
json token_metrics
json model_metrics
date date
string aggregation_period
bigint created_at
bigint updated_at
boolean completed
}
```

**图表来源**
- [schemas.py](file://libs/agno/agno/db/mysql/schemas.py#L75-L95)

### 数据库操作方法

#### 会话管理

MySQLDb 提供了完整的会话生命周期管理：

```python
# 创建或更新会话
def upsert_session(self, session: Session, deserialize: Optional[bool] = True) -> Optional[Union[Session, Dict[str, Any]]]:

# 获取会话
def get_session(self, session_id: str, session_type: SessionType, user_id: Optional[str] = None, deserialize: Optional[bool] = True) -> Optional[Union[Session, Dict[str, Any]]]:

# 删除会话
def delete_session(self, session_id: str) -> bool:

# 重命名会话
def rename_session(self, session_id: str, session_type: SessionType, session_name: str, deserialize: Optional[bool] = True) -> Optional[Union[Session, Dict[str, Any]]]:
```

#### 内存管理

```python
# 存储用户记忆
def upsert_user_memory(self, memory: UserMemory, deserialize: Optional[bool] = True) -> Optional[Union[UserMemory, Dict[str, Any]]]:

# 获取用户记忆
def get_user_memory(self, memory_id: str, deserialize: Optional[bool] = True) -> Optional[UserMemory]:

# 删除用户记忆
def delete_user_memory(self, memory_id: str):

# 获取所有记忆主题
def get_all_memory_topics(self) -> List[str]:
```

**章节来源**
- [mysql.py](file://libs/agno/agno/db/mysql/mysql.py#L200-L400)
- [mysql.py](file://libs/agno/agno/db/mysql/mysql.py#L600-L800)

### 连接池和事务管理

MySQLDb 使用 SQLAlchemy 的 `scoped_session` 来管理连接池：

```python
# 初始化连接池
self.Session: scoped_session = scoped_session(sessionmaker(bind=self.db_engine))

# 事务处理
with self.Session() as sess, sess.begin():
    # 执行数据库操作
    stmt = mysql.insert(table).values(...)
    stmt = stmt.on_duplicate_key_update(...)
    sess.execute(stmt)
```

这种设计确保了：
- 连接的自动管理和复用
- 事务的原子性保证
- 线程安全的操作

**章节来源**
- [mysql.py](file://libs/agno/agno/db/mysql/mysql.py#L80-L120)

### 查询优化策略

#### 索引设计

MySQLDb 自动为关键字段创建索引：

```python
# 为会话类型创建索引
indexes.append("session_type")

# 为用户ID创建索引  
indexes.append("user_id")

# 为时间戳创建索引
indexes.append("created_at")
```

#### JSON 字段优化

对于 JSON 类型字段，使用 MySQL 的 JSON 函数进行高效查询：

```python
# MySQL JSON_EXTRACT 和 JSON_UNQUOTE 语法
stmt = stmt.where(
    func.coalesce(
        func.json_unquote(func.json_extract(table.c.session_data, "$.session_name")), ""
    ).ilike(f"%{session_name}%")
)
```

**章节来源**
- [mysql.py](file://libs/agno/agno/db/mysql/mysql.py#L150-L200)
- [utils.py](file://libs/agno/agno/db/mysql/utils.py#L25-L50)

## 依赖关系分析

MySQL 集成模块的依赖关系图展示了各组件之间的相互依赖：

```mermaid
graph TD
subgraph "外部依赖"
A[SQLAlchemy]
B[pymysql]
C[Pydantic]
end
subgraph "Agno 基础模块"
D[BaseDb]
E[Session]
F[UserMemory]
G[EvalRunRecord]
H[KnowledgeRow]
end
subgraph "MySQL 实现"
I[MySQLDb]
J[Schema Definitions]
K[Utility Functions]
end
subgraph "示例应用"
L[mysql_for_agent.py]
M[mysql_for_team.py]
end
A --> I
B --> I
C --> F
C --> G
C --> H
D --> I
E --> I
F --> I
G --> I
H --> I
J --> I
K --> I
I --> L
I --> M
```

**图表来源**
- [mysql.py](file://libs/agno/agno/db/mysql/mysql.py#L1-L30)
- [base.py](file://libs/agno/agno/db/base.py#L1-L20)

**章节来源**
- [mysql.py](file://libs/agno/agno/db/mysql/mysql.py#L1-L30)
- [schemas.py](file://libs/agno/agno/db/mysql/schemas.py#L1-L10)

## 性能考虑

### 存储引擎选择

推荐使用 InnoDB 存储引擎，因为它提供了以下优势：

1. **事务支持**：ACID 特性确保数据一致性
2. **行级锁定**：提高并发性能
3. **外键约束**：维护数据完整性
4. **崩溃恢复**：自动恢复机制

### 事务隔离级别

默认使用 MySQL 的 REPEATABLE READ 隔离级别，适合大多数智能体应用场景。如果需要更高的并发性，可以考虑：

```python
# 设置事务隔离级别
session.connection().execution_options(isolation_level="READ COMMITTED")
```

### 连接池配置

建议配置合理的连接池参数：

```python
# SQLAlchemy 连接池配置
engine = create_engine(
    db_url,
    pool_size=10,           # 连接池大小
    max_overflow=20,        # 最大溢出连接数
    pool_timeout=30,        # 连接超时时间
    pool_recycle=3600,      # 连接回收时间
    pool_pre_ping=True      # 连接前验证
)
```

### 查询优化技巧

1. **批量操作**：使用 `bulk_upsert_metrics` 进行批量插入
2. **索引优化**：为频繁查询的字段创建适当索引
3. **JSON 查询**：使用 MySQL 的 JSON 函数避免全表扫描
4. **分页查询**：合理使用 LIMIT 和 OFFSET 进行分页

## 故障排除指南

### 常见连接问题

#### 连接字符串错误

```python
# 错误示例
db_url = "mysql://user:pass@localhost:3306/db"  # 缺少驱动程序

# 正确示例
db_url = "mysql+pymysql://user:pass@localhost:3306/db"
```

#### 权限问题

确保 MySQL 用户具有适当的权限：

```sql
GRANT ALL PRIVILEGES ON ai.* TO 'ai'@'%';
FLUSH PRIVILEGES;
```

### 表结构问题

#### 表不存在

```python
# 自动创建表
db = MySQLDb(db_url=db_url, create_table_if_not_found=True)
```

#### 字段不匹配

检查表结构是否符合预期：

```python
# 验证表结构
is_valid_table(db_engine, table_name, table_type, db_schema)
```

### 性能问题诊断

#### 查询慢查询日志

启用慢查询日志：

```sql
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 1;
```

#### 分析查询执行计划

```sql
EXPLAIN SELECT * FROM agno_sessions WHERE user_id = 'user123';
```

**章节来源**
- [mysql.py](file://libs/agno/agno/db/mysql/mysql.py#L120-L180)
- [utils.py](file://libs/agno/agno/db/mysql/utils.py#L80-L120)

## 对比分析

### MySQL vs PostgreSQL

| 特性 | MySQL | PostgreSQL |
|------|-------|------------|
| 事务支持 | ACID 兼容 | 完全 ACID |
| JSON 支持 | 较好 | 优秀 |
| 复杂查询 | 良好 | 优秀 |
| 扩展性 | 中等 | 优秀 |
| 社区支持 | 广泛 | 活跃 |
| 性能 | 高 | 高 |
| 学习曲线 | 简单 | 中等 |

### 选择建议

**选择 MySQL 的场景：**
- 需要简单易用的关系型数据库
- 主要进行 CRUD 操作
- 需要快速部署和维护
- 团队熟悉 MySQL 技术栈

**选择 PostgreSQL 的场景：**
- 需要复杂查询和高级功能
- 需要 JSON 和数组类型支持
- 需要扩展性和灵活性
- 团队有 PostgreSQL 经验

## 结论

MySQL 集成为 Agno 智能体框架提供了可靠的数据持久化解决方案。通过合理的架构设计、完善的表结构和优化的查询策略，MySQL 能够很好地满足智能体应用对数据存储的各种需求。

关键优势包括：
- 完整的数据库操作接口
- 自动化的表结构管理
- 高效的连接池和事务处理
- 丰富的查询优化选项

建议在实际应用中根据具体需求选择合适的存储引擎和配置参数，并定期监控和优化数据库性能。