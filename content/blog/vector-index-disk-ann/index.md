---
title: 向量检索图索引：ANN On Disk 的几条优化主线
date: 2026-06-02T10:00:00+08:00
tags: [向量检索, DiskANN, Starling, SPANN, ANN, 图索引, 向量数据库, 磁盘索引]
series: [向量检索]
featured: true
description: "围绕 DiskANN 之后的几篇代表性工作（Starling、加权重排、PipeANN、BAM、SAQ 等），梳理 ANN On Disk 在 I/O 布局、缓存、计算瓶颈上的演进路线。"
draft: false
ShowToc: true
TocOpen: true
---

---
todo: 这里增加对于DiskANN系列文章包括DiskANN/FreshDiskANN/FilterDiskANN的描述

大规模向量(billion+)，HNSW/NSG这种最初设计全放到内存的索引单机无法处理（因为billion级别向量普通单机RAM甚至连图结构都放不下了）。

diskann把图结构和原始向量都放到SSD上了,vamana图放宽了nsg裁边的限制（有什么好处）；

图结构直接放到SSD上面有个问题是，因为每次IO都是随机访问，无法去做有效的prefetch，关键问题就是能不能最小化平均路径长度（但其实这也是内存型图ANN在做的事），或者工程上能不能用并行加速检索掩盖损失也行。核心目标就是减少IO次数

beam search时并行扩展：如何处理竞争的问题？
批量IO

1. Vamana graph设计目标：1.最小化图直径，最小化IO跳数
    Vamana支持子图合并，可以分布式构建

2. beam search：把多跳邻居都拿到RAM
3. PQ+: 减小在过滤候选过程的计算量，同时PQ后的vector可以放到RAM中（这个实际上依赖一个观察：在图搜索的中间阶段，我们其实不太需要精确的距离计算，只需要方向没错就行）
4. 
---

DiskANN 把图索引搬到 SSD 之后，每跳一次都要随机 IO 一次。要让 recall@100 ≥ 0.9，`ef_search` 通常得开到 100 以上，这意味着一次查询要打 100+ 次 4KB 的随机读。后续几年所有所谓「DiskANN 优化」基本都在围绕这一点做文章——要么降 IO 次数，要么降单次 IO 的浪费，要么把那些被 IO 拖慢的计算重新利用起来。

<!--more-->

## DiskANN 到底慢在哪

先把原始 DiskANN 的搜索循环过一遍，方便后面对照：

- **压缩向量常驻内存**：PQ（或现在更常用的 RaBitQ）把原始向量压到 1/16 ~ 1/32，只占原始向量 5%–10%。
- **原始向量 + 邻居顺序写在 SSD 上**：每个点的邻居和它的全精度向量挨着存。
- 搜索维护两个队列：candidate set 用 PQ 距离驱动，决定下一跳走哪；result set 用全精度距离维护，保证最终精度。
- 每跳从 candidate 里取一个点，**IO 这个点的 block**，顺手拿到它的原始向量（算 exact 距离塞进 result）和它的邻居（算 PQ 距离塞进 candidate）。

问题就出在「每跳 IO 一次」。SSD 的最小读取单元是 4KB / 8KB，读 1KB 和读 4KB 的延迟一样。SIFT 一个 128 维向量才 512 字节，理论上一个 block 能装好几个，但 DiskANN 的布局是「一个点的向量 + 邻居占满一个 block」，顺带读上来的其他内容直接丢掉。这是优化空间最显眼的一块。

## Starling：让访问有空间局部性

Starling 的思路非常直接：既然一个 block 能放多个点，那就把图中**相邻的点尽量放进同一个 block**，一次 IO 解决多跳。

它的重排目标可以一句话讲清楚：

> 给定图索引，把所有点分配到固定大小（4KB）的 block 中，**最大化两端点落在同一 block 的边数量**。

原论文那个公式写得很复杂，本质上就是这一句。这是个 NP-完全问题，论文里走的是贪心近似——拿一个空 block，先挑权重最大的两个端点放进去，再不断从它们的邻居里挑边塞进来，直到 block 填满。

效果是肉眼可见的：DiskANN 默认的布局是 A1、A2、A3 按编号顺序排，相邻 block 之间几乎没关系；Starling 重排后，一个 block 里就是图上相邻的一簇点。

## 给边加权重：VLDB 2025 的小改进

Starling 的目标函数把所有边一视同仁，但实际搜索过程中**有的边明显比别的边更常被走到**。VLDB 2025 那篇就是顺着这个直觉做了一个加权版本。

权重是这么定义的：对一条边 $(u, v)$，统计有多少条「单调路径」会经过它到达 v，记作 $w(u, v)$；再统计有多少个起点 p 能通过单调路径到达 v，记作 $|P^*(v)|$。两者相乘作为这条边的权重，丢进 Starling 的目标里：

