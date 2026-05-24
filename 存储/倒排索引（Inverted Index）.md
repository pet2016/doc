
简单来说，传统数据库索引（正向索引）是“从文档查找单词”，而倒排索引是“从单词查找文档”。

#### 核心概念

为了直观理解，我们对比两种索引模式：
##### 正向索引 (Forward Index)
类似于书的目录。
- 逻辑： 文档 ID -> 包含的内容（单词）。
- 痛点： 如果你想找包含“架构”这个词的所有文档，数据库必须扫描每一行（全表扫描），效率极低。

##### 倒排索引 (Inverted Index)
类似于书末尾的索引页。
- 逻辑： 单词 -> 出现该单词的文档 ID 列表。
- 优势： 无论数据量多大，通过单词可以直接定位到所有相关文档。


#### 倒排索引的内部结构

一个完整的倒排索引主要由两个核心部分组成：

##### A. 词典 (Term Dictionary)
记录所有文档中出现过的唯一单词（Term）。
- 为了提高查询效率，词典通常采用 B-Tree 或 FST (Finite State Transducer) 结构存储在内存中，以便快速定位单词。

##### B. 倒排列表 (Posting List)
对应每一个单词，记录了所有包含该词的文档信息，通常包括：
- 文档 ID (DocID)： 用于获取原始文档。
- 词频 (TF)： 该词在文档中出现的次数（用于相关性打分）。
- 位置 (Position)： 单词在文档中的偏移量（用于短语搜索，如搜索 "Java 架构" 时要求两个词相邻）。

#### 3. 倒排索引的构建过程（示例）

假设我们有两个文档：
- Doc 1: "Elasticsearch is fast"
- Doc 2: "Elasticsearch is distributed"

第一步：分词 (Tokenization)将文本拆分为独立的单词，并统一转为小写（标准化）：
- Doc 1: [elasticsearch, is, fast]
- Doc 2: [elasticsearch, is, distributed]

第二步：构建映射
记录每个单词出现在哪个文档中：

| 单词 (Term)  | 文档 ID (Posting List)  |
| ------- | ------- |
| distributed | [2]  | 


| elasticsearch |  [1, 2]    |
| fast | [1]   |
| is | [1, 2]   |

第三步：查询
当你搜索 "fast" 时，ES 无需查看 Doc 2，直接命中词典中的 "fast"，并返回 Posting List 中的 [1]。

#### 4. 为什么倒排索引性能高？

- FST 压缩： 词典（Term Dictionary）非常大，ES 使用 FST 压缩技术将其驻留在内存中，极大减少了磁盘 I/O。

- 跳表 (Skip List)： 在进行多词交集查询（如搜索“高性能 AND 架构”）时，利用跳表结构可以快速合并两个长 Posting List。

- Frame of Reference (FOR) 压缩： Posting List 中的 DocID 是增量编码的（只存差值），并通过位压缩进一步减少存储空间。

- 不可变性 (Immutability)： 倒排索引一旦写成段（Segment）就不再更改。这使得索引可以被缓存，且避免了复杂的并发锁问题。

#### 总结
倒排索引是搜索技术的“基石”。它通过预先分词和建立单词到文档的映射，将复杂的文本检索问题转化为了极快的内存查找和位运算问题。