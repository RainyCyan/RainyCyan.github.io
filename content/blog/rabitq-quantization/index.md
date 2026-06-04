---
title: RaBitQ：具有理论误差界的向量量化
date: 2026-06-04T10:00:00+08:00
tags: [向量搜索, 量化, 近似最近邻, RaBitQ]
series: [向量搜索]
featured: false
description: "理解 RaBitQ 如何通过无偏距离估计和理论误差界实现高效的向量量化"
draft: false
ShowToc: true
TocOpen: true
---

## 概述

**RaBitQ** 是 2024 年发表的向量量化方法，全名《RaBitQ: Quantizing High-Dimensional Vectors with a Theoretical Error Bound for Approximate Nearest Neighbor Search》。与 PQ 等经典量化方法不同，RaBitQ 的核心创新是提供了**理论误差界保证**，这对于需要可预测性能的生产系统至关重要。

- **论文链接**：https://dl.acm.org/doi/10.1145/3654970
- **官方代码**：https://github.com/gaoj0017/RaBitQ
- **文档与推导**：https://github.com/VectorDB-NTU/RaBitQ-Library/tree/main/docs

## 问题背景

在高维向量的近似最近邻（ANN）搜索中，距离计算是性能瓶颈。距离计算和比较往往占据整个搜索时间的 80%-90%。

虽然 PQ 及其变体已经提供了有效的压缩和加速方案，但它们存在两个根本性的问题：

1. **缺乏理论误差界**：无法预测量化引入的误差上限
2. **鲁棒性不足**：在某些真实数据集上会出现灾难性的性能下降

RaBitQ 针对这些问题进行了改进，通过数学上的无偏估计和高维向量的维度集中特性，建立了从量化距离到原始距离的理论保证。

## 核心设计思想

### 高维向量的维度集中现象

对于**归一化到单位球面上的向量**，随着维度 D 增加，向量的每个分量 $x_i$ 的分布呈现明显的集中特性——大部分分量聚集在 0 附近。

**直观解释**：在高维空间中，如果某一个分量取值较大（偏离平均值），为了保持单位长度，其他分量必须被急剧压缩。因此，"反常的"大分量很难出现。

Johnson-Lindenstrauss 引理从理论上证明了：高维单位向量中每个分量不太可能偏离 0 超过 $\frac{2}{\sqrt{D}}$。

这个性质是 RaBitQ 无偏估计的数学基础。

## 索引构建阶段

### 1. 数据向量归一化

将原始向量 $o_r$ 相对于一个全局中心点 $c$ 进行中心化和归一化：

$$o = \frac{o_r - c}{\|o_r - c\|}$$

其中 $c$ 通常是数据集的 centroid 或某个聚类的中心。

**为什么要中心化再归一化**：
- **消除全局偏移**：使数据围绕原点分布，减少数据集的偏斜（skewness）
- **统一尺度**：所有向量落在单位超球面上，后续的误差分析和距离计算有统一的参考
- **减少噪声**：消除了可能被所有向量共享的无用信息

### 2. 码本构造

**原始码本**：构造由 D 维二值向量组成的超立方体顶点集合：

$$C = \left\{ x \in \{-\frac{1}{\sqrt{D}}, +\frac{1}{\sqrt{D}}\}^D \right\}$$

所有码字都是单位向量，理论上均匀分布在球面上。

**问题**：固定的码本对某些方向有"偏好"。例如，向量 $(1, 0, \ldots, 0)$ 与码本中任何码字都不匹配，量化误差很大；而向量 $(\frac{1}{\sqrt{D}}, \ldots, \frac{1}{\sqrt{D}})$ 与某个码字完美匹配，误差为 0。这种"不公平"会导致某些数据的量化质量很差。

**解决方案：随机旋转**：

$$C_{\text{rand}} = \{P x \mid x \in C\}$$

其中 $P$ 是一个随机正交矩阵。旋转后的码本在统计意义上对所有方向公平对待，避免了系统性的偏向。虽然某一次旋转下仍可能存在偏差，但多次使用不同的 $P$ 会使这种偏差"平均掉"。

### 3. 数据向量量化

对于归一化后的向量 $o$，目标是找到码本中与之最近的码（最大化内积）。

由于 $\|o - Px\|^2 = 2 - 2\langle o, Px \rangle$，寻找距离最近的码等价于最大化内积：

$$\arg\max_x \langle o, P x \rangle = \arg\max_x \langle P^{-1}o, x \rangle$$

不直接旋转码本，而是先对 $o$ 进行逆旋转得到 $o' = P^{-1}o$，然后对 $o'$ 的每个分量取符号（正为 1，负为 0），得到 D 位的二进制码 $\bar{x}_b$。

最终存储的量化向量为：

$$\bar{o} := P\bar{x}, \quad \text{其中} \quad \bar{x}[i] = \frac{2\bar{x}_b[i] - 1}{\sqrt{D}}$$

## 查询阶段：无偏距离估计

### 1. 距离公式的推导

原始向量间的欧氏距离可以通过余弦定理表示为：

$$\|o_r - q_r\|^2 = \|o_r - c\|^2 + \|q_r - c\|^2 - 2 \cdot \|o_r - c\| \cdot \|q_r - c\| \cdot \langle q, o \rangle$$

