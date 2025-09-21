# MongoDB集成

<cite>
**本文档中引用的文件**  
- [mongodb_for_agent.py](file://cookbook/db/mongo/mongodb_for_agent.py)
- [mongodb_for_team.py](file://cookbook/db/mongo/mongodb_for_team.py)
- [mongodb.py](file://libs/agno/agno/vectordb/mongodb/mongodb.py)
- [README.md](file://cookbook/db/mongo/README.md)
- [run_mongodb.sh](file://cookbook/scripts/run_mongodb.sh)
</cite>

## 目录
1. [简介](#简介)
2. [项目结构](#项目结构)
3. [核心组件](#核心组件)
4. [架构概述](#架构概述)
5. [详细组件分析](#详细组件分析)
6. [依赖分析](#依赖分析)
7. [性能考虑](#性能考虑)
8. [故障排除指南](#故障排除指南)
9. [结论](#结论)

## 简介
本文档详细介绍了如何在Agno系统中配置和使用MongoDB作为持久化存储。重点分析了`mongodb_for_agent.py`和`mongodb_for_team.py`中的实现细节，包括连接管理、数据序列化和CRUD操作。文档还详细说明了MongoDB集合的设计模式，如会话、智能体和团队数据的存储结构。提供了索引创建的最佳实践以优化查询性能，并讨论了MongoDB在处理分层文档数据方面的优势。此外，还包括错误处理、连接池配置和数据一致性策略的示例。

## 项目结构
Agno系统的MongoDB集成主要位于`cookbook/db/mongo`目录下，包含两个核心文件：`mongodb_for_agent.py`和`mongodb_for_team.py`。这些文件演示了如何将MongoDB用作智能体和团队的数据库。此外，`libs/agno/agno/vectordb/mongodb/mongodb.py`文件包含了MongoDB向量数据库的具体实现。

```mermaid
graph TD
cookbook.db.mongo[mongo]:::directory
cookbook.db.mongo.mongodb_for_agent[mongodb_for_agent.py]:::file
cookbook.db.mongo.mongodb_for_team[mongodb_for_team.py]:::file
cookbook.db.mongo.README[README.md]:::file
libs.agno.agno.vectordb.mongodb[mongodb]:::directory
libs.agno.agno.vectordb.mongodb.mongodb[mongodb.py]:::file
cookbook.scripts[scripts]:::directory
cookbook.scripts.run_mongodb[run_mongodb.sh]:::file
cookbook.db.mongo --> cookbook.db.mongo.mongodb_for_agent
cookbook.db.mongo --> cookbook.db.mongo.mongodb_for_team
cookbook.db.mongo --> cookbook.db.mongo.README
libs.agno.agno.vectordb.mongodb --> libs.agno.agno.vectordb.mongodb.mongodb
cookbook.scripts --> cookbook.scripts.run_mongodb
```

**图示来源**
- [mongodb_for_agent.py](file://cookbook/db/mongo/mongodb_for_agent.py)
- [mongodb_for_team.py](file://cookbook/db/mongo/mongodb_for_team.py)
- [mongodb.py](file://libs/agno/agno/vectordb/mongodb/mongodb.py)
- [run_mongodb.sh](file://cookbook/scripts/run_mongodb.sh)

**本节来源**
- [mongodb_for_agent.py](file://cookbook/db/mongo/mongodb_for_agent.py)
- [mongodb_for_team.py](file://cookbook/db/mongo/mongodb_for_team.py)
- [mongodb.py](file://libs/agno/agno/vectordb/mongodb/mongodb.py)

## 核心组件
Agno系统中的MongoDB集成主要由以下几个核心组件构成：
- `MongoDb`类：提供MongoDB向量数据库的具体实现，支持向量搜索、关键字搜索和混合搜索。
- `Agent`类：表示一个智能体，可以使用MongoDB作为其持久化存储。
- `Team`类：表示一个团队，可以使用MongoDB作为其持久化存储。
- `Document`类：表示一个文档，包含内容、元数据和嵌入向量。

这些组件共同工作，实现了智能体和团队的数据持久化和检索功能。

**本节来源**
- [mongodb.py](file://libs/agno/agno/vectordb/mongodb/mongodb.py)
- [mongodb_for_agent.py](file://cookbook/db/mongo/mongodb_for_agent.py)
- [mongodb_for_team.py](file://cookbook/db/mongo/mongodb_for_team.py)

## 架构概述
Agno系统的MongoDB集成架构如下图所示：

```mermaid
graph TD
subgraph "Agno系统"
Agent[智能体]
Team[团队]
Document[文档]
end
subgraph "MongoDB"
Collection[集合]
Index[索引]
DocumentStorage[文档存储]
end
Agent --> |使用| MongoDb
Team --> |使用| MongoDb
MongoDb --> |管理| Collection
MongoDb --> |创建| Index
Document --> |存储| DocumentStorage
MongoDb --> |操作| DocumentStorage
```

**图示来源**
- [mongodb.py](file://libs/agno/agno/vectordb/mongodb/mongodb.py)
- [mongodb_for_agent.py](file://cookbook/db/mongo/mongodb_for_agent.py)
- [mongodb_for_team.py](file://cookbook/db/mongo/mongodb_for_team.py)

## 详细组件分析
### MongoDb类分析
`MongoDb`类是Agno系统中MongoDB集成的核心，提供了向量数据库的所有功能。

#### 类图
```mermaid
classDiagram
class MongoDb {
+collection_name : str
+database : str
+search_index_name : str
+cosmos_compatibility : bool
+search_type : SearchType
+hybrid_vector_weight : float
+hybrid_keyword_weight : float
+hybrid_rank_constant : int
+embedder : Embedder
+distance_metric : str
+connection_string : str
+overwrite : bool
+wait_until_index_ready_in_seconds : float
+wait_after_insert_in_seconds : float
+kwargs : dict
+_client : MongoClient
+_db : Database
+_collection : Collection
+_async_client : AsyncMongoClient
+_async_db : Database
+_async_collection : Collection
+__init__(collection_name : str, db_url : Optional[str], database : str, embedder : Optional[Embedder], distance_metric : str, overwrite : bool, wait_until_index_ready_in_seconds : Optional[float], wait_after_insert_in_seconds : Optional[float], max_pool_size : int, retry_writes : bool, client : Optional[MongoClient], search_index_name : Optional[str], cosmos_compatibility : Optional[bool], search_type : SearchType, hybrid_vector_weight : float, hybrid_keyword_weight : float, hybrid_rank_constant : int, **kwargs)
+_get_client() MongoClient
+_get_async_client() AsyncMongoClient
+_get_or_create_collection() Collection
+_get_collection() Collection
+_get_async_collection() Collection
+_create_search_index(overwrite : bool) None
+_create_search_index_async() None
+_search_index_exists() bool
+_wait_for_index_ready() None
+_wait_for_index_ready_async() None
+collection_exists() bool
+create() None
+async_create() None
+doc_exists(document : Document) bool
+name_exists(name : str) bool
+id_exists(id : str) bool
+content_hash_exists(content_hash : str) bool
+insert(content_hash : str, documents : List[Document], filters : Optional[Dict[str, Any]]) None
+upsert(content_hash : str, documents : List[Document], filters : Optional[Dict[str, Any]]) None
+upsert_available() bool
+search(query : str, limit : int, filters : Optional[Dict[str, Any]], min_score : float) List[Document]
+vector_search(query : str, limit : int) List[Document]
+keyword_search(query : str, limit : int) List[Document]
+hybrid_search(query : str, limit : int, filters : Optional[Dict[str, Any]]) List[Document]
+drop() None
+exists() bool
+optimize() None
+delete() bool
+prepare_doc(content_hash : str, document : Document, filters : Optional[Dict[str, Any]]) Dict[str, Any]
+get_count() int
+async_doc_exists(document : Document) bool
}
```

**图示来源**
- [mongodb.py](file://libs/agno/agno/vectordb/mongodb/mongodb.py)

#### 连接管理
`MongoDb`类通过`_get_client`和`_get_async_client`方法管理MongoDB连接。连接字符串、最大连接池大小和重试写入等参数可以在初始化时配置。

```python
def __init__(
    self,
    collection_name: str,
    db_url: Optional[str] = "mongodb://localhost:27017/",
    database: str = "agno",
    embedder: Optional[Embedder] = None,
    distance_metric: str = Distance.cosine,
    overwrite: bool = False,
    wait_until_index_ready_in_seconds: Optional[float] = 3,
    wait_after_insert_in_seconds: Optional[float] = 3,
    max_pool_size: int = 100,
    retry_writes: bool = True,
    client: Optional[MongoClient] = None,
    search_index_name: Optional[str] = "vector_index_1",
    cosmos_compatibility: Optional[bool] = False,
    search_type: SearchType = SearchType.vector,
    hybrid_vector_weight: float = 0.5,
    hybrid_keyword_weight: float = 0.5,
    hybrid_rank_constant: int = 60,
    **kwargs,
):
```

**本节来源**
- [mongodb.py](file://libs/agno/agno/vectordb/mongodb/mongodb.py#L45-L110)

#### 数据序列化
`MongoDb`类通过`prepare_doc`方法将`Document`对象序列化为MongoDB文档。文档ID使用内容的MD5哈希值生成，确保唯一性。

```python
def prepare_doc(
    self, content_hash: str, document: Document, filters: Optional[Dict[str, Any]] = None
) -> Dict[str, Any]:
    # Add filters to document metadata if provided
    if filters:
        meta_data = document.meta_data.copy() if document.meta_data else {}
        meta_data.update(filters)
        document.meta_data = meta_data

    cleaned_content = document.content.replace("\x00", "\ufffd")
    doc_id = md5(cleaned_content.encode("utf-8")).hexdigest()
    doc_data = {
        "_id": doc_id,
        "name": document.name,
        "content": cleaned_content,
        "meta_data": document.meta_data,
        "embedding": document.embedding,
        "content_id": document.content_id,
        "content_hash": content_hash,
    }
    log_debug(f"Prepared document: {doc_data['_id']}")
    return doc_data
```

**本节来源**
- [mongodb.py](file://libs/agno/agno/vectordb/mongodb/mongodb.py#L965-L985)

#### CRUD操作
`MongoDb`类提供了完整的CRUD操作支持，包括插入、更新、删除和查询。

```python
def insert(self, content_hash: str, documents: List[Document], filters: Optional[Dict[str, Any]] = None) -> None:
    """Insert documents into the MongoDB collection."""
    log_debug(f"Inserting {len(documents)} documents")
    collection = self._get_collection()

    prepared_docs = []
    for document in documents:
        try:
            document.embed(embedder=self.embedder)
            if document.embedding is None:
                raise ValueError(f"Failed to generate embedding for document: {document.id}")
            doc_data = self.prepare_doc(content_hash, document, filters)
            prepared_docs.append(doc_data)
        except ValueError as e:
            logger.error(f"Error preparing document '{document.name}': {e}")

    if prepared_docs:
        try:
            collection.insert_many(prepared_docs, ordered=False)
            log_info(f"Inserted {len(prepared_docs)} documents successfully.")
            if self.wait_after_insert_in_seconds and self.wait_after_insert_in_seconds > 0:
                time.sleep(self.wait_after_insert_in_seconds)
        except errors.BulkWriteError as e:
            logger.warning(f"Bulk write error while inserting documents: {e.details}")
        except Exception as e:
            logger.error(f"Error inserting documents: {e}")
```

**本节来源**
- [mongodb.py](file://libs/agno/agno/vectordb/mongodb/mongodb.py#L517-L545)

### 智能体和团队集成分析
`mongodb_for_agent.py`和`mongodb_for_team.py`文件演示了如何将MongoDB集成到智能体和团队中。

#### 智能体集成
```python
from agno.agent import Agent
from agno.db.mongo import MongoDb
from agno.tools.duckduckgo import DuckDuckGoTools

# MongoDB连接设置
db_url = "mongodb://mongoadmin:secret@localhost:27017"

db = MongoDb(db_url=db_url)

agent = Agent(
    db=db,
    tools=[DuckDuckGoTools()],
    add_history_to_context=True,
)
agent.print_response("How many people live in Canada?")
agent.print_response("What is their national anthem called?")
```

**本节来源**
- [mongodb_for_agent.py](file://cookbook/db/mongo/mongodb_for_agent.py#L25-L36)

#### 团队集成
```python
from typing import List

from agno.agent import Agent
from agno.db.mongo import MongoDb
from agno.models.openai import OpenAIChat
from agno.team import Team
from agno.tools.duckduckgo import DuckDuckGoTools
from agno.tools.hackernews import HackerNewsTools
from pydantic import BaseModel

# MongoDB连接设置
db_url = "mongodb://mongoadmin:secret@localhost:27017"
db = MongoDb(db_url=db_url)


class Article(BaseModel):
    title: str
    summary: str
    reference_links: List[str]


hn_researcher = Agent(
    name="HackerNews Researcher",
    model=OpenAIChat("gpt-4o"),
    role="Gets top stories from hackernews.",
    tools=[HackerNewsTools()],
)

web_searcher = Agent(
    name="Web Searcher",
    model=OpenAIChat("gpt-4o"),
    role="Searches the web for information on a topic",
    tools=[DuckDuckGoTools()],
    add_datetime_to_context=True,
)


hn_team = Team(
    name="HackerNews Team",
    model=OpenAIChat("gpt-4o"),
    members=[hn_researcher, web_searcher],
    db=db,
    instructions=[
        "First, search hackernews for what the user is asking about.",
        "Then, ask the web searcher to search for each story to get more information.",
        "Finally, provide a thoughtful and engaging summary.",
    ],
    output_schema=Article,
    markdown=True,
    show_members_responses=True,
    add_member_tools_to_context=False,
)

hn_team.print_response("Write an article about the top 2 stories on hackernews")
```

**本节来源**
- [mongodb_for_team.py](file://cookbook/db/mongo/mongodb_for_team.py#L25-L74)

## 依赖分析
Agno系统的MongoDB集成依赖于以下几个关键组件：
- `pymongo`：用于与MongoDB进行交互。
- `bson`：用于处理MongoDB的二进制JSON格式。
- `hashlib`：用于生成文档ID的MD5哈希值。
- `agno`：Agno系统的核心库，提供智能体、团队和文档等组件。

```mermaid
graph TD
pymongo[pymongo]:::dependency
bson[bson]:::dependency
hashlib[hashlib]:::dependency
agno[agno]:::dependency
cookbook.db.mongo.mongodb_for_agent[mongodb_for_agent.py]:::file
cookbook.db.mongo.mongodb_for_team[mongodb_for_team.py]:::file
libs.agno.agno.vectordb.mongodb.mongodb[mongodb.py]:::file
cookbook.db.mongo.mongodb_for_agent --> pymongo
cookbook.db.mongo.mongodb_for_agent --> bson
cookbook.db.mongo.mongodb_for_agent --> hashlib
cookbook.db.mongo.mongodb_for_agent --> agno
cookbook.db.mongo.mongodb_for_team --> pymongo
cookbook.db.mongo.mongodb_for_team --> bson
cookbook.db.mongo.mongodb_for_team --> hashlib
cookbook.db.mongo.mongodb_for_team --> agno
libs.agno.agno.vectordb.mongodb.mongodb --> pymongo
libs.agno.agno.vectordb.mongodb.mongodb --> bson
libs.agno.agno.vectordb.mongodb.mongodb --> hashlib
libs.agno.agno.vectordb.mongodb.mongodb --> agno
```

**图示来源**
- [mongodb.py](file://libs/agno/agno/vectordb/mongodb/mongodb.py)
- [mongodb_for_agent.py](file://cookbook/db/mongo/mongodb_for_agent.py)
- [mongodb_for_team.py](file://cookbook/db/mongo/mongodb_for_team.py)

**本节来源**
- [mongodb.py](file://libs/agno/agno/vectordb/mongodb/mongodb.py)
- [mongodb_for_agent.py](file://cookbook/db/mongo/mongodb_for_agent.py)
- [mongodb_for_team.py](file://cookbook/db/mongo/mongodb_for_team.py)

## 性能考虑
### 索引创建最佳实践
为了优化查询性能，建议在MongoDB集合上创建适当的索引。`MongoDb`类自动创建向量搜索索引，但也可以手动创建其他索引。

```python
def _create_search_index(self, overwrite: bool = True) -> None:
    """Create or overwrite the Atlas Search index with proper error handling."""
    index_name = self.search_index_name or "vector_index_1"
    max_retries = 3
    retry_delay = 5

    if self.cosmos_compatibility:
        try:
            collection = self._get_collection()

            # Handle overwrite if requested
            if overwrite and index_name in collection.index_information():
                log_info(f"Dropping existing index '{index_name}'")
                collection.drop_index(index_name)

            embedding_dim = getattr(self.embedder, "dimensions", 1536)
            log_info(f"Creating vector search index '{index_name}'")

            # Create vector search index using Cosmos DB IVF format
            collection.create_index(
                [("embedding", "cosmosSearch")],
                name=index_name,
                cosmosSearchOptions={
                    "kind": "vector-ivf",
                    "numLists": 1,
                    "dimensions": embedding_dim,
                    "similarity": self._get_cosmos_similarity_metric(),
                },
            )

            log_info(f"Created vector search index '{index_name}' successfully")

        except Exception as e:
            logger.error(f"Error creating vector search index: {e}")
            raise
    else:
        for attempt in range(max_retries):
            try:
                if overwrite and self._search_index_exists():
                    log_info(f"Dropping existing search index '{index_name}'.")
                    try:
                        collection = self._get_collection()
                        collection.drop_search_index(index_name)
                        # Wait longer after index deletion
                        time.sleep(retry_delay * 2)
                    except errors.OperationFailure as e:
                        if "Index already requested to be deleted" in str(e):
                            log_info("Index is already being deleted, waiting...")
                            time.sleep(retry_delay * 2)  # Wait longer for deletion to complete
                        else:
                            raise

                # Verify index is gone before creating new one
                retries = 3
                while retries > 0 and self._search_index_exists():
                    log_info("Waiting for index deletion to complete...")
                    time.sleep(retry_delay)
                    retries -= 1

                log_info(f"Creating search index '{index_name}'.")

                # Get embedding dimension from embedder
                embedding_dim = getattr(self.embedder, "dimensions", 1536)

                search_index_model = SearchIndexModel(
                    definition={
                        "fields": [
                            {
                                "type": "vector",
                                "numDimensions": embedding_dim,
                                "path": "embedding",
                                "similarity": self.distance_metric,
                            },
                        ]
                    },
                    name=index_name,
                    type="vectorSearch",
                )

                collection = self._get_collection()
                collection.create_search_index(model=search_index_model)

                if self.wait_until_index_ready_in_seconds:
                    self._wait_for_index_ready()

                log_info(f"Search index '{index_name}' created successfully.")
                return

            except errors.OperationFailure as e:
                if "Duplicate Index" in str(e) and attempt < max_retries - 1:
                    logger.warning(f"Index already exists, retrying... (attempt {attempt + 1})")
                    time.sleep(retry_delay * (attempt + 1))
                    continue
                logger.error(f"Failed to create search index: {e}")
                raise
            except Exception as e:
                logger.error(f"Unexpected error creating search index: {e}")
                raise
```

**本节来源**
- [mongodb.py](file://libs/agno/agno/vectordb/mongodb/mongodb.py#L226-L319)

### 连接池配置
为了提高性能，建议配置适当的连接池大小。`MongoDb`类允许在初始化时设置最大连接池大小。

```python
def __init__(
    self,
    collection_name: str,
    db_url: Optional[str] = "mongodb://localhost:27017/",
    database: str = "agno",
    embedder: Optional[Embedder] = None,
    distance_metric: str = Distance.cosine,
    overwrite: bool = False,
    wait_until_index_ready_in_seconds: Optional[float] = 3,
    wait_after_insert_in_seconds: Optional[float] = 3,
    max_pool_size: int = 100,
    retry_writes: bool = True,
    client: Optional[MongoClient] = None,
    search_index_name: Optional[str] = "vector_index_1",
    cosmos_compatibility: Optional[bool] = False,
    search_type: SearchType = SearchType.vector,
    hybrid_vector_weight: float = 0.5,
    hybrid_keyword_weight: float = 0.5,
    hybrid_rank_constant: int = 60,
    **kwargs,
):
    # ...
    self.kwargs.update(
        {
            "maxPoolSize": max_pool_size,
            "retryWrites": retry_writes,
            "serverSelectionTimeoutMS": 5000,  # 5 second timeout
        }
    )
    # ...
```

**本节来源**
- [mongodb.py](file://libs/agno/agno/vectordb/mongodb/mongodb.py#L104-L107)

## 故障排除指南
### 错误处理
`MongoDb`类提供了详细的错误处理机制，包括连接失败、操作失败和意外错误的处理。

```python
def _get_client(self) -> MongoClient:
    """Create or retrieve the MongoDB client."""
    if self._client is None:
        if self.cosmos_compatibility:
            try:
                log_debug("Creating MongoDB Client for Azure Cosmos DB")
                # Cosmos DB specific settings
                cosmos_kwargs = {
                    "retryWrites": False,
                    "ssl": True,
                    "tlsAllowInvalidCertificates": True,
                    "maxPoolSize": 100,
                    "maxIdleTimeMS": 30000,
                }

                # Suppress UserWarning about CosmosDB
                import warnings

                with warnings.catch_warnings():
                    warnings.filterwarnings(
                        "ignore", category=UserWarning, message=".*connected to a CosmosDB cluster.*"
                    )
                    self._client = MongoClient(self.connection_string, **cosmos_kwargs)  # type: ignore

                    self._client.admin.command("ping")

                log_info("Connected to Azure Cosmos DB successfully.")
                self._db = self._client.get_database(self.database)  # type: ignore
                log_info(f"Using database: {self.database}")

            except errors.ConnectionFailure as e:
                raise ConnectionError(f"Failed to connect to Azure Cosmos DB: {e}")
            except Exception as e:
                logger.error(f"An error occurred while connecting to Azure Cosmos DB: {e}")
                raise
        else:
            try:
                log_debug("Creating MongoDB Client")
                self._client = MongoClient(self.connection_string, **self.kwargs)
                # Trigger a connection to verify the client
                self._client.admin.command("ping")
                log_info("Connected to MongoDB successfully.")
                self._db = self._client[self.database]  # type: ignore
            except errors.ConnectionFailure as e:
                logger.error(f"Failed to connect to MongoDB: {e}")
                raise ConnectionError(f"Failed to connect to MongoDB: {e}")
            except Exception as e:
                logger.error(f"An error occurred while connecting to MongoDB: {e}")
                raise
    return self._client
```

**本节来源**
- [mongodb.py](file://libs/agno/agno/vectordb/mongodb/mongodb.py#L126-L175)

### 数据一致性策略
为了确保数据一致性，`MongoDb`类提供了多种策略，包括插入、更新和删除操作的原子性保证。

```python
def insert(self, content_hash: str, documents: List[Document], filters: Optional[Dict[str, Any]] = None) -> None:
    """Insert documents into the MongoDB collection."""
    log_debug(f"Inserting {len(documents)} documents")
    collection = self._get_collection()

    prepared_docs = []
    for document in documents:
        try:
            document.embed(embedder=self.embedder)
            if document.embedding is None:
                raise ValueError(f"Failed to generate embedding for document: {document.id}")
            doc_data = self.prepare_doc(content_hash, document, filters)
            prepared_docs.append(doc_data)
        except ValueError as e:
            logger.error(f"Error preparing document '{document.name}': {e}")

    if prepared_docs:
        try:
            collection.insert_many(prepared_docs, ordered=False)
            log_info(f"Inserted {len(prepared_docs)} documents successfully.")
            if self.wait_after_insert_in_seconds and self.wait_after_insert_in_seconds > 0:
                time.sleep(self.wait_after_insert_in_seconds)
        except errors.BulkWriteError as e:
            logger.warning(f"Bulk write error while inserting documents: {e.details}")
        except Exception as e:
            logger.error(f"Error inserting documents: {e}")
```

**本节来源**
- [mongodb.py](file://libs/agno/agno/vectordb/mongodb/mongodb.py#L517-L545)

## 结论
本文档详细介绍了Agno系统中MongoDB集成的各个方面，包括配置、使用、实现细节、设计模式、性能优化和错误处理。通过`mongodb_for_agent.py`和`mongodb_for_team.py`文件，我们展示了如何将MongoDB用作智能体和团队的持久化存储。`MongoDb`类提供了完整的向量数据库功能，支持向量搜索、关键字搜索和混合搜索。通过适当的索引创建和连接池配置，可以显著提高查询性能。此外，详细的错误处理和数据一致性策略确保了系统的稳定性和可靠性。