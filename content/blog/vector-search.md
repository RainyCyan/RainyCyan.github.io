---
title: 当我们谈到向量检索时我们到底在谈些什么
date: 2026-05-15T00:00:00Z
tags: [向量检索]
series: [向量检索]
featured: true
description: "世界上并不存在十全十美的东西，一切都是trade-off"
---

# 向量检索
embedding model 将非结构化数据（文本、语音、图像等）降维映射到语义空间的向量，保留了语义信息。

原始的信息检索可以变成向量空间的检索，也就是向量检索

向量化技术


向量检索的核心技术是近似最近邻ANN，因为执行精确的kNN的代价太大（由于维度和数据量的关系）。

ANN的三个技术流派，确切的说只有两种：倒排，图和量化，量化实际上是通过降低数据精度把计算的复杂度再次降低。【这里忽略了LSH和树形分区如kdtree,因为在实际工程中这些方法都存在重大缺陷无法使用】

倒排

图

nsw

hnsw

nsg

...... to be continued ......
<!-- Graph

Disk/SSD

GPU for ANN

Filtering ANN



ANN的发展方向

RAG

搜广推召回
检索库
faiss

Agent Memory

VectorDB
Milvus



 -->