---
title: 范围过滤 ANN 各方法思路精华（Digest）
date: 2026-06-02T12:00:00+08:00
tags: [向量检索, 范围过滤, Range Filter, ANN, WoW, IRangeGraph, DIGRA, SIGMOD]
series: [向量检索]
draft: false
description: "范围过滤 ANN 从 IRangeGraph 到 WoW 各方法的思路速查：核心结构、关键工夫、代价三点式对比。"
---

带范围过滤的 ANN：查询同时带一个向量和一个标量区间 `[l, r]`，在区间内的向量里找最近邻。以下把 IRangeGraph → DIGRA → SERF/DSG → HSIG → WoW 这条线上各方法的思路提炼成「核心结构 / 关键工夫 / 代价」三点式。

## 背景：为什么不能直接套图索引

- **pre-filtering**：先按属性过滤出区间内的点，再暴力算距离。选择度极高（区间只命中万分之几）时最优——索引导航开销已超过直接算距离。
- **post-filtering**：先取 top-k' 再筛掉区间外的。只在选择度低（大部分点在区间内）时有效。
- **搬到图索引上就暴露问题**：pre-filtering 只走合法邻居 → 合法点少时连通性塌，搜两步就停、召回断崖；post-filtering 让非法点参与导航 → 路径变长，大量算力花在区间外。
- **共通解**：用一种数据结构管理「按属性切分的多份图」，让查询只动相关那部分图，把过滤固化进索引结构。

## 方法对比总表

| 方法 | 核心结构 | 关键工夫 | 代价 / 约束 |
|---|---|---|---|
| **IRangeGraph** (SIGMOD'25) | 属性离散化建线段树，每个节点建一张图 | query range 拆成 O(log n) 节点合并结果；查询时自顶向下**动态选边**、跳过冗余覆盖层 | 图多（n=1e8 建 30+ 张 HNSW，索引 4-5 倍）；**纯静态**，不支持插入 |
| **DIGRA** | 线段树换成 **B+ 树**（结构同构，每段建图） | 建图时子图合并进右子图；查询用**近似 range** 逼近实际 range（最坏 4 倍定理），放宽过滤精度换扫描节点数 | 插入比线段树轻，但 split 仍需重建该段图 + 标记删除，**流式大规模插入仍重** |
| **SERF / DSG** (SIGMOD'24) | 把所有 `[l,r]` 对应的图**无损压缩到一张图** | 每条边有「生存期」区间 `[b,e]`，只在 r 落在区间内时活跃；推广到任意 `[l,r]` 成 2D Segment Graph（边=五元组） | 索引降到单张 HNSW 量级，但**只能按属性顺序静态构建**；DSG 试图动态化，边数膨胀、无静态时的代差优势 |
| **HSIG / UNIFY** (VLDB'25) | 把 pre/post/hybrid **集成到一张图** | 按深度直方图分 s 段，点在本段连 KNN + 每个其他段各连一组 KNN；邻居链表分 s 组 + 跳表(pre) + bitmap(post)，两阈值 τ_a/τ_b 选策略 | **支持流式插入**（整套 incremental）；空间约 3 倍；corner case（2^-10）不如特化方案 |
| **WoW** (SIGMOD'26 R2) | **层次化窗口图**（见下） | 窗口图逼近 Oracle Graph + WBT 选着陆层 + 两阶段剪枝 + 延迟淘汰 | **无序流式插入 + 全选择度 + 接近 Oracle 图质量**同时做到；索引约 HNSW 的 2-3 倍 |

## WoW 核心：层次化窗口图

- **Oracle Graph**：若提前知道 query range `[l,r]`，只对区间内合法点单独建一张图就是当前查询最理想的邻近图。所有 range filter ANN 本质都在用通用图逼近它。
- **窗口边**：给定窗口 w，点 i 只在 `|rank(j)-rank(i)| ≤ w/2` 内选邻居；窗口内邻居属性上接近，天然符合 Oracle Graph 该有的结构。
- **多层叠加**：单一窗口不够（如 `(15,14)` 在 w=4 下被 16 抢占，需要 15 有个 w=2 的更小窗口图才重现）。层号 L，窗口大小 `2·O^L`，**扩张系数 O=4 最优**。
- 每一层都是**全量节点**（只是连边不同），不像 HNSW 上稀下密。

```
L=2 大窗口(w=32) → 覆盖全局，粗导航
L=1 中窗口(w=8)  → 中等选择度过滤
L=0 小窗口(w=2)  → 高选择度，几乎全是合法邻居
```

## WoW 搜索：跨层动态选边

- 每一层都是完整 NSW，**每一跳同时翻多个层级的邻居表**——高层跳得远但噪声大、低层边少但合规，同时利用两者、避免局部最优。
- **早停策略**：某一层邻居全部合规就不再往更低层翻（已能提供高质量合法导航，继续往下纯浪费）。
- **着陆层**：用 WBT 在 O(log n) 内估算区间内合法点数 N'，挑窗口尺度最接近 N' 的层作起点——该层结构上最接近当前 query 的 Oracle Graph。

## WoW 插入：无序流式，局部修补

1. **算窗口**：WBT 在 O(log n) 内算出 v 在每层的窗口范围；WBT 是轻量插件，与图解耦（不像 RangePQ 树与索引紧耦合）。
2. **找候选邻居**：从高层往下每层做 beam search 找 M 个候选，**复用上一层结果**（定理：高层窗口邻居平均更近，且符合低层窗口者必存在于低层）。
3. **建边 + 两阶段剪枝**：只连 M/2 条边留余量；邻居表满触发剪枝——① 窗口检查（删已明显出窗口的旧邻居）② RNG 剪枝（删冗余边）。
4. **延迟剪枝**：不立即删「刚因新点插入失去合法性」的旧边，只在邻居表真满时才剪——动态图后续窗口会变，过早删边破坏导航稳定性。

**本质：所有维护操作都是局部的**，新点只影响 WBT 窗口附近的连接，无需线段树/B+ 树式全局重构——这是 WoW 能在持续无序插入下维持图质量的核心。插入最坏 O(log²n)，因大量复用实测开销接近 HNSW，16 线程并行扩展 5-6.9x。

## 参考资源

- WoW: Hierarchical Window Graphs for Range-Filtered ANN Search. SIGMOD 2026 (R2). 南京大学.
- IRangeGraph: Improvising Range-dedicated Graphs. SIGMOD 2025.
- DIGRA: Dynamic Insertable Graph for Range-filtering ANN.
- SERF: Segment Graph for Range-Filtering ANN. SIGMOD 2024. / DSG: Dynamic Segment Graph.
- UNIFY / HSIG: A Unified Index for Hybrid Range-Filtering ANN. VLDB 2025.
