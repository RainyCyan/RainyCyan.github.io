---
title: ANN 研究全景：论文、开源与工业系统的对照地图
date: 2026-06-02T20:00:00+08:00
tags: [向量检索, ANN, HNSW, DiskANN, IVF, ColBERT, CAGRA, 综述]
series: [向量检索]
featured: true
description: "把 ANN 这条赛道上的核心算法、动态/过滤/多向量/硬件加速等各支线、以及主流开源系统和工业落地放在同一张地图上对照。每个分支给出代表论文、对应的开源实现和工业系统,以及和本系列其他精读文章的链接。"
draft: false
ShowToc: true
TocOpen: true
---

向量检索这条赛道这几年的论文数量大到一个人很难全跟。光是 SIGMOD/VLDB/NeurIPS 这种主会,每年都会有十几篇 ANN 相关的系统/算法工作,加上 NVIDIA、Microsoft、Meta、Google、Pinecone、Milvus 这些工业系统不断刷新工程边界,很容易陷在某一个分支上看不到全图。

这篇文章不展开任何一篇论文的细节,只做一件事:**把这条赛道的主要分支、代表性论文、对应开源实现和工业系统拉到同一张图上**,作为后续精读文章的索引。系列里已经精读过的方向用 `精读:` 链到对应文章。

<!--more-->

## 1. 核心 ANN 算法

ANN 的底盘,这一层决定了上面所有支线的性能上限。三大族:图索引、聚类索引、量化。

### 图索引(Graph-based ANN)

