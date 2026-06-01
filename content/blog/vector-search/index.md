---
title: 当我们谈到向量检索时我们到底在谈些什么
date: 2026-05-15T00:00:00Z
tags: [向量检索]
series: [向量检索]
featured: true
description: "世界上并不存在十全十美的东西，一切都是trade-off"
draft: false
---
<!-- ## 向量检索系统架构
一条典型的向量检索链路：
```
Query -> Embedding -> ANN Retrival -> Re-ranking -> ... -> Result
``` -->

## 向量化Embedding

<!-- ### 数据源
不同于传统的结构化数据，非结构化数据 -->
### 向量化模型Embedding Model
embedding model 将非结构化数据（文本、语音、图像等）降维映射到语义空间的向量，保留了语义信息。

原始的信息检索可以变成向量空间的检索，也就是向量检索

<!-- ### 向量表示问题与挑战 -->

## 近似最近邻检索ANNS
### 向量度量
向量距离度量一般有欧式距离、余弦距离、内积距离和海明距离四种

#### 欧式距离 L2 distance
L2距离指的是在欧式空间中两个点之间的直线距离，即
$$
d(p,q) = \sqrt{\sum_{i=1}^{d}(p_i-q_i)^{2}}
$$

#### 余弦距离 Cosine distance

用于衡量向量方向的差异，忽略长度：

$$
\text{cosine}(p,q) = \frac{p \cdot q}{\|p\|\|q\|}
$$

余弦距离通常定义为：

$$
d(p,q) = 1 - \text{cosine}(p,q)
$$


#### 内积 Inner Product distance
衡量两个向量的相似程度：

$$
p \cdot q = \sum_{i=1}^{d} p_i q_i
$$

在检索中通常直接使用最大内积（MIPS）作为排序依据：

$$
\text{score}(p,q) = p \cdot q
$$

---
`余弦和内积的相互转换：Cosine similarity
≈ Inner Product (向量归一化后)`
#### 海明距离Hamming distance
用于二值向量（binary vector），表示对应位不同的个数：

$$
d(p,q) = \sum_{i=1}^{d} \mathbb{1}(p_i \neq q_i)
$$
---

在一些特殊场景下，可能还会引入其他度量方式：比如在稀疏特征使用L1距离，KL散度用于“概率向量”。
### 评价指标

#### 召回率 Recall@K
Recall@K 用于衡量 ANN 检索结果与真实 Top-K 近邻的一致程度，其定义为：

$$
Recall@K = \frac{|Result_K \cap GT_K|}{K}
$$

其中 $Result_K$ 为索引返回的 Top-K 结果，$GT_K$ 为暴力检索得到的真实 Top-K 近邻(ground truth)。Recall@K 越接近 1，说明索引的检索精度越高。

#### 平均延迟 Latency

延迟指标需要明确统计范围，是否包含召回（Recall）、粗排（Ranking）以及重排（Re-ranking）等不同阶段。

对于十亿级向量检索系统，Top-K=10 查询通常要求在保证较高 Recall 的前提下，将召回阶段延迟控制在毫秒级以内，高性能系统往往能够达到亚毫秒级水平。

如果要分析查询分布，可以打印P90、P95、P99进一步分析。

#### Recall-Latency曲线
实际评估 ANN 索引时，很多团队最关注的不是单个数字，而是recall-latency的对应趋势，因此论文或者benchmark常常会绘制Recall-Latency曲线来比较不同索引的综合性能。

#### QPS
qps(query-per-second)衡量系统每秒可以处理多少查询。

$$QPS = \frac {Query\ \ Count}{Time}$$
#### Build Time/Index Size
在索引构建阶段(离线侧)，需要关注资源以及构建时间。

#### 距离计算次数 Compute_Count
在向量检索中，查询延迟的主要开销来自距离计算，因此通常会引入计算成本类指标比如Compute_Count

### ANN索引
通过向量化技术我们得到了embedding,之后我们需要对embedding进行检索。向量检索的核心技术是近似最近邻ANN，因为执行精确的kNN的代价太大（由于维度和数据量的关系）。ANN索引的内容非常多，仅仅陈述主流的方法都值得再开一篇文章，这里只描述最最简单的介绍。


很多ANN综述都会把ANN分成
- 树
- Hash
- 量化
- 图
这四类，这里我将描述实践中广泛使用的倒排、量化和图三类的典型方法。【这里忽略了LSH和树形分区如kdtree,因为在实际工程中这两类方法都存在重大缺陷无法使用】

### 倒排索引IVF

### 量化技术

### 图

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
## 混合检索/多路召回

## 重排

## 工程扩展

## 参考资源
- [1] https://github.com/datawhalechina/what-is-vs/

- [2]