$$\max \sum_{(u,v)} w(u,v) \cdot |P^*(v)| \cdot \mathbb{1}[\text{block}(u) = \text{block}(v)]$$

`$\mathbb{1}$` 是指示函数，两端在同一 block 才算分。求解走的还是 Starling 那一套贪心，区别只是「优先把权重大的边塞进 block」。

值得一提的细节：

- 权重不需要额外开销算。Vamana/DiskANN 的 RNG 裁边过程本来就要遍历单调路径，顺手把这两个量记下来就行。
- 实测提升其实不大，尤其在 IND（query 分布和 base 一致）场景下。在 SIFT、GIST 这种典型数据集上几乎跟 Starling 重合，只在 OOD 数据集（如 Tiny5M）上 IO 次数有比较明显的下降。
- 比较反直觉的一点：用 query 集去测，边的访问频率并不像论文那样明显偏。论文里的「热边」结论是用 query = base 的极端实验得出来的，迁到真实查询上效果会打折扣。
- 它没改变搜索路径本身，只是把更重要的边尽量放进同一 block——所以不会像「为了加速去裁路径」那样把 recall 也降下去。

我们之前也尝试过类似思路，把热点点/边放进内存，但因为干涉了搜索路径本身（破坏了单调性），最后 recall 跌得很惨。Starling 这一系的好处就是：**只动布局，不动算法**。

## 几个挺反直觉的观察

接下来这篇综述性的工作（SIGMOD'26）有几个观察很值得拿出来单独说。

### 观察一：静态缓存的收益几乎在 0~1% 就饱和了

DiskANN 自带的静态缓存是「从 entry point 出发的前几跳的点和边」；Starling 的导航图也类似。这两种缓存随着内存容量从 0% 涨到 1%，QPS 有一次跳跃，之后几乎不再涨。

原因也好理解：搜索过程天然分两段——入口点到 query 附近、附近的精细搜索。**前一段确实能被缓存住，后一段散布在整个图上，缓存命中率被稀释到几乎为零。**

### 观察二：高维向量下 Starling 的重排直接失效

LINE-512 这种 512 维以上的向量，一个 4KB block 只放得下一个点。重排在物理上没有意义，Starling 和 DiskANN 的 QPS 在这种场景下几乎一模一样。

### 观察三：压缩率存在极值点

QPS 关于压缩率的曲线呈山形——

- 压缩太低（如 1/4、1/8）：PQ 距离计算变成主要瓶颈，反而比 IO 还慢。
- 压缩太高：精度掉得太快，需要更多次 IO 去补 recall。

极值点在 IND 上大约是 1/16 ~ 1/24，OOD 因为量化误差大要更低（约 1/8）。论文的做法是用 sample query 在线挑一个最优压缩率。

### 观察四：60% 的 rerank 就够了

如果把所有候选先用 PQ 距离维护，最后再统一 rerank，会发现**只 rerank top-60% 和 rerank top-100% 的 recall 几乎一样**。

这个观察很关键，它直接戳到 DiskANN 的设计假设：

> DiskANN 每跳都 IO 一个原始向量做精确距离，等价于每跳都做一次 rerank。但实际只有约 60% 的 rerank 是有效的——剩下 40% 的原始向量 IO 是白白浪费的。

这浪费在高维场景下更夸张：960 维的向量，每次 IO 都几乎被一整个原始向量吃掉，但其中大量根本进不了最终 top-k。

## 边缓存 + 延迟 rerank

顺着观察四，自然就有了下面这套方案（PipeANN 是早期代表，SIGMOD'26 这篇综述把它系统化）：

- **搜索过程只用量化距离驱动**，candidate 和 result 都按 PQ 距离维护，不再每跳 rerank。
- **边走 cache**：边的总体积比向量小一个量级，缓存 10% 内存就能盖住绝大部分边。命中就 0 IO，没命中才发起一次盘上 IO。
- **批量 rerank**：搜索结束后，对最终候选集合统一发一批 IO，用 io_uring 的异步队列把 SSD 带宽打满。

布局上他给出两种选择：

```
(a) Vector + Neighbors + 邻居的邻居（distance layer）
    一次 IO 顺带捞回邻居的邻居 PQ；
    SSD 体积大约是 DiskANN 的 1.5–2x，空间换 IO。

(b) 点边分离
    边连续、向量连续；搜索时只读边，rerank 时再批量读向量。
```

论文倾向 (a)，因为搜索过程中还能机会性地用上邻居距离；但如果叠加了 Starling 的重排，(b) 不一定比 (a) 差——一个 block 里的邻居本身就紧凑，重排的局部性放大了缓存效果。

实际跑下来，IO 队列很容易打满 SSD 带宽，整体瓶颈会迅速从单 IO 延迟转移到带宽。

