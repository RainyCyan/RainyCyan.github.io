---
title: Raft：一个可以被理解的共识算法
date: 2026-05-29T00:00:00+08:00
tags: [分布式系统, Raft, 共识算法, 一致性]
series: [分布式系统]
featured: true
description: "Paxos 难以理解，Raft 以可理解性为第一目标重新设计共识算法：Leader 选举、日志复制、安全性，聊聊 Raft 是如何让分布式一致性变得清晰的。"
draft: true
ShowToc: true
TocOpen: true
---

## 前言

在分布式系统中，多个节点需要对某个状态达成一致。共识算法是解决这个问题的基石，但传统的 Paxos 算法难以理解和实现。Raft 则以**可理解性**为首要设计目标，通过强 Leader 和更清晰的状态机，让共识变得优雅而易懂。

<!--more-->

## 背景：为什么需要共识算法

- 分布式系统中的一致性问题
- 单点故障与高可用的需求
- 共识算法在数据库、分布式锁、服务发现等场景的应用

## Raft 的核心设计哲学

### 可理解性第一

与 Paxos 不同，Raft 从一开始就以清晰易懂为目标，将共识问题分解为三个独立且易于理解的子问题。

### 强 Leader 模式

区别于 Paxos 的无中心设计，Raft 采用强 Leader，简化决策流程。

### 状态机复制

所有节点应用相同的日志，在相同顺序下执行相同的命令，最终达到一致状态。

## Leader 选举

### 心跳与超时

- 心跳间隔与随机超时的设计
- 避免分裂投票的机制

### 投票过程

- 节点状态转移（Follower → Candidate → Leader）
- 投票规则与限制

### 任期（Term）概念

- 任期如何标记"逻辑时钟"
- 旧任期 Leader 的淘汰机制

## 日志复制

### 日志结构

日志条目包含：命令、任期号、索引值。

### 同步过程

- Leader 向 Follower 发送 AppendEntries RPC
- Follower 验证与应用日志的规则
- 处理日志不一致的场景

### 提交（Commit）与应用

- 什么时候日志条目被认为已提交
- Leader 与 Follower 之间的 commitIndex 同步

## 安全性保证

### Election Safety

一个任期内最多一个 Leader，通过投票过程保证。

### Log Matching Property

如果两个日志在同一索引处的任期号相同，则该索引之前的所有日志完全相同。

### State Machine Safety

如果某日志条目在某节点已应用，则其他节点在同一索引不会应用不同的条目。

## 集群成员变更

### 两阶段方案

为什么不能直接切换新旧配置，引入中间的联合共识状态。

### 配置变更流程

1. Leader 接收新配置
2. 在新旧配置的并集上达成共识
3. 切换到新配置

## 与 Paxos 的对比

| 特性 | Raft | Paxos |
|------|------|-------|
| 设计目标 | 可理解性 | 容错性、安全性 |
| Leader | 强 Leader | 无中心（最终一致） |
| 任期/轮次 | Term | Ballot/轮次 |
| 复杂度 | 相对简洁 | 相对复杂 |

## 工程实践

### 常见实现库

- etcd（Go）
- consul（Go）
- TiKV（Rust）

### 实现要点

- 快照机制优化日志存储
- 预投票（Pre-vote）避免不必要的选举
- 批处理提高吞吐量

## 小结

Raft 的成功在于设计权衡：通过强 Leader 的假设与更严格的约束，换取清晰的理解与相对简洁的实现。这使得 Raft 成为现代分布式系统的首选共识算法。

## 参考资源

- [Raft 论文](https://raft.github.io/raft.pdf)
- [Raft 可视化演示](https://raft.github.io/raftscope/index.html)
- [Raft 官网](https://raft.github.io/)
