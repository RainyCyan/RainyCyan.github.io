---
title: 漫谈 Agent Memory：让模型从"健忘"走向"有记忆"
date: 2026-05-28T16:17:00+08:00
tags: [Agent, Memory, LLM]
series: []
featured: true
description: "Agent 的记忆不是一块磁盘，而是一整套围绕上下文工程的系统设计：短期、长期、情景、语义、程序性记忆是如何协作的？"
draft: true
---

## 前言
上下文有限限制了我们在一次会话完成任务的幻想，注意力丢失导致我们在一轮会话也无法很好完成任务；

LLM 本身是无状态的：每次推理都只依赖输入的 prompt，模型并不会"记得"上一次和你说过什么。
但当我们用 LLM 搭建 Agent 时，"记忆"几乎是绕不开的能力——

- 多轮对话里，agent 要记得用户的姓名、偏好、上下文话题；
- 长任务里，agent 要记得自己已经做过什么、剩下什么没做；
- 跨会话里，agent 要记得用户上周提过的项目、踩过的坑、定下的约定。

所以 Agent Memory 的本质并不是给模型加一块"硬盘"，而是一整套围绕 **context engineering** 的系统设计：
**在合适的时机，把合适的信息，以合适的形式，塞进有限的 context window 里**。

这里有两层意思，一个是存信息，也就是“记”；一个是信息检索的触发，也就是“忆”；

<!--more-->

## 为什么需要 Memory
最朴素的方案是把所有历史都拼到 prompt 里。但很快会遇到几个硬约束：

1. **Context window 有限**：哪怕是 1M token 的模型，把过去半年聊天全塞进去也是奢侈而低效的。
2. **Attention 是稀释的**：context 越长，模型对关键信息的关注度越容易被噪声稀释，出现 "lost in the middle"。
3. **成本与延迟**：token 越多，单次调用越贵越慢。
4. **跨会话需求**：浏览器关掉、进程重启、换一台机器，对话历史就丢了。

因此我们需要一套机制：能筛选、能压缩、能持久化、能按需召回。这就是 Memory 系统要做的事情。

## Memory 的几种分类
学界和工业界对 Agent Memory 有不同的划分方式，我比较喜欢按"时间维度"和"内容维度"两条线一起看。

这里根据一篇综述和我自己的理解尝试对Agent Memory分一下类：

### 按时间维度
- **Short-term Memory（短期记忆）**：本次会话/本次任务的上下文，通常直接活在 prompt 里。
- **Long-term Memory（长期记忆）**：跨会话、跨任务持久化的信息，通常存到外部存储里，按需检索。

### 按内容维度（借鉴认知心理学）
- **Episodic Memory（情景记忆）**：发生过的具体事件，比如"用户上周让我帮他订过一张去上海的机票"。
- **Semantic Memory（语义记忆）**：抽象出来的事实和知识，比如"用户是 Java 程序员，偏好 Spring 全家桶"。
- **Procedural Memory（程序性记忆）**：怎么做事的经验，比如"这个用户喜欢先看结论再看分析"，或者 agent 学到的 skills、workflows。

###
- token记忆

- latent记忆

- param记忆

不同的产品形态对各类记忆的侧重不同：
- 聊天助手类（如 ChatGPT 的 memory）更依赖 episodic + semantic；
- 编码 agent（如 Cursor、Codewiz）更依赖 procedural + semantic（项目规约、风格、技术栈）；
- 任务型 agent 更依赖 short-term 的执行状态 + long-term 的经验沉淀。

## 一个 Memory 系统通常长什么样
抛开具体实现，一个 memory 系统大致可以拆成四个阶段：

```
[输入] -> [Encode 抽取]  -> [Store 存储]  -> [Retrieve 检索]  -> [Inject 注入]
                                              ^
                                              |
                                       [Update / Forget]
```

### 1. Encode：从对话流里抽出"值得记的东西"
不是所有 token 都值得被记住。常见做法：
- **基于规则**：识别"我叫 xxx"、"我喜欢 xxx"等模式。
- **基于 LLM 抽取**：用一个小模型/同模型在后台跑 summarizer，把对话压缩成结构化的 facts。
- **基于事件**：任务完成、用户显式说"记住这个"时触发写入。