## 从「以点为单位」到「以 block 为单位」

DiskANN 系工作的搜索粒度一直是「点」，block 只是物理承载。BAM（ICDE'26）和 SIGMOD'26 这篇综述里的另一个方案换了视角：**搜索粒度也改成 block**。

核心想法：

- 建图时优先保证 **block 内部连通**，因为 block 内的边不需要 IO。
- 然后用「block 内的边 + RNG 裁边」去裁 block 间的边。
- 这等价于把 Vamana 的「点的单调性」放宽成「IO 的单调性」——一次 IO 拉上来若干个点，整体能形成一条单调路径就行。
- 搜索时，每命中一个 block 就把里面所有点都算一遍精确距离，结束后再按 candidate 拉原始向量做最后一次 rerank。

SIGMOD'26 那一篇还顺手把入口点选择从「导航图」换成 LSH，少了维护边的开销，效果差不多——我个人感觉是凑创新点。它另一个值得提的设计是**极致内存优化**：把压缩向量也放到 SSD（紧跟在每个 block 后面），内存只留 graph cache 和 vector cache 的一小部分。不过它默认配置里其实还是把全部 PQ 放在内存里，SSD 上那份是冗余的——属于「写在论文里，实际不一定开启」的设计。

## 高维向量的真正瓶颈：算不过来

到 960 维以上，roofline 模型就开始踩在算力侧而不是 IO 侧。即便 IO 已经异步流水起来，每跳算几百个 PQ 距离的开销也会成为新的瓶颈。

SIGMOD'26 给出的解法是**冗余存储 + SIMD 友好布局**：

- 原版 DiskANN 的 PQ 散布在各点附近，访存是随机的，FastScan 没法用。
- 把邻居的 PQ 集中、连续地放在一起（一个 PQ 在多个 block 里被冗余存储），就能跑 FastScan，让 SIMD 把距离计算批量并行掉。
- 代价同样是 SSD 体积放大。

另一个正交的优化是**降维 + 长尾方差量化**。原始向量 PCA 之后，各维方差呈长尾分布——前面几维方差大，后面方差小。如果对所有维度用同样比特数 PQ，是浪费的。SIGMOD R6 的 SAQ 就是顺着这个思路：

- 前一半维度用 RaBitQ 2-bit，后一半维度用 RaBitQ 1-bit；
- 在给定总 bit 预算下，用背包 DP 决定各维度分配多少比特。

这条路其实和 ADSampling、TribasePQ 这一系工作是同源的——都是利用 PCA 后的方差长尾去做更激进的剪枝/量化。

## 一些没人做的方向

聊到最后，能想到的开放空间大致是这几类：

**1. 缝合怪。** Starling 的重排 + PipeANN 的边 cache + SAQ 的量化 + FastScan 的 SIMD 布局，目前没有任何一个工作把这些全部缝起来。各自都是正交的，理论上叠加能再上一个台阶。

**2. Batch query 调度。** 现在所有 layout/重排都是在 base 向量上做的，本质是 OOD 场景下的猜测。但生产系统里查询天然是 batch 的，多个 query 之间的 IO 必然有大量重合。基于一个 query batch 去重排或调度 IO，调度空间比单 query 大得多——尤其在网络 IO 的存算分离场景（block 不是 4KB 而是 MB 级），单 query 根本填不满一次 IO。

**3. 自适应 SSD。** roofline 的拐点和具体 SSD 强相关，差的 SSD 上 IO 占主导，好的 SSD 上算力占主导。一个能根据 `compute/IO` 比自动选 layout 和算法的版本，比单点优化更有工程价值。

**4. 写入与删除。** 现在所有 layout 优化都是为读优化的，做完几乎不可能再实时插入——邻居和向量物理捆绑，插一个点要动一片。FreshDiskANN 那篇 update（删除等于先删后插）的思路是通用的，但跟 layout 没结合好。能不能在 buffer 写之外做一些「先不重排、后台 lazy 重排」的方案，是个开放问题。

ANN On Disk 这两年的工作密度很高，但本质上都还是在排列组合上面那几把锤子。真正的下一档，多半得换条赛道——要么场景变（batch、存算分离、新硬件），要么把读和写一起考虑。

## 参考资源

- DiskANN: Fast Accurate Billion-point Nearest Neighbor Search on a Single Node. NeurIPS 2019.
- Starling: An I/O-Efficient Disk-Resident Graph Index Framework. SIGMOD 2024.
- VLDB 2025 加权重排（Starling 扩展）。
- PipeANN：异步 IO 流水化的边缓存方案。
- BAM. ICDE 2026.
- SAQ：基于方差长尾的混合比特 RaBitQ。SIGMOD R6 2026.
- FreshDiskANN：DiskANN 上的 update 机制。
- https://zhuanlan.zhihu.com/p/655865563