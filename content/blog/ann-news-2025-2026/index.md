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
> 这是一个**模板**：复制一个 `.timeline-item` 块，改日期、标题、正文、标签即可新增一条。

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
    <div class="tl-date">2026-01</div>
    <div class="tl-card">
      <p class="tl-title">条目标题（示例）</p>
      <p class="tl-body">在此填写该进展的一句话摘要与关键信息：来源、核心贡献、指标提升等。删除本段落文字，替换为真实内容。</p>
      <div class="tl-tags"><span>Graph ANN</span><span>Filtered Search</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2025-09</div>
    <div class="tl-card">
      <p class="tl-title">条目标题（示例）</p>
      <p class="tl-body">复制上方 <code>.timeline-item</code> 块并修改即可新增一条时间线。保持日期从新到旧排列。</p>
      <div class="tl-tags"><span>Quantization</span><span>GPU</span></div>
    </div>
  </div>

  <div class="timeline-item">
    <div class="tl-date">2025-03</div>
    <div class="tl-card">
      <p class="tl-title">条目标题（示例）</p>
      <p class="tl-body">最早的一条放在最底部。圆圈 + 竖线会自动对齐，无需手动调整。</p>
      <div class="tl-tags"><span>Disk ANN</span><span>Benchmark</span></div>
    </div>
  </div>

</div>
