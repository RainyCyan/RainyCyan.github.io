---
title: "ANN News (2025–2026)"
date: 2026-08-17T10:00:00+08:00
lastmod: 2026-08-17
tags: [ANN, Vector Search, Timeline, News]
series: []
featured: false
description: "近似最近邻（ANN）领域 2025–2026 年重要进展的竖向时间线，按更新时间倒序排列。"
draft: false
ShowToc: false
TocOpen: false
---

> 近似最近邻（Approximate Nearest Neighbor, ANN）领域 2025–2026 的进展时间线。按时间倒序排列（最新在上），竖线 + 圆圈标记。
>
> 新增一条：复制任意一个 `.timeline-item` 块，改日期、标题、正文、标签即可。区间日期（如 `2026-07/08`）按其**较晚月份**排序。

<style>
.ann-timeline{
  --line:#d0d7de;
  --dot:#1d4ed8;
  --dot-ring:#dbeafe;
  --card:#ffffff;
  --card-border:#e5e7eb;
  --muted:#6b7280;
  position:relative;
  margin:2rem 0;
  padding-left:2.25rem;
}
/* 竖线 */
.ann-timeline::before{
  content:"";
  position:absolute;
  left:0.5rem;
  top:0.35rem;
  bottom:0.35rem;
  width:2px;
  background:var(--line);
}
.timeline-item{
  position:relative;
  margin:0 0 1.75rem 0;
}
.timeline-item:last-child{ margin-bottom:0; }
/* 圆圈标记 */
.timeline-item::before{
  content:"";
  position:absolute;
  left:-1.97rem;
  top:0.25rem;
  width:0.85rem;
  height:0.85rem;
  border-radius:50%;
  background:var(--dot);
  border:3px solid var(--dot-ring);
  box-shadow:0 0 0 1px var(--line);
}
.timeline-item .tl-date{
  display:inline-block;
  font-size:0.8rem;
  font-weight:600;
  letter-spacing:0.02em;
  color:var(--dot);
  margin-bottom:0.3rem;
}
.timeline-item .tl-card{
  background:var(--card);
  border:1px solid var(--card-border);
  border-radius:0.6rem;
  padding:0.85rem 1.1rem;
}
.timeline-item .tl-title{
  margin:0 0 0.35rem 0;
  font-size:1.05rem;
  font-weight:700;
  line-height:1.35;
}
.timeline-item .tl-body{
  margin:0;
  font-size:0.92rem;
  line-height:1.6;
  color:#374151;
}
.timeline-item .tl-tags{
  margin-top:0.55rem;
  display:flex;
  flex-wrap:wrap;
  gap:0.35rem;
}
.timeline-item .tl-tags span{
  font-size:0.72rem;
  color:var(--muted);
  background:#f3f4f6;
  border:1px solid var(--card-border);
  border-radius:999px;
  padding:0.1rem 0.55rem;
}
@media (prefers-color-scheme: dark){
  .ann-timeline{
    --line:#30363d;
    --card:#161b22;
    --card-border:#30363d;
    --dot-ring:#1e293b;
    --muted:#8b949e;
  }
  .timeline-item .tl-body{ color:#c9d1d9; }
  .timeline-item .tl-tags span{ background:#21262d; }
}
</style>

<div class="ann-timeline">

  <div class="timeline-item">
    <div class="tl-date">2026-08-09</div>
    <div class="tl-card">
      <p class="tl-title">InSituANN — PCIe 感知的十亿级向量搜索</p>
      <p class="tl-body">核心观察：GPU 高维向量搜索的真正瓶颈是 PCIe 数据传输而非 GPU 计算。将原始向量保留在 CPU RAM 做细搜索，GPU 仅负责紧凑路由/剪枝。SIFT-1B 上 IVF 构建仅 5.2 分钟（HNSW 需 30.4 小时），匹配召回下比 DiskANN 快约 2.4–4.6 倍，已开源。截至 2026-08-17 最新系统工作。</p>
      <div class="tl-tags"><span>GPU</span><span>PCIe</span><span>Billion-scale</span><span>Open Source</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2026-07/08</div>
    <div class="tl-card">
      <p class="tl-title">Memory-Constrained DiskANN（SISAP 2026）</p>
      <p class="tl-body">针对 2025 年 SISAP 索引挑战赛 Task 1（2300 万 × 384 维向量在严格内存限制下搜索），提出结合 PCA 与第二分配机制的优化方案。</p>
      <div class="tl-tags"><span>DiskANN</span><span>SISAP 2026</span><span>Memory-Constrained</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2026-07</div>
    <div class="tl-card">
      <p class="tl-title">DiskANN3 — 可组合向量索引原语库（Microsoft）</p>
      <p class="tl-body">DiskANN 不再只是一个实现，而成为可嵌入不同数据库的向量索引原语库。核心抽象：DiskANN algorithm + DataProvider(RAM/SSD/DB)，提供 vector update、query API、量化、BF-tree provider 等模块。ANN 算法 → 可复用数据库组件。</p>
      <div class="tl-tags"><span>DiskANN</span><span>Primitives</span><span>Microsoft</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2026-06-24</div>
    <div class="tl-card">
      <p class="tl-title">ETALE — GPU Dynamic Graph ANN</p>
      <p class="tl-body">首批 GPU-native dynamic graph ANN 之一，支持 GPU 上 streaming insertion/deletion，无需 global rebuild；采用 lock-free copy-on-write graph，在查询持续运行时进行拓扑更新。标志 Dynamic ANN 从 CPU 图索引进一步进入 GPU-native 阶段。</p>
      <div class="tl-tags"><span>GPU</span><span>Dynamic</span><span>Graph ANN</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2026-06-15</div>
    <div class="tl-card">
      <p class="tl-title">VectorDB 综合综述（arXiv）</p>
      <p class="tl-body">对向量数据库存储与检索技术的综合综述，涵盖设计原理与演进，并对 Milvus、Weaviate、Qdrant 等主流方案深入对比。特别指出结合 LLM 的新兴机遇，如新型索引策略。</p>
      <div class="tl-tags"><span>Survey</span><span>Vector DB</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2026-06-03</div>
    <div class="tl-card">
      <p class="tl-title">ANN Evaluation: Recall What Matters — 重新定义检索质量指标</p>
      <p class="tl-body">提出 1/Ratio@k 替代 Recall@k，关注"实际最近邻距离与近似距离之比"。实验发现：下游分类/RAG 任务中，1/Ratio@k 比 Recall@k 更接近真实效用。</p>
      <div class="tl-tags"><span>Evaluation</span><span>Metrics</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2026-06-02</div>
    <div class="tl-card">
      <p class="tl-title">Puffin-backed Vector Index — ANN + Apache Iceberg 快照</p>
      <p class="tl-body">将 ANN 索引嵌入 Apache Iceberg 的 Puffin 文件，索引生命周期、快照、时间旅行、一致性、垃圾回收全部复用 Iceberg 机制。这是"Vector Index → Data Lake Physical Index"的明确范例。</p>
      <div class="tl-tags"><span>Data Lake</span><span>Iceberg</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2026-06</div>
    <div class="tl-card">
      <p class="tl-title">Dynamic Graph Random Walk（ICLR 2026 正式）</p>
      <p class="tl-body">此前预印本工作正式在 ICLR 2026 发表，提供 HNSW 删除的理论基础与工程实践（随机游走理论框架 + 确定性删除算法）。</p>
      <div class="tl-tags"><span>HNSW</span><span>Delete</span><span>ICLR 2026</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2026-06</div>
    <div class="tl-card">
      <p class="tl-title">Fast Distance Survey — 快速距离计算的统一区间估计框架（HKUST-GZ）</p>
      <p class="tl-body">系统综述加速 ANN 距离计算的两种互补技术：Fast Distance Calculation（向量量化压缩）与 Fast Distance Comparison（边界剪枝）。提出统一区间估计框架，将两类方法统一为对真实距离生成边界、可靠性和成本的三元组，揭示两者本质的设计权衡。</p>
      <div class="tl-tags"><span>Survey</span><span>Distance</span><span>Quantization</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2026-06</div>
    <div class="tl-card">
      <p class="tl-title">OpenSearchCon Europe 2026 — 真实生产负载下的三引擎对决</p>
      <p class="tl-body">Zeta Alpha 对比 Lucene HNSW、Faiss、jVector 在真实生产负载（布尔过滤、聚合、混合查询）下的表现。结论：性能差异根源在索引构建、内存管理、并发处理等系统细节，而非原始速度，且没有单一引擎通吃所有场景，揭示了工业界基准测试的"剧场效应"。</p>
      <div class="tl-tags"><span>Benchmark</span><span>OpenSearch</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2026-05-29</div>
    <div class="tl-card">
      <p class="tl-title">Cloud-Native Vector Search Storage Survey（SIGMOD Companion 2026 正式发表）</p>
      <p class="tl-body">前述存储架构综述正式进入 SIGMOD Companion，确认架构主线从 index-centric 转向 storage-centric。</p>
      <div class="tl-tags"><span>Survey</span><span>Storage</span><span>SIGMOD</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2026-05-26</div>
    <div class="tl-card">
      <p class="tl-title">SilverTorch 正式公开（Meta Engineering Blog）</p>
      <p class="tl-body">Meta 正式公开 SilverTorch 技术细节，确认在生产环境部署。80M-item 端到端：23.7× requests/s，约 20.9× TCO 效率提升。检索层从"数据库的事"变为"模型的事"。</p>
      <div class="tl-tags"><span>Meta</span><span>Recommendation</span><span>GPU</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2026-05</div>
    <div class="tl-card">
      <p class="tl-title">PathSeer — 自适应过滤 ANN 搜索（PACMMOD/SIGMOD 2026）</p>
      <p class="tl-body">在同一个图索引中同时支持两种邻域处理策略（distance-first / filter-first），遍历过程中动态调整比例。相比现有方法，过滤 ANN 搜索吞吐量最高提升约 47.4 倍。</p>
      <div class="tl-tags"><span>Filtered Search</span><span>Adaptive</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2026-05</div>
    <div class="tl-card">
      <p class="tl-title">边缘设备 HNSW 实时维护（DASFAA 2026）</p>
      <p class="tl-body">聚焦边缘设备频繁更新下的节点孤立问题，提出轻量级实时维护框架，动态检测并修复孤立节点。实验显示在 &lt;10% 延迟开销和 &lt;2% 额外内存开销下稳定召回率。</p>
      <div class="tl-tags"><span>Edge</span><span>HNSW</span><span>Maintenance</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2026-05</div>
    <div class="tl-card">
      <p class="tl-title">VecBench — Filtered ANN 可控基准（2026-03/05）</p>
      <p class="tl-body">支持控制数据规模、维度、filter selectivity、filter/query correlation 和 workload，尝试使用 holistic end-to-end 指标评估 Filtered Vector Search。解决传统 FANNS benchmark 过于简单、缺少真实高维数据和 filter correlation 的问题。</p>
      <div class="tl-tags"><span>Filtered Search</span><span>Benchmark</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2026-04</div>
    <div class="tl-card">
      <p class="tl-title">Ada-ef / FGIM 正式进入 SIGMOD — 学术认可节点</p>
      <p class="tl-body">Ada-ef 与 FGIM 的正式 PACMMOD 版本于 2026 年发表。query-adaptive ANN 与图索引合并已从探索性想法发展为正式数据库研究方向。</p>
      <div class="tl-tags"><span>SIGMOD</span><span>Adaptive</span><span>Index Merge</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2026-03-26</div>
    <div class="tl-card">
      <p class="tl-title">GateANN — SSD + Filtered ANN</p>
      <p class="tl-body">将 graph traversal 与 vector retrieval 解耦：过滤和图路由只使用内存中的 neighbor/approximate distance，只有必要节点才触发 SSD vector 读取，避免 post-filter 造成的大量无效 I/O，也避免重建 filter-aware graph。标志 Filtered ANN 与 Disk ANN 开始深度融合。</p>
      <div class="tl-tags"><span>Filtered Search</span><span>SSD</span><span>I/O</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2026-03/04</div>
    <div class="tl-card">
      <p class="tl-title">CD-ANN — 客户端/边缘设备 ANN</p>
      <p class="tl-body">面向资源受限边缘设备，采用 segmented HNSW + on-demand segment caching，内存减少约 10–82%，插入延迟降低约 70–97%。ANN 开始从数据中心走向边缘。</p>
      <div class="tl-tags"><span>Edge</span><span>HNSW</span><span>Caching</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2026-03-23</div>
    <div class="tl-card">
      <p class="tl-title">FGIM — 图索引快速合并框架（PACMMOD/SIGMOD 2026）</p>
      <p class="tl-body">Proximity graph → kNN graph → kNN refinement → proximity graph 流程实现图索引合并。相比 HNSW 增量构建最高快约 3.5 倍，对不支持增量的方法平均快约 7.9 倍。</p>
      <div class="tl-tags"><span>Graph Merge</span><span>SIGMOD</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2026-03-23</div>
    <div class="tl-card">
      <p class="tl-title">GateANN — I/O 高效的 SSD 过滤向量检索</p>
      <p class="tl-body">精准指出过滤向量检索的 I/O 困境：后过滤在低选择性下浪费 SSD 读取，预过滤破坏图连通性（召回率最高仅 ~57%）。核心创新 Graph Tunneling：在内存中检查节点过滤谓词，对不匹配节点完全在内存中路由，避免 SSD 读取。实验显示 SSD 读取减少最高 10 倍，吞吐量提升最高 7.6 倍。</p>
      <div class="tl-tags"><span>Filtered Search</span><span>SSD</span><span>I/O</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2026-03-13</div>
    <div class="tl-card">
      <p class="tl-title">d-HNSW — RDMA / Disaggregated Memory Vector Search</p>
      <p class="tl-body">面向 RDMA 远程内存重新设计 HNSW，包括代表性索引缓存、RDMA-friendly 布局、query-aware loading 和 pipeline 执行，解决 pointer chasing、远程内存碎片和冗余传输。标志 ANN 开始进入 disaggregated memory 架构。</p>
      <div class="tl-tags"><span>RDMA</span><span>Disaggregated Memory</span><span>HNSW</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2026-03-01</div>
    <div class="tl-card">
      <p class="tl-title">PAG — 投影增强图索引（ICML 2026）</p>
      <p class="tl-body">将投影技术集成到图索引，通过统计检验减少不必要的精确距离计算。在 6 个现代数据集上 QPS 最高达 HNSW 约 5 倍。核心思想：优化方向不限于图遍历本身，也可减少精确距离计算次数。</p>
      <div class="tl-tags"><span>Graph ANN</span><span>Projection</span><span>ICML 2026</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2026-02-27</div>
    <div class="tl-card">
      <p class="tl-title">IVF-RaBitQ（GPU）— GPU 原生 ANN</p>
      <p class="tl-body">GPU 原生实现 IVF+RaBitQ 量化，Recall≈0.95 时 QPS 为 CAGRA 约 2.2 倍，索引构建快约 7.7 倍，比 IVF-PQ 吞吐量高约 2.7 倍，无需原始向量重排序。RaBitQ 从压缩算法进化为 index/hardware 协同设计原语。</p>
      <div class="tl-tags"><span>GPU</span><span>Quantization</span><span>RaBitQ</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2026-02-26</div>
    <div class="tl-card">
      <p class="tl-title">VeloANN — SSD 常驻图索引优化</p>
      <p class="tl-body">通过协程异步 I/O + 记录染色布局 + 记录级缓冲池 + cache-aware beam search，解决图遍历随机访问导致的 SSD 读放大。达内存系统约 92% 吞吐量，仅用约 10% 内存，吞吐量最高提升约 5.8 倍。</p>
      <div class="tl-tags"><span>SSD</span><span>Graph</span><span>I/O</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2026-02-25</div>
    <div class="tl-card">
      <p class="tl-title">AQR-HNSW — 密度感知量化 + 多阶段重排序（DAC 2026）</p>
      <p class="tl-body">将 density-aware quantization、multi-stage reranking 和 SIMD 优化组合进 HNSW，报告约 4× 压缩、75% graph memory reduction、2.5–3.3× QPS 和 &gt;98% recall。代表"量化 + rerank + 硬件优化"的协同路线。</p>
      <div class="tl-tags"><span>HNSW</span><span>Quantization</span><span>Rerank</span><span>DAC 2026</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2026-02-24</div>
    <div class="tl-card">
      <p class="tl-title">DiskANN I/O 三维优化框架（VLDB 2026）— 中国电信云计算研究院</p>
      <p class="tl-body">系统剖析 DiskANN 在内存布局、磁盘布局和搜索算法三方面的 I/O 缺陷，提出三维优化框架及 Page 级复杂度模型。DiskANN 被 Azure、华为 GaussDB 广泛采用，但 70%–90% 查询延迟来自 I/O。这项研究填补了磁盘 ANN I/O 优化的系统性空白。</p>
      <div class="tl-tags"><span>DiskANN</span><span>I/O</span><span>VLDB 2026</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2026-02-19</div>
    <div class="tl-card">
      <p class="tl-title">Multiple Index Merge — 多 ANN 索引合并框架</p>
      <p class="tl-body">提出 Reverse Neighbor Sliding Merge 和 Merge Order Selection，解决大规模数据分片建多个索引后的合并问题。最高比现有合并方案快约 5.48 倍，比重建快约 9.92 倍，可扩展至 100M 向量/50 分区。Index Merge 从工程技巧上升为独立研究问题。</p>
      <div class="tl-tags"><span>Index Merge</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2026-02-11</div>
    <div class="tl-card">
      <p class="tl-title">Filtered ANN in Vector Databases（UC Merced）— FAISS/Milvus/pgvector 系统比较</p>
      <p class="tl-body">提出 MoReVec 和 GLS 指标，系统研究 Filtered ANN 在真实 Vector DB 中的执行策略。发现引擎级 query planning 可能比底层 ANN 算法更重要，低 selectivity 下 IVFFlat 可能优于 HNSW。Filtered ANN 开始从"索引问题"演化为"查询优化问题"。</p>
      <div class="tl-tags"><span>Filtered Search</span><span>Vector DB</span><span>Query Planning</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2026-02-10</div>
    <div class="tl-card">
      <p class="tl-title">GPU Graph ANN Survey — GPU-Accelerated Algorithms for Graph Vector Search</p>
      <p class="tl-body">系统比较 6 种 GPU 图 ANN 算法和 8 个大规模数据集，指出 distance computation 仍是主要计算瓶颈，而大规模场景下 CPU-GPU 数据传输成为关键系统瓶颈。GPU ANN 开始形成独立研究体系。</p>
      <div class="tl-tags"><span>GPU</span><span>Graph ANN</span><span>Survey</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2026-01-05</div>
    <div class="tl-card">
      <p class="tl-title">Vector Search Storage Architecture Survey（SIGMOD Companion 2026）</p>
      <p class="tl-body">将向量搜索架构演进概括为：Memory-resident → RAM+SSD → heterogeneous storage → RAM+SSD+对象存储 → cloud-native/万亿级。架构主线从 index-centric 转向 storage-centric。</p>
      <div class="tl-tags"><span>Survey</span><span>Storage</span><span>SIGMOD</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2025-12-28</div>
    <div class="tl-card">
      <p class="tl-title">OrchANN — I/O 驱动的外存向量搜索框架</p>
      <p class="tl-body">统一 I/O 编排框架，通过 query-aware 导航图 + 多级剪枝 + SSD I/O 协调应对偏斜负载。相比 SPANN，QPS 最高提升约 17.2 倍，延迟降低约 25 倍。标志 Disk ANN 进入"专门设计 I/O 执行引擎"阶段。</p>
      <div class="tl-tags"><span>Disk ANN</span><span>I/O</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2025-12-20</div>
    <div class="tl-card">
      <p class="tl-title">Dynamic Quantization（Streaming Updates）</p>
      <p class="tl-body">理论证明：有界磁盘 I/O 下，data-dependent 量化可动态维护而不损失 ANN 精度；开发了"动态一致性"的实用量化方法。将 dynamic ANN 从"动态图"推进到"动态图 + 动态量化器"。</p>
      <div class="tl-tags"><span>Quantization</span><span>Streaming</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2025-12-19</div>
    <div class="tl-card">
      <p class="tl-title">Dynamic Graph ANN（Amazon）— HNSW 删除问题的理论突破（ICLR 2026）</p>
      <p class="tl-body">首次建立随机游走理论框架分析图索引删除操作，设计确定性删除算法，在查询延迟、召回率、删除时间和内存之间取得更优权衡。HNSW 从此真正支持 delete 而非仅 insert。</p>
      <div class="tl-tags"><span>HNSW</span><span>Delete</span><span>ICLR 2026</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2025-12-05</div>
    <div class="tl-card">
      <p class="tl-title">Distance Skipping — 距离计算优化（SIGMOD 2026）</p>
      <p class="tl-body">针对高维 ANN 中 Distance Comparison Operation 成为瓶颈的问题，识别并跳过冗余距离计算。与 TRIM 共同代表一条新路线：不改变核心图结构，而是直接减少昂贵 distance computation。</p>
      <div class="tl-tags"><span>Distance</span><span>SIGMOD 2026</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2025-12 / 2026-02</div>
    <div class="tl-card">
      <p class="tl-title">AQR-HNSW — 密度感知量化 + 多级重排序</p>
      <p class="tl-body">提出密度感知自适应量化，实现 4 倍压缩的同时保持距离关系；结合多级重排序策略，显著加速 HNSW 搜索。验证了"压缩 + 重排序"组合在 HNSW 上的协同优化潜力。</p>
      <div class="tl-tags"><span>HNSW</span><span>Quantization</span><span>Rerank</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2025-11-25</div>
    <div class="tl-card">
      <p class="tl-title">SilverTorch（Meta）— Index as Model：推荐系统检索范式革命</p>
      <p class="tl-body">将 ANN、过滤、打分、重排序全部融合为 GPU/PyTorch 模型，采用 Int8 + fused GPU kernel + Bloom filter 实现端到端检索。80M 数据量下达 23.7× requests/s，TCO 效率提升约 20.9 倍。正式论文进入 SIGIR 2026。</p>
      <div class="tl-tags"><span>Meta</span><span>Recommendation</span><span>SIGIR 2026</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2025-10-14</div>
    <div class="tl-card">
      <p class="tl-title">OpenSearch 3.3 — 工业级向量引擎优化</p>
      <p class="tl-body">向量索引构建加速约 9 倍，存储减少约 3 倍，查询延迟降低约 55%，合并时间减少约 40%，并扩展 GPU 向量索引支持。标志工业界重心转向索引 + 压缩 + GPU + 查询执行的系统级优化。</p>
      <div class="tl-tags"><span>OpenSearch</span><span>GPU</span><span>Engineering</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2025-09-23</div>
    <div class="tl-card">
      <p class="tl-title">DARTH — Declarative Recall（SIGMOD 2026）</p>
      <p class="tl-body">不再要求用户手工调 ef/nprobe，而是直接指定目标 Recall，系统根据 query 搜索过程动态预测 Recall 并提前终止。HNSW 最高 14.6×、IVF 最高 41.8× 加速。标志 ANN 从"参数调优"走向"声明式搜索目标"。</p>
      <div class="tl-tags"><span>Declarative</span><span>Recall</span><span>SIGMOD 2026</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2025-09-15</div>
    <div class="tl-card">
      <p class="tl-title">Lucene-on-Faiss（OpenSearch）— Lucene + Faiss 混合向量引擎</p>
      <p class="tl-body">将 Faiss SIMD distance computation 与 Lucene 的 memory-efficient vector loading、HNSW 执行结合；32× 量化、k=100 场景下 QPS 接近翻倍，recall 下降最高约 4.5%。代表工业向量引擎从"单一 ANN 实现"转向系统级组合。</p>
      <div class="tl-tags"><span>OpenSearch</span><span>Faiss</span><span>Lucene</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2025-09-07</div>
    <div class="tl-card">
      <p class="tl-title">DISTRIBUTEDANN（Microsoft Research）— 50B 向量单一 DiskANN 图分布到 1000+ 台机器</p>
      <p class="tl-body">将单一 DiskANN 图分布到千台机器，实现 50B 向量、26ms 延迟、100K QPS，比传统分区路由架构效率提升约 6 倍，已应用于 Bing 搜索引擎。</p>
      <div class="tl-tags"><span>DiskANN</span><span>Distributed</span><span>Microsoft</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2025-08-25</div>
    <div class="tl-card">
      <p class="tl-title">TRIM — 高维 ANN 距离剪枝</p>
      <p class="tl-body">重新设计 triangle-inequality pruning，通过优化 landmark 和可调 lower-bound relaxation，在 HNSW/IVFPQ/DiskANN 中减少距离计算与 I/O。memory-based graph search 最高提升约 90%，DiskANN I/O 最高降低约 58%。标志 ANN 优化从"少访问节点"进一步转向"少计算距离"。</p>
      <div class="tl-tags"><span>Distance</span><span>Pruning</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2025-08-12</div>
    <div class="tl-card">
      <p class="tl-title">Tagore — GPU 加速图索引构建</p>
      <p class="tl-body">提出 GPU-native GNN-Descent、通用 GPU 剪枝 kernel 以及 GPU-CPU-Disk 异步建图框架，支持 NSG/Vamana 等图索引；7 个真实数据集上报告 1.32×–112.79× 构建加速。标志 ANN 从 GPU 搜索进一步走向 GPU 建图。</p>
      <div class="tl-tags"><span>GPU</span><span>Graph Build</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2025-06/07</div>
    <div class="tl-card">
      <p class="tl-title">Filtered ANN Benchmark &amp; Survey（ETH Zurich, SPCL）</p>
      <p class="tl-body">对带过滤的 ANN 搜索（FANNS）进行全面综述与分类，指出现有评估缺乏基于最新 Transformer 嵌入的真实数据集。团队发布超 270 万篇 arXiv 论文摘要的新数据集（含 11 种真实属性）。基准发现无单一方法通吃：ACORN 支持多过滤器但性能非最优，SeRF 在有序属性上优异但不支持分类属性，Filtered-DiskANN 与 UNG 在中型数据集表现好但大型数据集失效。</p>
      <div class="tl-tags"><span>Filtered Search</span><span>Benchmark</span><span>Dataset</span></div>
    </div>
  </div>

</div>
