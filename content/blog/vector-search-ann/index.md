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

- 百度Puck/Tinker

### 量化(Quantization)

- **PQ / OPQ** — *Product Quantization for Nearest Neighbor Search*, Jégou et al., TPAMI 2011。所有现代向量库的底层压缩基本都从这一脉演化出来。
  - 开源:[facebookresearch/faiss](https://github.com/facebookresearch/faiss)
- **Residual Quantization / Additive Quantization** — *Additive Quantization for Extreme Low Bit Rates*, Babenko & Lempitsky, CVPR 2014。RQ 的递进改进。
- **IVF-PQ hybrids** — FAISS 里默认的十亿规模配置。
- **IRVQ** — *Improved Residual Vector Quantization*, arXiv 2015。在 RQ 基础上的分析与优化。
- **RaBitQ** — *Re-thinking the Value of Labels for Improving Class-Imbalanced Learning*, Gao & Long, SIGMOD 2024。D-bit 编码的无偏距离估计器,具有 O(1/√D) 阶的理论误差界。SIMD/bitwise 加速,PQ 族的理论型替代方案。
- **Practical & Asymptotically Optimal Quantization** — SIGMOD 2025。扩展 RaBitQ 至灵活的每维 bit 数,保留理论保证。
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

### 查询驱动的索引优化

- **Query Distribution-aware ANN** — VLDB / SIGMOD 工作负载感知索引论文。利用倾斜的查询分布加速搜索。
- **Query-adaptive ANN Search** — 自适应参数调优。根据查询特征在线调整 efSearch(HNSW)、beam width 等搜索参数。
- **Auto-tuning 系统** — DiskANN 系统实现中内置的参数自适应机制。

### 系统层面的负载感知

- **Query Routing / Request Dispatching** — Google / Microsoft 内部大规模检索路由系统(SIGIR / WWW 系统 track)。不同查询路由到优化过的索引副本。
- **Embedding 语义缓存** — Redis 系向量缓存模式(热点向量);Pinecone 的缓存层。基于语义相似度的缓存一致性。
- **多租户隔离与资源调度** — VLDB / ICDE 多租户向量检索系统;Azure AI Search、Vespa、Pinecone 等产品的隔离方案。每个租户的数据/参数独立优化。
- **Workload Drift Detection** — embedding 分布漂移监控(系统文献)。检测数据/查询分布变化,触发索引重建。
- **Query Workload Modeling** — SIGMOD/VLDB 工作负载预测论文。基于历史查询预测未来模式。

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

### ANN 扫描与内核加速

距离计算与暴力扫描这一层对硬件的利用率直接决定整体吞吐。

- **ADC (Asymmetric Distance Computation)** — PQ/量化系列的基础概念,在编码空间与原始向量间的非对称距离计算,是现代 ANN 系统的标准优化。
- **FAISS SIMD 加速** — Meta。AVX2/AVX512 在距离计算上的向量化优化。FAISS 论文: *Billion-scale Similarity Search with GPUs*, TPAMI 2017。
  - 开源:[facebookresearch/faiss](https://github.com/facebookresearch/faiss)
- **ScaNN 量化加速** — Google Research, ICML 2020。各向异性量化配合快速扫描内核。
  - 开源:[google-research/scann](https://github.com/google-research/scann)
- **GPU 距离内核** — NVIDIA CAGRA、cuVS 中的 SIMD-on-GPU 优化。
  - 开源:[rapidsai/cuvs](https://github.com/rapidsai/cuvs)

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

### Agent Memory 与检索增强系统

Agent 对 memory 检索的需求与传统 RAG 不同:需要细粒度的语义分层、高频更新、跨对话持久化。这条线已经成为 LLM 应用的标配。

- **MemGPT** — *MemGPT: Towards LLMs as Operating Systems*, arXiv 2023, UC Berkeley。操作系统级别的多层 memory 架构(context window / external storage / 分级 retrieval)。
- **LangChain Memory 系统** — 工业 OSS,提供 buffer/vector/summary 多种 memory 模块。
  - 开源:[langchain-ai/langchain](https://github.com/langchain-ai/langchain)
- **LangGraph** — LangChain 的图计算框架,支持 memory 持久化与状态管理。
  - 开源:[langchain-ai/langgraph](https://github.com/langchain-ai/langgraph)
- **Vector DB as Memory Backend** — 向量数据库(Pinecone、Weaviate、Qdrant)作为 LLM 应用的 memory 层。具有语义检索、ACID、增量更新的特性。
- **Mem0** — 工业化的 memory 层抽象,在向量 DB + LLM 之上构建。
  - 开源:[mem0ai/mem0](https://github.com/mem0ai/mem0)

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
