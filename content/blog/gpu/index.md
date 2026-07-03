---
title: "GPU 基础：架构、执行模型与显存层级"
date: 2026-06-30T12:00:00+08:00
tags: [GPU, CUDA, AI Infra, 硬件]
categories: [硬件]
series: [AI Infra]
featured: false
description: "从 SM、Warp、Tensor Core 到 HBM，梳理 GPU 为什么适合大规模并行计算，以及它对训练与推理性能的影响。"
draft: true
ShowToc: true
TocOpen: true
---
```
GPU
│
├── SM（执行单元）
│     │
│     ├── CUDA Core（标量计算）
│     ├── Tensor Core（矩阵计算）
│     ├── Register（最快存储）
│     └── Shared Memory（SM 内共享）
│
└── Global Memory（显存）
```
```
Kernel
│
└── Grid
      │
      └── Block
             │
             └── Warp（32 Threads）
                     │
                     └── Thread
```

## 前言

这篇文章从工程视角梳理 GPU 的核心概念：它为什么适合并行计算，执行模型是怎样的，显存层级如何影响吞吐，以及这些硬件特性为什么会直接决定训练和推理系统的性能上限。

<!--more-->

## 为什么 GPU 适合并行计算

CPU 擅长低延迟、复杂控制流和通用任务处理；GPU 则把更多晶体管预算放在计算单元和内存带宽上，换取更高的吞吐。  
如果一个任务可以拆成大量相似的小计算，并且这些计算之间依赖较少，那么 GPU 往往比 CPU 更有优势。

可以先抓住一个核心区别：

- CPU 优先优化单线程性能和分支处理能力
- GPU 优先优化大规模并行与高吞吐

这也是为什么矩阵乘、卷积、注意力、向量检索中的距离计算这类工作天然适合放到 GPU 上。

## GPU 的基本执行模型

从 CUDA 视角看，程序通常会把一个 kernel 发射到 GPU，然后由大量线程并行执行。  
这些线程不是完全独立调度的，而是以更粗粒度的组织形式运行：

- `thread`：最小执行单元
- `warp`：一组一起执行的线程，通常是 32 个
- `block`：多个线程组成的线程块
- `grid`：一次 kernel 启动的全体线程块

理解 warp 很重要，因为 GPU 的许多性能问题都和它有关。  
例如同一个 warp 内如果线程走了不同分支，就会出现 branch divergence，硬件只能串行执行不同路径，吞吐会下降。

## SM：真正执行计算的地方

SM（Streaming Multiprocessor）可以理解为 GPU 上负责调度和执行线程块的核心计算单元。  
一个 GPU 上通常有很多个 SM，kernel 启动后，不同 block 会被分配到不同的 SM 上执行。

每个 SM 内部通常包含：

- 标量/向量计算单元
- Tensor Core
- 寄存器文件
- shared memory / L1 cache
- warp 调度器

GPU 能隐藏内存访问延迟，一个关键原因就是 SM 上会同时挂起很多 warp。  
当某个 warp 因为访存而 stall 时，调度器可以快速切到另一个已经就绪的 warp 继续执行。

所以在 GPU 上，很多优化问题最终都会落到两个指标上：

- occupancy 是否足够高
- 单个 warp 是否经常因为访存或分支而空转

## 显存层级：越快越小，越慢越大

GPU 的存储层级和 CPU 类似，但更强调带宽与访问模式：

1. 寄存器：最快，容量最小
2. shared memory：线程块内共享，延迟低
3. L2 cache：全局共享
4. HBM / GDDR：容量大、带宽高，但延迟仍明显高于片上存储

很多 kernel 优化，本质上就是尽量让数据留在更靠近计算单元的位置，减少对全局显存的反复访问。  
例如 GEMM 会通过 tiling 把输入切成小块，先搬到 shared memory，再在片上反复复用。

如果访存模式不连续、不能 coalescing，或者中间结果频繁回写 HBM，那么即使算力很强，最终性能也可能被带宽限制住。

## Tensor Core 为什么重要

Tensor Core 是 GPU 为矩阵运算提供的专用单元。  
在训练和推理场景里，大量热点算子都能归结为矩阵乘，因此 Tensor Core 往往直接决定了峰值吞吐。

这也是为什么现代 AI 系统会强调：

- `FP16 / BF16`
- `INT8 / FP8`
- 张量布局和对齐
- fused kernel

因为只有把数据类型、排布和算子形式组织到硬件喜欢的样子，才能真正吃满 Tensor Core。

## GPU 性能瓶颈通常出在哪

实践里，GPU 程序不一定总是算力受限，更常见的是以下几类问题：

- 显存带宽瓶颈
- kernel launch 过碎，调度开销过高
- 数据搬运过多，例如 CPU 和 GPU 之间频繁 copy
- warp divergence 导致执行效率下降
- occupancy 不足，无法有效隐藏延迟
- 多卡通信成为整体训练或推理的瓶颈

所以看 GPU 性能时，不能只盯着 FLOPS，还要一起看：

- SM 利用率
- HBM 带宽利用率
- kernel 时间分布
- PCIe / NVLink 通信占比
- 显存占用与碎片情况

## GPU 对训练与推理系统意味着什么

对训练系统来说，GPU 决定了单步计算速度，而互联拓扑和通信效率决定了多卡扩展效率。  
对推理系统来说，除了算力，batch size、KV cache、显存容量和访存模式同样关键。

几个常见现象都能从硬件层解释：

- 小 batch 推理 often 更容易被 launch 和访存开销主导
- 大模型推理常常先撞上显存容量限制，而不是纯算力限制
- 训练里的 all-reduce 性能会直接影响多机扩展曲线
- 向量检索和 rerank 场景里，memory-bound kernel 很常见

## 学 GPU 时最值得先建立的心智模型

如果只保留最重要的几个判断框架，我会建议优先建立这几条：

1. GPU 优势来自并行和吞吐，不来自单线程速度
2. 很多性能问题本质是访存问题，不是算术问题
3. warp、SM、shared memory 是理解 kernel 行为的关键抽象
4. Tensor Core 吃不满时，理论峰值没有实际意义
5. 单卡性能和多卡性能之间，还隔着拓扑与通信这一层

## 小结

GPU 不是“更快的 CPU”，而是一种围绕大规模并行吞吐设计的计算设备。  
理解它，最重要的不是记住参数表，而是建立一套从执行模型、存储层级到系统性能的因果链条。

后续如果继续展开，可以分别写成几篇独立文章：

- CUDA 编程模型与 kernel 优化
- Tensor Core 与混合精度
- HBM、带宽墙与 memory-bound 分析
- 多卡互联：NVLink、RDMA、NCCL