抽取阶段最容易踩的坑是 **过度抽取**：把所有无关闲聊都当成事实存起来，长期看会污染整个记忆库。

### 2. Store：怎么存
常见的三类底座：
- **Key-Value / 文档**：结构化字段，例如 `user.name`、`user.timezone`，适合 profile 类信息。
- **Vector DB**：把片段 embedding 后存起来，支持语义检索，适合海量非结构化记忆。
- **Graph**：用实体-关系来组织记忆，适合需要做多跳推理的场景（比如 GraphRAG、社交关系类应用）。

实际系统里这三者往往是混合使用的：profile 走 KV，长文本走 vector，实体关系走 graph。

### 3. Retrieve：召回
召回策略决定了 memory 用得好不好。常见的有：
- **相似度检索**：当前 query embedding 与历史片段最相近的 top-k；
- **时间衰减**：越近的记忆权重越高；
- **重要性打分**：抽取时给每条记忆打一个 importance score（参考 Generative Agents 的设计）；
- **混合检索**：rerank 多路召回结果。

> Score = α * similarity + β * recency + γ * importance

这个加权公式是 Generative Agents 论文里很经典的设计，今天大量产品仍在沿用。

### 4. Inject：把记忆塞回 prompt
召回出的记忆不是直接 concat 就完事，还要考虑：
- **位置**：放 system prompt 里，还是放在 user 消息前？前者更稳定，后者更新鲜。
- **格式**：用自然语言摘要，还是结构化字段？
- **预算**：给 memory 预留多少 token？超额怎么裁剪？

### 5. Update & Forget：会写也要会忘
这一步常被忽略，但它决定了 memory 系统会不会随时间崩塌：
- **去重**：同一事实多次出现要合并而不是重复存；
- **更新**：用户改了偏好（"我现在用 Rust 不用 Go 了"），老记忆要被覆盖或标记过期；
- **遗忘**：对低频、低重要性、过时的记忆做 decay 或直接删除。

可以类比人脑：你记得清楚的，是反复用到的和情绪强烈的，其他都在慢慢遗忘。

## 几个值得了解的工程实现
- **MemGPT / Letta**：把 LLM 当成 OS，区分 main context 和 external context，引入"分页"思想。
- **Mem0**：开源 memory 层，主打"抽取-存储-召回"的标准化 pipeline，对接 KV + Vector。
- **Generative Agents（斯坦福小镇）**：开创性地提出 reflection + importance score + retrieval 的组合。
- **ChatGPT Memory**：典型的 user-level semantic memory，强调用户可控（可以查看、删除每一条）。
- **Cursor / Codewiz Rules**：本质是 procedural memory 的工程化形态，把"怎么和这个项目相处"以规则文件的形式固化。

## 给做 Agent 的几个实践建议
1. **先不要急着上 vector DB**。很多场景一个结构化 profile + 一段滚动 summary 就够了，引入 vector 之前先想清楚为什么需要它。
2. **把 memory 当作 product feature 来设计**，而不是技术黑盒：让用户能看到、能修改、能删除记住了什么，这关乎信任。
3. **写入要克制，召回要激进**。宁可记少一点准一点，也不要塞一堆噪声；但召回时多路融合、宽召回 + rerank 通常效果更好。
4. **给 memory 加版本和时间戳**。事实是会变的，没有时间戳的记忆迟早会自相矛盾。
5. **观测先于优化**。把每次召回了什么、注入了什么、最终模型用没用都打点出来，否则调优就是玄学。

## 写在最后
Agent Memory 现在还远没有形成标准答案。
不同业务形态需要的记忆是不同的，甚至同一个产品里，不同子能力的记忆策略也不该一样。

我自己更倾向于把它看成 **context engineering 的一个子问题**：
一切都是为了在有限的 context 里，让模型"恰好"知道它该知道的事情——
多一分嫌吵，少一分嫌笨。
