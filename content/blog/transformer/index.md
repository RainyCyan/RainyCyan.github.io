---
title: Transformer：一切大模型的起点
date: 2026-05-28T17:23:00+08:00
tags: [Transformer, LLM, DeepLearning, Attention]
series: []
featured: true
description: "从 Attention is All You Need 出发，拆解 Transformer 的结构、直觉与工程细节，理解它为什么能成为今天所有大模型的共同地基。"
draft: false
---
## 前言
> 我曾经找好朋友豪哥询问是否有transformer的入门文章，他建议我说与其看文章不如用ai辅助从single-head attention-multi-head->transformer手搓一遍，我体验下来确实感觉比看文章理解的更加深入。

我对深度学习几乎是零基础。与其按照深度学习的发展脉络从最基础的神经网络学起，不如直接把 Transformer 作为一个”锚点”，向前和向后逐步探索。这篇文章就是这个过程的记录。

首先，什么是神经网络？简单说，神经网络是一个数学工具，用一堆”加权求和 + 非线性函数”的叠加去逼近任何复杂关系。最基础的单元是神经元：

$$y = f(w_1x_1 + w_2x_2 + \cdots + b)$$

其中 $f$ 是激活函数。最简单的神经网络是单层感知机（输入层 → 输出层），进阶一点是多层感知机 MLP（输入 → 隐藏层 → 输出），可以堆很多层：输入 → 线性 → 激活 → 线性 → 激活 → ⋯ → 输出。

Transformer 是一种专门处理文本序列的神经网络结构，相比之前的 RNN 和 LSTM，它在并行性和长距离依赖建模上有本质的改进。

<!--more-->

## 背景：为什么需要 Transformer

在 Transformer 之前，处理序列的主要方式是 RNN 及其变体（LSTM、GRU）。这些结构有两个根本的问题：

1. **并行效率低**：RNN 必须按顺序处理每个 token，前一步的隐状态才能流向下一步，无法并行化。
2. **长距离依赖困难**：即使用 LSTM 来缓解，对于很长的句子，模型还是容易”记不住前面”的信息。

Transformer 用 **注意力机制**（Attention）建模所有 token 之间的关系，打破了顺序的束缚。这是它相比 RNN 最根本的进步。


## 注意力机制 Attention Mechanism

## Transformer架构
![](transformer.png)
## Self Attention


## Multi-Head Attention


## 位置编码 Positional Encoding


--- 
## Encoder & Decoder


## 训练与推理


## 演进与变体


## 小结

## 参考

- [1] https://jalammar.github.io/illustrated-transformer/

### single-head attention
