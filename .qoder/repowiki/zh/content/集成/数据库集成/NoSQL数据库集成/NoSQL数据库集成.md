
# NoSQL数据库集成

<cite>
**本文档中引用的文件**  
- [dynamo_for_agent.py](file://cookbook/db/dynamodb/dynamo_for_agent.py)
- [firestore_for_agent.py](file://cookbook/db/firestore/firestore_for_agent.py)
- [mongodb_for_agent.py](file://cookbook/db/mongo/mongodb_for_agent.py)
- [redis_for_agent.py](file://cookbook/db/redis/redis_for_agent.py)
- [singlestore_for_agent.py](file://cookbook/db/singlestore/singlestore_for_agent.py)
- [dynamo.py](file://libs/agno/agno/db/dynamo/dynamo.py)
- [firestore.py](file://libs/agno/agno/db/firestore/firestore.py)
- [mongo.py](file://libs/agno/agno/db/mongo/mongo.py)
- [redis.py](file://libs/agno/agno/db/redis/redis.py)
- [singlestore.py](file://libs/agno/agno/db/singlestore/singlestore.py)
- [base.py](file://libs/agno/agno/db/base.py)
</cite>

## 目录
1. [引言](#引言)
2. [核心NoSQL数据库支持](#核心nosql数据库支持)
3. [连接配置与初始化](#连接配置与初始化)
4. [数据模型设计](#数据模型设计)
5. [适配器实现机制](#适配器实现机制)
6. [在智能体、团队和工作流中的应用](#在智能体团队和工作流中的应用)
7. [性能调优建议](#性能调优建议)
8. [高可用性与故障恢复](#高可用性与故障恢复)
9. [最佳实践](#最佳实践)
10. [结论](#结论)

## 引言
Agno平台提供了一套全面的NoSQL数据库集成方案，支持DynamoDB、Firestore、MongoDB、Redis和SingleStore等多种数据库系统。本文档详细阐述了这些NoSQL数据库的集成方式，包括连接配置、数据模型设计、适配器实现机制以及在智能体、团队和工作流中的实际应用。通过统一的抽象层和适配器模式，Agno实现了对不同NoSQL数据库的无缝集成，为高速缓存、会话存储、用户记忆和非结构化数据管理提供了灵活高效的解决方案。

## 核心NoSQL数据库支持
Agno平台支持多种NoSQL数据库，每种数据库都有其独特的特性和适用场景。通过统一的`BaseDb`抽象基类，所有数据库适配器都实现了相同的核心接口，确保了API的一致性和可互换性。

```mermaid
graph TD
A[BaseDb] --> B[DynamoDb]
A --> C[FirestoreDb]
A --> D[MongoDb]
A --> E[RedisDb]
A --> F[SingleStoreDb]
A --> G[SqliteDb]
A --> H[InMemoryDb]
A --> I[JsonDb]
```

**Diagram sources**
- [base.py](file://libs/agno/agno/db/base.py#L24-L245)
- [dynamo.py](file://libs/agno/agno/db/dynamo/dynamo.py#L77-L78)
- [firestore.py](file://libs/agno/agno/db/firestore/firestore.py#L31-L32)
- [mongo.py](file://libs/agno/agno/db/mongo/mongo.py#L56-L57)
- [redis.py](file://libs/agno/agno/db/redis/redis.py#L42-L43)
- [singlestore.py](file://libs/agno/agno/db/singlestore/singlestore.py#L36-L37)

**Section sources**
- [base.py](file://libs/agno/agno/db/base.py#L24-L245)

## 连接配置与初始化
每种NoSQL数据库都有其特定的连接配置方式，Agno通过环境变量、URL连接字符串或直接传入客户端实例来建立数据库连接。

### DynamoDB连接配置
DynamoDB使用AWS凭证进行身份验证，可以通过环境变量或直接参数传递来配置。

```python
# 使用环境变量配置
db = DynamoDb()

# 或者显式传递凭证
db = DynamoDb(
    region_name="us-west-2",
    aws_access_key_id="your-access-key",
    aws_secret_access_key="your-secret-key"
)
```

**Section sources**
- [dynamo_for_agent.py](file://cookbook/db/dynamodb/dynamo_for_agent.py#L1-L22)
- [dynamo.py](file://libs/agno/agno/db/dynamo/dynamo.py#L42-L77)

### Firestore连接配置
Firestore利用Google Cloud的默认凭证机制，通常通过gcloud CLI配置。

```python
# 仅需指定项目ID
db = FirestoreDb(project_id="your-project-id")

# 或者传入已配置的客户端
from google.cloud import firestore
client = firestore.Client()
db = FirestoreDb(db_client=client)
```

**Section sources**
- [firestore_for_agent.py](file://cookbook/db/firestore/firestore_for_agent.py#L1-L32)
- [firestore.py](file://libs/agno/agno/db/firestore/firestore.py#L31-L61)

### MongoDB连接配置
MongoDB通过连接字符串URL进行配置，支持本地和远程实例。

```python
# 使用连接字符串
db_url = "mongodb://mongoadmin:secret@localhost:27017"
db = MongoDb(db_url=db_url)

# 或者传入已创建的客户端
from pymongo import MongoClient
client = MongoClient(db_url)
db = MongoDb(db_client=client)
```

**Section sources**
- [mongodb_for_agent.py](file://cookbook/db/mongo/mongodb_for_agent.py#L1-L36)
- [mongo.py](file://libs/agno/agno/db/mongo/mongo.py#L56-L86)

### Redis连接配置
Redis支持URL连接字符串或直接传入Redis客户端实例。

```python
# 使用连接URL
db = RedisDb(db_url="redis://localhost:6379")

# 或者传入现有客户端
import redis
client = redis.Redis(host='localhost', port=6379)
db = RedisDb(redis_client=client)
```

**Section sources**
- [redis_for_agent.py](file://cookbook/db/redis/redis_for_agent.py#L1-L41)
- [redis.py](file://libs/agno/agno/db/redis/redis.py#L42-L77)

### SingleStore连接配置
SingleStore使用SQLAlchemy风格的连接URL，基于MySQL协议。

```python
# 从环境变量构建连接URL
USERNAME = getenv("SINGLESTORE_USERNAME")
PASSWORD = getenv("SINGLESTORE_PASSWORD")
HOST = getenv("SINGLESTORE_HOST")
PORT = getenv("SINGLESTORE_PORT")
DATABASE = getenv("SINGLESTORE_DATABASE")

db_url = f"mysql+pymysql://{USERNAME}:{PASSWORD}@{HOST}:{PORT}/{DATABASE}?charset=utf8mb4"
db = SingleStoreDb(db_url=db_url)
```

**Section sources**
- [singlestore_for_agent.py](file://cookbook/db/singlestore/singlestore_for_agent.py#L1-L32)
- [singlestore.py](file://libs/agno/agno/db/singlestore/singlestore.py#L36-L66)

## 数据模型设计
Agno为每种NoSQL数据库设计了统一的数据模型，同时充分利用了各数据库的特性来优化存储和查询性能。

### 会话数据模型
会话数据是Agno平台的核心，存储了智能体、团队和工作流的运行状态。

```mermaid
erDiagram
SESSIONS {
string session_id PK
string session_type
string user_id
string agent_id
string team_id
string workflow_id
json runs
json agent_data
json team_data
json workflow_data
json session_data
json summary
json metadata
timestamp created_at
timestamp updated_at
}
SESSIONS ||--o{ AGENT_SESSION : "is"
SESSIONS ||--o{ TEAM_SESSION : "is"
SESSIONS ||--o{ WORKFLOW_SESSION : "is"
```

**Diagram sources**
- [base.py](file://libs/agno/agno/db/base.py#L24-L245)
- [dynamo.py](file://libs/agno/agno/db/dynamo/dynamo.py#L100-L101)
- [mongo.py](file://libs/agno/agno/db/mongo/mongo.py#L148-L149)
- [redis.py](file://libs/agno/agno/db/redis/redis.py#L100-L101)
- [singlestore.py](file://libs/agno/agno/db/singlestore/singlestore.py#L100-L101)

### 用户记忆数据模型
用户记忆存储了与特定用户相关的个性化信息和上下文。

```mermaid
erDiagram
MEMORIES {
string memory_id PK
string user_id
string agent_id
string team_id
string workflow_id
json memory
string[] topics
timestamp created_at
timestamp updated_at
}
MEMORIES ||--o{ USER_MEMORY : "contains"
USER ||--o{ MEMORIES : "has"
```

**Diagram sources**
- [base.py](file://libs/agno/agno/db/base.py#L24-L245)
- [dynamo.py](file://libs/agno/agno/db/dynamo/dynamo.py#L100-L101)
- [mongo.py](file://libs/agno/agno/db/mongo/mongo.py#L148-L149)
- [redis.py](file://libs/agno/agno/db/redis/redis.py#L100-L101)
- [singlestore.py](file://libs/agno/agno/db/singlestore/singlestore.py#L100-L101)

### 知识数据模型
知识数据模型用于存储非结构化的知识文档和内容。

```mermaid
erDiagram
KNOWLEDGE {
string id PK
string name
string content
json meta_data
json embedding
json usage
timestamp created_at
timestamp updated_at
}
KNOWLEDGE ||--o{ KNOWLEDGE_ROW : "represents"
```

**Diagram sources**
- [base.py](file://libs/agno/agno/db/base.py#L24-L245)
- [dynamo.py](file://libs/agno/agno/db/dynamo/dynamo.py#L100-L101)
- [singlestore.py](file://libs/agno/agno/db/singlestore/singlestore.py#L100-L101)

## 适配器实现机制
Agno的NoSQL数据库适配器通过统一的抽象层实现了对不同数据库系统的集成，同时针对每种数据库的特性进行了优化。

### 抽象基类设计
`BaseDb`抽象基类定义了所有数据库适配器必须实现的核心接口。

```mermaid
classDiagram
class BaseDb {
+str id
+str session_table_name
+str memory_table_name
+str metrics_table_name
+str eval_table_name
+str knowledge_table_name
+delete_session(session_id)
+delete_sessions(session_ids)
+get_session(session_id, session_type, user_id)
+get_sessions(session_type, user_id, component_id)
+rename_session(session_id, session_type, session_name)
+upsert_session(session)
+delete_user_memory(memory_id)
+delete_user_memories(memory_ids)
+get_all_memory_topics()
+get_user_memory(memory_id)
+get_user_memories(user_id, agent_id, team_id)
+get_user_memory_stats(limit, page)
+upsert_user_memory(memory)
+get_metrics(starting_date, ending_date)
+calculate_metrics()
+delete_knowledge_content(id)
+get_knowledge_content(id)
+get_knowledge_contents(limit, page)
+upsert_knowledge_content(knowledge_row)
+create_eval_run(eval_run)
+delete_eval_runs(eval_run_ids)
+get_eval_run(eval_run_id)
+get_eval_runs(limit, page, agent_id, team_id)
+rename_eval_run(eval_run_id, name)
}
class DynamoDb {
+boto3.client client
+_create_tables()
+_table_exists(table_name)
+_get_table(table_type)
}
class FirestoreDb {
+Client db_client
+project_id
}
class MongoDb {
+MongoClient db_client
+Database database
+_get_collection(table_type)
+_get_or_create_collection(collection_name)
}
class RedisDb {
+Redis redis_client
+str db_prefix
+int expire
+_store_record(table_type, record_id, data)
+_get_record(table_type, record_id)
+_delete_record(table_type, record_id)
+_get_all_records(table_type)
}
class SingleStoreDb {
+Engine db_engine
+str db_schema
+Session Session
+_create_table_structure_only(table_name, table_type)
+_create_table(table_name, table_type)
+_get_table(table_type)
+_get_or_create_table(table_name, table_type)
}
BaseDb <|-- DynamoDb
BaseDb <|-- FirestoreDb
BaseDb <|-- MongoDb
BaseDb <|-- RedisDb
BaseDb <|-- SingleStoreDb
```

**Diagram sources**
- [base.py](file://libs/agno/agno/db/base.py#L24-L245)
- [dynamo.py](file://libs/agno/agno/db/dynamo/dynamo.py#L77-L78)
- [firestore.py](file://libs/agno/agno/db/firestore/firestore.py#L31-L32)
- [mongo.py](file://libs/agno/agno/db/mongo/mongo.py#L56-L57)
- [redis.py](file://libs/agno/agno/db/redis/redis.py#L42-L43)
- [singlestore.py](file://libs/agno/agno/db/singlestore/singlestore.py#L36-L37)

### 数据序列化与反序列化
每种数据库适配器都实现了特定的数据序列化和反序列化逻辑，以适应其数据存储格式。

```mermaid
flowchart TD
A[Python对象] --> B{数据库类型}
B --> |DynamoDB| C[转换为DynamoDB格式]
B --> |Firestore| D[保持为Python字典]
B --> |MongoDB| E[保持为Python字典]
B --> |Redis| F[序列化为JSON字符串]
B --> |SingleStore| G[保持为Python字典]
C --> H[存储到DynamoDB]
D --> I[存储到Firestore]
E --> J[存储到MongoDB]
F --> K[存储到Redis]
G --> L[存储到SingleStore]
M[从数据库读取] --> N{数据库类型}
N --> |DynamoDB| O[从DynamoDB格式反序列化]
N --> |Firestore| P[直接使用Python字典]
N --> |MongoDB| Q[直接使用Python字典]
N --> |Redis| R[从JSON字符串反序列化]
N --> |SingleStore| S[直接使用Python字典]
O --> T[Python对象]
P --> T
Q --> T
R --> T
S --> T
```

**Diagram sources**
- [dynamo.py](file://libs/agno/agno/db/dynamo/dynamo.py#L100-L101)
- [mongo.py](file://libs/agno/agno/db/mongo/mongo.py#L148-L149)
- [redis.py](file://libs/agno/agno/db/redis/redis.py#L100-L101)
- [singlestore.py](file://libs/agno/agno/db/singlestore/singlestore.py#L100-L101)

### 索引策略
不同的NoSQL数据库采用了不同的索引策略来优化查询性能。

```mermaid
graph TD
A[索引策略] --> B[DynamoDB]
A --> C[MongoDB]
A --> D[Redis]
A --> E[SingleStore]
B --> F[全局二级索引<br>session_type-created_at-index]
B --> G[查询过滤器]
C --> H[集合索引<br>create_collection_indexes]
C --> I[字段索引<br>user_id, agent_id等]
D --> J[键模式索引<br>generate_redis_key]
D --> K[字段索引<br>create_index_entries]
E --> L[SQL索引<br>CREATE INDEX]
E --> M[唯一约束<br>UNIQUE KEY]
E --> N[分片键<br>SHARD KEY]
```

**Diagram sources**
- [dynamo.py](file://libs/agno/agno/db/dynamo/dynamo.py#L100-L101)
- [mongo.py](file://libs/agno/agno/db/mongo/mongo.py#L148-L149)
- [redis.py](file://libs/agno/agno/db/redis/redis.py#L100-L101)
- [singlestore.py](file://libs/agno/agno/db/singlestore/singlestore.py#L100-L101)

### 查询模式
Agno实现了统一的查询接口，但针对不同数据库的查询能力进行了适配。

```mermaid
sequenceDiagram
    participant App as 应用程序
    participant Agno as Agno框架
    participant Db as 数据库适配器
    
    App->>Agno: get_sessions(session_type=AGENT, user_id="123")
    Agno->>Db: 调用具体数据库的get_sessions方法
    alt DynamoDB
        Db->>Db: 使用GSI查询session_type-created_at-index
        Db->>Db: 应用用户ID过滤器
    else MongoDB