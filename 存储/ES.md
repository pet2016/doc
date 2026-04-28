#### 什么是ElasticSearch

Elasticsearch 是一个基于 Apache Lucene 构建的开源、分布式、RESTful 搜索引擎。

- 核心引擎： 不同于传统数据库的 B+ Tree 索引，ES 使用倒排索引 (Inverted Index)，这使其在处理海量文本检索时具有数量级的性能优势。

- 数据模型： 以 JSON 文档为基本单位。虽然它是 Schema-less 的，但架构师通常会通过显式 Mapping 来优化性能。

- 分布式特质： 天生支持分片（Sharding）与副本（Replication），能够在廉价的硬件集群上实现水平扩展。


#### ElasticSearch的应用场景有哪些

在系统架构设计中，ES 通常扮演以下角色：

- 全文本检索： 电商商品搜索、APP 内全局搜索、多条件组合过滤及打分排序。

- 日志与指标分析 (ELK Stack)： 系统日志、业务埋点、网络流量的实时采集、存储与可视化。

- 向量检索 (Vector Search)： 在 AI 时代，ES 支持 HNSW 算法，用于存储 Embedding 并支撑 RAG（检索增强生成）架构。

- 时序数据分析： 监控系统运行指标（Metrics），利用其强大的聚合（Aggregation）功能生成实时报表。

- 安全分析 (SIEM)： 实时侦测异常登录、DDoS 攻击行为分析。


#### ElasticSearch 的性能如何
ES 的性能设计遵循“空间换时间”与“近实时（NRT）”原则：

- 读取性能（极快）： 借助操作系统文件缓存（OS Cache）以及倒排索引，复杂的关键词匹配通常能在毫秒级完成。

- 写入性能（准实时）： 数据写入后需经过 refresh 操作（默认 1 秒）生成新的 Segment 方可被搜索，因此不适合作为强实时一致性的事务数据库。

- 聚合性能（强大）： 使用 Doc Values（列式存储）在内存中进行统计运算，适合大规模数据分析。

- 瓶颈点： 深度分页（Deep Paging）和高基数（High Cardinality）字段的频繁聚合会显著消耗 CPU 和内存。

#### ElasticSearch 是如何保证高可用

ES 通过分布式的集群管理机制确保系统稳定性：

- 分片副本 (Replication)： 每个主分片（Primary）可拥有多个副本（Replica）。当节点故障时，集群自动进行 Master 选举并将副本提升为主分片。

- 节点角色分离：

    - Master-eligible Nodes： 负责集群元数据管理。

    - Data Nodes： 负责数据读写与运算。

    - Coordinating Nodes： 负责请求分发与结果汇聚，降低数据节点压力。

- 分片分配感知 (Shard Allocation Awareness)： 可配置分片在不同机架或可用区（AZ）的分布，防止单点物理故障导致全量数据丢失。

#### ElasticSearch的数据备份与恢复

ES 提供了成熟的快照机制，而非传统的物理文件拷贝：

- 快照 API (Snapshot)： 支持将索引数据增量备份到外部存储（如 S3、HDFS、Azure Blob 或 NAS）。

- 跨集群复制 (CCR)： 对于 RPO/RTO 要求极高的场景，可实施主从集群同步，实现异地多活或灾备。

- 索引恢复 (Restore)： 支持对单个索引或整个集群进行快照回滚，恢复过程不影响集群其他索引的正常运行。

#### 如何规划ElasticSearch的资源

资源规划（Capacity Planning）直接决定了集群的生产稳定性：

##### 内存规划 (The 50% Rule)
- JVM Heap： 建议设置为物理内存的 50%，但上限不得超过 32GB（为了利用压缩对象指针 Compressed OOPs 以节省内存）。
- OS Cache： 剩余的 50% 留给操作系统，用于缓存 Lucene 段文件，这是保证搜索飞快的关键。

###### 磁盘规划

- 磁盘类型： 强烈建议使用 NVMe SSD。磁盘 I/O 通常是 ES 的第一性能瓶颈。
- 存储容量： 总空间 = 源数据 * (1 + 副本数) * 1.1 (索引膨胀) * 1.1 (安全阈值)。

##### 分片规划

- 单分片大小： 日志类建议 30GB-50GB；搜索类建议 10GB-30GB。
- 分片总量： 确保单个节点上的分片数量与内存比例适中（建议每 GB 堆内存对应的分片数不超过 20 个）。


在引入 ES 时，应优先评估业务是“写多读少（日志型）”还是“读多写少（搜索型）”。对于海量数据，建议采用 Hot-Warm-Cold（冷热分层）架构，将热数据放在高性能 SSD 节点，老数据迁移至低成本 HDD 节点，以平衡成本与性能。


