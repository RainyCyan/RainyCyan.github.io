---
title: 向量检索系列：图 ANN 索引结构的演进
date: 2026-05-28T17:52:00+08:00
tags: [向量检索, ANN, HNSW, NSG, DiskANN]
series: [向量检索]
featured: true
description: "从 NSW、HNSW 到 NSG/SSG、再到 DiskANN/Vamana 与 SPANN：聊聊图 ANN 索引这十年都在解决什么问题，又是怎么一步步走到今天的。"
draft: true
---
## 前言
<!-- 不要堆砌技术名词，遵循 准确，简单 的语言风格 -->


<!--more-->

## 背景：为什么是"图"
ANN要解决的问题到底是什么？距离计算？快速搜索！
图的结构——点+边的形式天然适合存储邻居关系，在近邻图上执行搜索算法

长边到底是什么？区域连通，快速跳转只是副作用。

对图结构的限制：我们想要什么样的图？近邻图+导航图+分层图=可导航的近邻小世界图：局部近邻、全局联通、少量长边的稀疏图

## 朴素近邻图

## NSW：一切的起点


## HNSW：分层带来的飞跃


## NSG / SSG：更"瘦"的图


## DiskANN / Vamana：从内存走向磁盘


## SPANN / SPFresh：超大规模与增量更新


## 横向对比与选型


## 小结

## 参考资源
- https://zhuanlan.zhihu.com/p/610454162