- **HNSW** — *Efficient and Robust Approximate Nearest Neighbor Search Using Hierarchical Navigable Small World Graphs*, Malkov & Yashunin, TPAMI 2018。事实上的图索引标准。
  - 开源:[hnswlib](https://github.com/nmslib/hnswlib)
  - 工业:Milvus、Weaviate、Qdrant、OpenSearch kNN 几乎都默认用 HNSW
- **NSG** — *Fast Approximate Nearest Neighbor Search With Navigating Spreading-out Graphs*, VLDB 2019。MRNG 简化版,在某些数据集上比 HNSW 更省边。
- **DiskANN / Vamana** — *DiskANN: Fast Accurate Billion-point Nearest Neighbor Search on a Single Node*, NeurIPS 2019, Microsoft。开启了"单机十亿规模"这条线。
  - 开源:[microsoft/DiskANN](https://github.com/microsoft/DiskANN)
  - 工业:Azure AI Search、Bing 检索栈
  - 精读:[ANN On Disk 优化全景]({{< ref "/blog/vector-index-disk-ann" >}})
- **CAGRA** — NVIDIA, ICDE 2024。GPU 上的图索引,搜索阶段几乎打满 GPU 算力。
  - 开源:[rapidsai/cuvs](https://github.com/rapidsai/cuvs)
- **HNSW 变体** — 各种 Filtered HNSW(VLDB/SIGMOD 2022–2024),把 filter 集成到图导航里。

### 聚类索引(Cluster-based ANN)

- **IVF / IVF-PQ** — *Billion-scale Similarity Search with GPUs*, FAISS 论文, Meta AI, 2017。倒排 + 量化组合,十亿规模的基础方案。
  - 开源:[facebookresearch/faiss](https://github.com/facebookresearch/faiss)
- **ScaNN** — Google Research, ICML 2020。各向异性量化把 PQ 的精度推到当时的 SOTA。
  - 开源:[google-research/scann](https://github.com/google-research/scann)
- **SPANN / SPFresh** — Microsoft, NeurIPS 2021 / SOSP 2023。SSD 友好的聚类索引,SPFresh 加入了增量更新。

### 量化(Quantization)

- **PQ / OPQ** — *Product Quantization for Nearest Neighbor Search*, Jégou et al., TPAMI 2011。所有现代向量库的底层压缩基本都从这一脉演化出来。
- **IVF-PQ hybrids** — FAISS 里默认的十亿规模配置。

## 2. 动态 / 流式 ANN

传统 HNSW 这种"批量构建"的思路在生产里有硬伤——数据持续来、需要删除、需要 ACID。这条支线在过去三年是 SIGMOD/VLDB 的高频赛道。

- **FreshDiskANN** — Microsoft, SIGMOD/VLDB lineage。把 DiskANN 改造成支持持续插入/删除的版本。
- **LSM-based Vector Indexing** — 把 LSM 树的思路搬到向量索引上(SIGMOD/VLDB 2023–2025 多篇)。
- **Dynamic HNSW / 增量图更新** — 大量系统论文研究"插入和删除如何不破坏图质量"。

工业系统这一侧:

- **Pinecone** — *HNSW is not enough for production*,博客详细讲了纯 HNSW 在生产环境里的痛点。<https://www.pinecone.io/blog/hnsw-not-enough/>
- **Milvus** — [milvus-io/milvus](https://github.com/milvus-io/milvus),动态 HNSW + 磁盘索引
- **Weaviate** — [weaviate/weaviate](https://github.com/weaviate/weaviate),动态 HNSW + filter 集成
- **Qdrant** — [qdrant/qdrant](https://github.com/qdrant/qdrant),payload-aware HNSW 更新

## 3. 负载感知 ANN

这一支线相对偏冷,但在大规模在线服务里收益很高:同样的索引,在不同查询分布下用不同搜索策略,能省一大半算力。

- **Query Distribution-aware ANN** — VLDB / SIGMOD 工作负载感知索引论文。
- **Query-adaptive ANN Search** — 自适应 beam search / efSearch 在线调参。
- **Query Routing Systems** — Google / Microsoft 内部检索路由系统(SIGIR / WWW 系统 track)。
- **Embedding 语义缓存** — Redis 系向量缓存模式;Pinecone 的缓存层。
- **多租户 ANN** — VLDB / ICDE 多租户向量检索系统;Azure AI Search、Vespa、Pinecone。
- **Query Workload Modeling** — SIGMOD/VLDB 工作负载预测论文。

## 4. 过滤 / 混合 / 结构化 ANN

加约束的 ANN——只在满足某些元数据条件的向量里找最近邻。商品按价格区间、文档按时间窗口、按标签过滤等,都属于这一类。

### Filtered ANN(元数据过滤)

- Microsoft DiskANN 系列的 filtered search 论文(VLDB/SIGMOD 2022–2024)。
- 工业:Milvus、Weaviate、Qdrant、Elastic kNN 全部支持。

### 混合检索(Sparse + Dense)

- **BM25** — 经典 IR,稀疏检索基础。
- **SPLADE** — SIGIR 2021–2023。学得的稀疏表示。
  - 开源:[naver/splade](https://github.com/naver/splade)
- **ColBERT** — SIGIR 2020。Late interaction,token 级稀疏交互。
  - 开源:[stanford-futuredata/ColBERT](https://github.com/stanford-futuredata/ColBERT)
- 工业:Elastic、OpenAI RAG 栈、Azure AI Search 都做了 sparse+dense 融合。

### Range / Radius ANN

带范围属性过滤的近邻搜索这两年专门成为一个 SIGMOD/VLDB 的高频赛道。

- 精读:[范围过滤 ANN:从 IRangeGraph 到 WoW 的窗口图]({{< ref "/blog/vector-search-range-filter-wow" >}})

## 5. 多向量与 Late Interaction

把一段文本拆到 token 级别、每个 token 一个向量的检索范式。精度高,但存储和计算各放大 1–2 个量级。

- **ColBERT** — SIGIR 2020, Stanford。这条线的起点。
  - 开源:[stanford-futuredata/ColBERT](https://github.com/stanford-futuredata/ColBERT)
- **PLAID** — SIGIR 2022。把 ColBERT 的延迟从秒级压到几十毫秒。
- **WARP** — 2024, Stanford / ETH / Berkeley。继续往下压。
- **EMVB** — arXiv 2024。多向量压缩。
- 精读:[多向量检索:ByteFlow Chamber 在 GH200 上把端到端压到 1.8 ms]({{< ref "/blog/vector-search-multi-vector" >}})

工业系统:

- Google DeepMind 内部检索系统
- OpenAI 多阶段 / token 级检索管线
- Perplexity AI 检索栈

## 6. 硬件加速 ANN

ANN 这条线对硬件特别敏感——SIMD、GPU tensor core、SSD 顺序读、NVLink 这些硬件特性都直接决定算法选型。

### GPU ANN

- **CAGRA** — NVIDIA, ICDE 2024。GPU 图索引。
- **FAISS GPU** — Meta。聚类 + 量化的 GPU 实现。
  - 开源:[facebookresearch/faiss](https://github.com/facebookresearch/faiss)

### CPU SIMD ANN

- FAISS 的 AVX2 / AVX512 优化。距离计算这一步对 SIMD 收益巨大。

### 分布式 ANN

- **Vespa** — Yahoo。[vespa-engine/vespa](https://github.com/vespa-engine/vespa)
- **Milvus** — Zilliz。
- **OpenSearch kNN plugin** — Elastic 系。

### Disk / SSD ANN

- **DiskANN** — Microsoft。
- **SPANN** — Microsoft。
- 精读:[ANN On Disk 优化全景]({{< ref "/blog/vector-index-disk-ann" >}})

## 7. 学习式 / 自动调优 ANN

把"学到的模型"放进 ANN 链路里——可以是 learned index,也可以是参数自动调优。

- **Learned Index 起源** — Kraska et al., SIGMOD 2018。
- **检索-嵌入联合学习** — SIGIR / NeurIPS 系列。
- **Auto-tuning ANN** — SIGMOD / VLDB ANN 参数调优系统论文。
- **基于成本的 ANN 优化** — DB 研究方向,目前还没有规范化的系统。

## 8. 多模态 / OOD / AI 时代检索

ANN 的下游应用层。embedding 模型本身从单模态走向多模态,且 agent 这种新型应用对 memory 检索提出了新要求。

### 多模态嵌入模型

- **CLIP** — OpenAI, 2021。
- **ALIGN** — Google, 2021。
- **OpenCLIP** — LAION OSS 实现。

### OOD / 分布漂移检索

- NeurIPS / ICLR embedding 鲁棒性文献。

### Agent Memory 检索

- **MemGPT** — arXiv 2023。
- **LangGraph / LangChain memory 系统** — 工业 OSS。

### Retrieval Foundation Models(萌芽期)

- DeepMind / Google 的研究方向,还没有规范化的代表论文。

### Reranker(端侧推理优化)

- 精读:[端侧 Reranker 优化:PRISM 怎么把 Cross-Encoder 跑进笔记本]({{< ref "/blog/vector-search-rerank-prism" >}})

## 9. 开源生态对照表

| 系统 | 主体 | 定位 | 链接 |
|------|------|------|------|
| FAISS | Meta | 算法库,IVF/PQ/HNSW 全家桶 | [github](https://github.com/facebookresearch/faiss) |
| DiskANN | Microsoft | 单机十亿规模,SSD-friendly | [github](https://github.com/microsoft/DiskANN) |
| Milvus | Zilliz | 分布式向量数据库 | [github](https://github.com/milvus-io/milvus) |
| Weaviate | Weaviate | 向量数据库 + filter | [github](https://github.com/weaviate/weaviate) |
| Qdrant | Qdrant | 向量数据库,payload-aware | [github](https://github.com/qdrant/qdrant) |
| Vespa | Yahoo | 分布式检索引擎 | [github](https://github.com/vespa-engine/vespa) |
| ScaNN | Google | 各向异性量化 | [github](https://github.com/google-research/scann) |
| SPLADE | Naver | 学得的稀疏表示 | [github](https://github.com/naver/splade) |
| ColBERT | Stanford | Late interaction | [github](https://github.com/stanford-futuredata/ColBERT) |
| cuVS / CAGRA | NVIDIA | GPU 向量检索栈 | [github](https://github.com/rapidsai/cuvs) |

## 几个判断

读完这张全景图之后的几个观察:

**1. 主线(图索引 + 聚类)在算法上几乎收敛了。** HNSW、DiskANN、IVF-PQ 这三套基础结构已经成为不变量,后续工作要么是工程优化(更好的存储布局、更好的硬件适配),要么是把它们适配到新场景(过滤、多向量、增量)。这一层很难再做颠覆性的新算法。

**2. 真正活跃的支线在"附加约束"和"硬件特化"。** 范围过滤、多向量、端侧推理、GPU 加速,这些方向的论文密度远超主线。原因很简单——主线已经稳定,但生产场景的复杂度持续上升(agent memory、多模态、超大规模 reranker),每一个新场景都需要把基础算法重新适配一遍。

**3. 学术和工业的脱节越来越大。** SIGMOD/VLDB 上的论文大多在追求"算法精度/延迟的 Pareto 前沿",但工业系统(Pinecone、Milvus)更关注的是"插入稳定性、删除一致性、多租户隔离"这种没有论文价值但极度重要的工程问题。读论文的时候要时刻警惕这种 gap。

**4. 真正的"统一索引"还没有出现。** 同时支持任意过滤(range + label + 正则)+ 多向量 + 高吞吐插入 + 分布式 + 多模态的索引目前还不存在。HSIG/UNIFY 这种尝试做"统一前/后/混合过滤"的论文是这个方向的早期探索,但离生产可用还很远。这是接下来几年最值得期待的方向。

## 参考资源

每个分支的代表论文和开源实现已在正文给出。本系列其他精读文章:

- [ANN On Disk 优化全景]({{< ref "/blog/vector-index-disk-ann" >}})
- [范围过滤 ANN:从 IRangeGraph 到 WoW 的窗口图]({{< ref "/blog/vector-search-range-filter-wow" >}})
- [多向量检索:ByteFlow Chamber 在 GH200 上把端到端压到 1.8 ms]({{< ref "/blog/vector-search-multi-vector" >}})
- [端侧 Reranker 优化:PRISM 怎么把 Cross-Encoder 跑进笔记本]({{< ref "/blog/vector-search-rerank-prism" >}})