其中 $q = \frac{q_r - c}{\|q_r - c\|}$ 是归一化的查询向量。

注意到 $\|o_r - c\|$ 和 $\|q_r - c\|$ 分别可以在索引和查询时预计算，问题的关键归结为估计 $\langle q, o \rangle$。

### 2. 无偏估计器

当 $\bar{o}$ 是 $o$ 的量化近似时，直接用 $\langle \bar{o}, q \rangle$ 作为 $\langle o, q \rangle$ 的近似会有偏差。RaBitQ 构造了一个无偏估计器：

$$\langle q, o \rangle \approx \frac{\langle \bar{o}, q \rangle}{\langle \bar{o}, o \rangle}$$

**几何直觉**：在 $o$ 和 $q$ 张成的二维平面上建立坐标系。由于 $\bar{o}$ 是随机旋转后的量化向量，它在垂直于 $o$ 方向的投影期望为 0（高维随机向量几乎必然正交于任何固定的低维子空间）。因此分母中的交叉项可以被忽略。

**误差界**（理论保证）：

$$\left|\frac{\langle \bar{o}, q \rangle}{\langle \bar{o}, o \rangle} - \langle o, q \rangle\right| \leq O\left(\frac{1}{\sqrt{D}}\right)$$

这是 RaBitQ 相比 PQ 最根本的进步——量化误差有明确的数学上界。

### 3. 查询向量量化

为了进一步加速，对查询向量的旋转后结果 $q' = P^{-1}q$ 进行标量量化（scalar quantization），量化到 $B_q$ 比特的整数。

具体地，将 $q'$ 的值域 $[v_l, v_r]$ 均匀划分成 $2^{B_q} - 1$ 个区间，然后用随机量化避免偏斜：

$$\bar{q}_u[i] = \frac{q'[i] - v_l}{\Delta} + u_i$$

其中 $\Delta = \frac{v_r - v_l}{2^{B_q} - 1}$，$u_i$ 是 [0, 1) 上的随机数。

### 4. 高效计算

量化后的内积计算可以分解为几个可预计算的项。对于单位码本（1 比特编码），计算变成简单的位操作：

$$\langle \bar{x}, \bar{q} \rangle = \frac{1}{D}\left(4 \langle \bar{x}_b, \bar{q}_u \rangle - 2 \sum \bar{x}_b - 2 \sum \bar{q}_u + D\right)$$

其中 $\bar{x}_b$ 是二进制码，$\bar{q}_u$ 是量化后的查询向量。这可以用 SIMD 指令或 popcount 等底层操作高效实现。

## 重排与候选剪枝

RaBitQ 的误差界保证使得可以进行**有保证的重排剪枝**。

维护一个大小为 K 的最大堆，对于候选向量 A：
1. 如果 A 的下界 > 堆内向量的上界最大值 → 直接丢弃（确定不在 KNN 中）
2. 如果 A 的下界 ≤ 堆顶上界，且堆顶已重排 → 重排 A
3. 如果 A 的下界 ≤ 堆顶上界，但堆顶未重排 → 先重排堆顶，再判断

## 工业应用

RaBitQ 已被多个向量数据库和搜索系统采用：

- **Milvus**（C++）：IVF + RaBitQ
- **Faiss**（C++）：IVF + RaBitQ
- **VSAG**（C++）：HGraph + RaBitQ
- **VectorChord**（Rust）：IVF + RaBitQ
- **Volcengine OpenSearch**：DiskANN + RaBitQ
- **CockroachDB**（Golang）：CSPANN + RaBitQ
- **Elasticsearch**（Java）：HNSW + RaBitQ（改进版本称为 BBQ）
- **Lucene**（Java）：HNSW + RaBitQ
- **OceanBase**：集成支持

## 优缺点

### 优势

1. **理论误差界**：提供了 $O(1/\sqrt{D})$ 的无偏估计误差上界，这在生产系统中极其重要
2. **鲁棒性**：归一化和随机旋转使得量化对数据分布的依赖性更小，避免灾难性故障
3. **计算高效**：二值编码 + SIMD 优化可以实现非常快的内积计算
4. **有保证的重排**：误差界使得可以安全地剪枝候选集，进一步加速

### 局限

1. **仅支持特定距离度量**：目前主要支持欧氏距离和余弦距离
2. **索引大小**：存储量化码需要 D 比特（或更多），对于超高维向量可能仍需压缩
3. **复杂性**：相比 PQ 的实现，RaBitQ 涉及更多的数学和工程细节

## 参考资源

- [RaBitQ 论文](https://dl.acm.org/doi/10.1145/3654970)
- [RaBitQ 官方库](https://github.com/gaoj0017/RaBitQ)
- [RaBitQ 库文档和推导](https://github.com/VectorDB-NTU/RaBitQ-Library/tree/main/docs)
- [作者技术博客](https://dev.to/gaoj0017/quantization-in-the-counterintuitive-high-dimensional-space-4feg)
- [Elastic RaBitQ 解读](https://www.elastic.co/search-labs/blog/rabitq-explainer-101)
- [Johnson-Lindenstrauss 引理](https://www.spaces.ac.cn/archives/8679/)
