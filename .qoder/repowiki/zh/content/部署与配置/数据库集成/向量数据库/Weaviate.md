# Weaviate 向量数据库集成文档

<cite>
**本文档引用的文件**
- [weaviate.py](file://libs/agno/agno/vectordb/weaviate/weaviate.py)
- [index.py](file://libs/agno/agno/vectordb/weaviate/index.py)
- [__init__.py](file://libs/agno/agno/vectordb/weaviate/__init__.py)
- [test_weaviatedb.py](file://libs/agno/tests/unit/vectordb/test_weaviatedb.py)
- [weaviate_db_upsert.py](file://cookbook/knowledge/vector_db/weaviate_db/weaviate_db_upsert.py)
- [weaviate_db_hybrid_search.py](file://cookbook/knowledge/vector_db/weaviate_db/weaviate_db_hybrid_search.py)
- [async_weaviate_db.py](file://cookbook/knowledge/vector_db/weaviate_db/async_weaviate_db.py)
- [filtering_weaviate.py](file://cookbook/knowledge/filters/vector_dbs/filtering_weaviate.py)
- [run_weaviate.sh](file://cookbook/scripts/run_weaviate.sh)
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

Weaviate 是一个现代化的向量数据库，专为 AI 应用程序设计。在 Agno 框架中，Weaviate 被用作强大的向量存储解决方案，支持混合搜索（向量相似度搜索和关键词搜索）、结构化数据存储以及灵活的元数据过滤功能。

Weaviate 的主要特性包括：
- **向量存储**：支持高维向量的高效存储和检索
- **混合搜索**：结合向量相似度和关键词搜索的优势
- **结构化数据**：支持文本、元数据等结构化信息存储
- **模块化架构**：可插拔的嵌入模型支持
- **GraphQL 接口**：强大的查询语言支持复杂查询
- **集群部署**：支持分布式部署和扩展

## 项目结构

Weaviate 集成在 Agno 项目中的组织结构如下：

```mermaid
graph TB
subgraph "Weaviate 集成模块"
A[weaviate.py<br/>主实现文件] --> B[index.py<br/>枚举定义]
A --> C[__init__.py<br/>导出模块]
subgraph "测试文件"
D[test_weaviatedb.py<br/>单元测试]
end
subgraph "示例文件"
E[weaviate_db_upsert.py<br/>基础使用示例]
F[weaviate_db_hybrid_search.py<br/>混合搜索示例]
G[async_weaviate_db.py<br/>异步操作示例]
H[filtering_weaviate.py<br/>过滤功能示例]
end
subgraph "部署脚本"
I[run_weaviate.sh<br/>本地部署脚本]
end
end
```

**图表来源**
- [weaviate.py](file://libs/agno/agno/vectordb/weaviate/weaviate.py#L1-L50)
- [index.py](file://libs/agno/agno/vectordb/weaviate/index.py#L1-L16)

**章节来源**
- [weaviate.py](file://libs/agno/agno/vectordb/weaviate/weaviate.py#L1-L935)
- [index.py](file://libs/agno/agno/vectordb/weaviate/index.py#L1-L16)

## 核心组件

### Weaviate 类

Weaviate 类是整个集成的核心，继承自 VectorDb 基类，提供了完整的向量数据库操作功能。

```python
class Weaviate(VectorDb):
    """
    Weaviate 类用于管理与 Weaviate 向量数据库的向量操作（v4 客户端）。
    """
```

### 关键参数配置

Weaviate 配置包含多个关键参数：

- **连接参数**：
  - `wcd_url`: Weaviate Cloud Deployment URL
  - `wcd_api_key`: Weaviate Cloud API 密钥
  - `client`: 自定义 Weaviate 客户端实例
  - `local`: 是否使用本地实例

- **集合参数**：
  - `collection`: 数据集名称，默认为 "default"
  - `vector_index`: 向量索引类型（HNSW、FLAT、DYNAMIC）
  - `distance`: 距离度量方式（COSINE、DOT、L2_SQUARED）

- **搜索参数**：
  - `search_type`: 搜索类型（vector、keyword、hybrid）
  - `reranker`: 重排序器
  - `hybrid_search_alpha`: 混合搜索权重（0.0-1.0）

**章节来源**
- [weaviate.py](file://libs/agno/agno/vectordb/weaviate/weaviate.py#L29-L69)

## 架构概览

Weaviate 在 Agno 中的架构采用分层设计，确保了良好的可扩展性和维护性：

```mermaid
graph TB
subgraph "应用层"
A[Agent/Team<br/>智能代理] --> B[Knowledge<br/>知识库]
B --> C[VectorDb<br/>向量数据库接口]
end
subgraph "Weaviate 层"
C --> D[Weaviate 类]
D --> E[客户端管理]
D --> F[搜索引擎]
D --> G[数据操作]
end
subgraph "存储层"
E --> H[Weaviate Cloud]
E --> I[本地 Weaviate 实例]
F --> J[向量索引]
F --> K[文本索引]
G --> L[文档存储]
end
subgraph "外部依赖"
M[Embedder<br/>嵌入模型] --> D
N[Reranker<br/>重排序器] --> D
end
```

**图表来源**
- [weaviate.py](file://libs/agno/agno/vectordb/weaviate/weaviate.py#L29-L148)

## 详细组件分析

### 向量索引配置

Weaviate 支持多种向量索引类型，每种都有不同的性能特征：

```mermaid
classDiagram
class VectorIndex {
<<enumeration>>
HNSW
FLAT
DYNAMIC
}
class Distance {
<<enumeration>>
COSINE
DOT
L2_SQUARED
HAMMING
MANHATTAN
}
class Weaviate {
+VectorIndex vector_index
+Distance distance
+get_vector_index_config(index_type, distance_metric)
+create()
+exists()
+drop()
}
Weaviate --> VectorIndex : 使用
Weaviate --> Distance : 使用
```

**图表来源**
- [index.py](file://libs/agno/agno/vectordb/weaviate/index.py#L3-L15)
- [weaviate.py](file://libs/agno/agno/vectordb/weaviate/weaviate.py#L759-L775)

### 搜索功能实现

Weaviate 提供三种主要的搜索类型：

#### 1. 向量搜索（Vector Search）

```mermaid
sequenceDiagram
participant Client as 客户端
participant Weaviate as Weaviate 类
participant Embedder as 嵌入模型
participant WeaviateDB as Weaviate 数据库
Client->>Weaviate : vector_search(query)
Weaviate->>Embedder : get_embedding(query)
Embedder-->>Weaviate : query_embedding
Weaviate->>WeaviateDB : near_vector(near_vector, limit)
WeaviateDB-->>Weaviate : search_results
Weaviate->>Weaviate : get_search_results(response)
Weaviate-->>Client : List[Document]
```

**图表来源**
- [weaviate.py](file://libs/agno/agno/vectordb/weaviate/weaviate.py#L433-L470)

#### 2. 关键词搜索（Keyword Search）

关键词搜索使用 BM25 算法进行文本匹配：

```mermaid
flowchart TD
A[接收查询] --> B[构建过滤表达式]
B --> C[执行 BM25 查询]
C --> D[返回匹配结果]
D --> E[解析搜索结果]
E --> F[应用重排序器]
F --> G[返回最终结果]
```

**图表来源**
- [weaviate.py](file://libs/agno/agno/vectordb/weaviate/weaviate.py#L520-L555)

#### 3. 混合搜索（Hybrid Search）

混合搜索结合向量和关键词搜索的优势：

```mermaid
sequenceDiagram
participant Client as 客户端
participant Weaviate as Weaviate 类
participant Embedder as 嵌入模型
participant WeaviateDB as Weaviate 数据库
Client->>Weaviate : hybrid_search(query)
Weaviate->>Embedder : get_embedding(query)
Embedder-->>Weaviate : query_embedding
Weaviate->>WeaviateDB : hybrid(query, vector, alpha)
Note over WeaviateDB : alpha 控制向量 vs 关键词权重
WeaviateDB-->>Weaviate : hybrid_results
Weaviate->>Weaviate : get_search_results(response)
Weaviate-->>Client : List[Document]
```

**图表来源**
- [weaviate.py](file://libs/agno/agno/vectordb/weaviate/weaviate.py#L600-L635)

**章节来源**
- [weaviate.py](file://libs/agno/agno/vectordb/weaviate/weaviate.py#L433-L635)

### 数据操作功能

Weaviate 提供了完整的 CRUD 操作：

#### 文档插入和更新

```mermaid
flowchart TD
A[插入文档] --> B[生成内容哈希]
B --> C[检查内容是否存在]
C --> D{存在?}
D --> |是| E[删除现有文档]
D --> |否| F[准备文档属性]
E --> F
F --> G[清理内容]
G --> H[序列化元数据]
H --> I[计算 UUID]
I --> J[调用 Weaviate 插入 API]
J --> K[记录日志]
```

**图表来源**
- [weaviate.py](file://libs/agno/agno/vectordb/weaviate/weaviate.py#L220-L270)

#### 异步操作支持

Weaviate 完全支持异步操作，提高并发性能：

```python
async def async_insert(self, content_hash: str, documents: List[Document], filters: Optional[Dict[str, Any]] = None):
    """异步插入文档到 Weaviate"""
    # 并行处理文档嵌入
    embed_tasks = [document.async_embed(embedder=self.embedder) for document in documents]
    await asyncio.gather(*embed_tasks, return_exceptions=True)
    
    # 批量插入文档
    for document in documents:
        # 处理每个文档...
        await collection.data.insert(properties=properties, vector=document.embedding, uuid=doc_uuid)
```

**章节来源**
- [weaviate.py](file://libs/agno/agno/vectordb/weaviate/weaviate.py#L220-L320)

### 元数据过滤系统

Weaviate 支持强大的元数据过滤功能：

```mermaid
classDiagram
class MetadataFilter {
+build_filter_expression(filters)
+delete_by_metadata(metadata)
+name_exists(name)
+content_hash_exists(hash)
}
class Weaviate {
+delete_by_name(name)
+delete_by_content_id(content_id)
+delete_by_content_hash(content_hash)
+_build_filter_expression(filters)
}
Weaviate --> MetadataFilter : 使用
```

**图表来源**
- [weaviate.py](file://libs/agno/agno/vectordb/weaviate/weaviate.py#L370-L420)

**章节来源**
- [weaviate.py](file://libs/agno/agno/vectordb/weaviate/weaviate.py#L370-L420)

## 依赖关系分析

Weaviate 集成的依赖关系图展示了各组件之间的交互：

```mermaid
graph TB
subgraph "外部依赖"
A[weaviate-client<br/>Weaviate SDK]
B[agno.knowledge.embedder<br/>嵌入模型]
C[agno.knowledge.document<br/>文档对象]
D[agno.vectordb.base<br/>向量数据库基类]
end
subgraph "内部模块"
E[Weaviate 类]
F[VectorIndex 枚举]
G[Distance 枚举]
H[SearchType 枚举]
end
subgraph "测试依赖"
I[pytest<br/>测试框架]
J[unittest.mock<br/>模拟工具]
end
A --> E
B --> E
C --> E
D --> E
F --> E
G --> E
H --> E
E --> I
E --> J
```

**图表来源**
- [weaviate.py](file://libs/agno/agno/vectordb/weaviate/weaviate.py#L1-L25)

**章节来源**
- [weaviate.py](file://libs/agno/agno/vectordb/weaviate/weaviate.py#L1-L25)

## 性能考虑

### 向量索引优化

Weaviate 提供多种向量索引类型以适应不同的性能需求：

- **HNSW（Hierarchical Navigable Small World）**：
  - 适用于大多数场景
  - 支持动态调整参数
  - 内存占用适中

- **FLAT**：
  - 精确搜索
  - 无近似误差
  - 内存占用较高

- **DYNAMIC**：
  - 自动选择最佳索引类型
  - 适合不确定的使用场景

### 搜索性能优化

1. **混合搜索权重调节**：
   ```python
   # 调整向量 vs 关键词搜索的平衡
   hybrid_search_alpha=0.6  # 60% 向量搜索，40% 关键词搜索
   ```

2. **批量操作**：
   - 使用异步操作提高并发性能
   - 批量插入减少网络开销

3. **连接池管理**：
   - 单例模式管理客户端连接
   - 自动重连机制

## 故障排除指南

### 常见问题及解决方案

#### 1. 连接问题

**问题**：无法连接到 Weaviate 实例
**解决方案**：
```python
# 检查环境变量
export WCD_URL="your-cluster-url"
export WCD_API_KEY="your-api-key"

# 或者直接在代码中指定
vector_db = Weaviate(
    wcd_url="your-cluster-url",
    wcd_api_key="your-api-key",
    local=False
)
```

#### 2. 向量维度不匹配

**问题**：嵌入向量维度与索引配置不匹配
**解决方案**：
```python
# 确保嵌入模型输出维度与索引配置一致
from agno.knowledge.embedder.openai import OpenAIEmbedder
embedder = OpenAIEmbedder(dimensions=1536)  # 根据需要调整
```

#### 3. 搜索结果为空

**问题**：搜索返回空结果
**解决方案**：
- 检查文档是否正确插入
- 验证嵌入模型是否正常工作
- 调整搜索阈值或增加搜索范围

**章节来源**
- [weaviate.py](file://libs/agno/agno/vectordb/weaviate/weaviate.py#L70-L148)

## 结论

Weaviate 在 Agno 框架中的集成提供了强大而灵活的向量数据库解决方案。通过支持混合搜索、结构化数据存储和元数据过滤，Weaviate 能够满足各种 AI 应用场景的需求。

### 主要优势

1. **功能完整**：支持向量搜索、关键词搜索和混合搜索
2. **易于使用**：简洁的 API 设计和丰富的示例
3. **高性能**：支持异步操作和多种索引类型
4. **可扩展**：支持集群部署和水平扩展
5. **灵活配置**：丰富的配置选项适应不同需求

### 最佳实践建议

1. **选择合适的索引类型**：根据数据规模和查询需求选择 HNSW、FLAT 或 DYNAMIC
2. **合理设置混合搜索权重**：根据应用场景调整向量和关键词搜索的平衡
3. **使用异步操作**：在高并发场景下使用异步 API 提高性能
4. **定期监控和优化**：关注查询性能和资源使用情况
5. **备份重要数据**：定期备份知识库内容

Weaviate 的模块化架构和 GraphQL 接口使其成为现代 AI 应用的理想选择，特别是在需要同时处理向量数据和结构化数据的场景中。