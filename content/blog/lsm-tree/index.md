---
title: LSM Tree
date: 2026-06-04T10:00:00+08:00
tags: [data-structures, indexing, storage, lsm-tree]
featured: false
description: "Understand Log-Structured Merge-tree design and its applications in high-performance databases"
draft: true
ShowToc: true
TocOpen: true
---
从最基础的put(),get()操作开始，我们想要一个查询快，写入快的结构。

最基础的是一个文件，每行存储一个kv，写入时直接追加到文件末尾，这样写入是最快的，但是查询需要遍历，遍历一次带来的信息量只有exist/not exist,也就是说一次遍历只能排除一个kv,我们希望查询时每次都能排除一块区域，这就要求数据必须保证一定的性质——有序性。

next, 为了维护这样一种有序的性质，我们写入的代价太大了，对于写密集场景根本无法接受。能不能放宽全局有序这个约束，保证局部有序——有多个分段，每个分段有序，也就是新增了一个原则——分段采用append-only策略，这也引入了一个问题，append-only不能删除已有kv,必须维护多个版本的kv。

however,为了维护这么多版本的kv,我们实际存储了很多无用的信息——之前版本的kv明明不会再用了，但是因为之前的写入分段后就不再修改的原则，我们没办法删除——因为删除也是一种修改。放宽这个限制，能不能一段时间后执行一个垃圾回收的机制，清理掉这些旧版本信息。

next,有了分段，是一个kv就对应一个分段吗——文件有固定的meta以及固定的成本cost,我们需要摊薄这一部分成本。也就是说我们需要“攒”一个batch再写入最终的分段。但是在“攒”这个工程中，需要涉及到不断的修改，最适合这个“修改”状态的当然是RAM。

at this time,断电了，存在RAM的状态就完全丢失了。如何能保证断电后可以恢复呢？可以存一个最小的信息——动作，这样断电后我们可以根据动作来恢复状态。And,很明显这个写动作到磁盘需要放在写RAM之前

还记得之前的compaction吗，如果我们不断的后台之行合并，很快就会发现磁盘IO被打满而CPU空闲，这是因为我们最naive的策略为了消除重复版本反而引入了新的代价——没有重复版本的数据也需要写文件，这就是写放大。
问题的根因在于我们支付了多余的代价，有一些没有交叉的分段我们还是执行了合并。只有存在"冲突（Conflict）"的地方，才值得支付 Compaction 的成本。

能不能新引入一个设计——让分段没有冲突，很明显memtable->flush无法满足这个约束，因为put是按时间顺序来的，那么能不能让compaction后满足这个约束，这个很明显可以。但又来一个问题，如果一个新的memtable flush，为了保证这个约束，很可能要修改很多个文件，这代价太大了。回忆一下，放宽一下约束，构造一个缓冲行不行，上面层可以允许冲突。

1. 所有数据都要全局有序吗？ → 不，局部有序即可。
2. 每次写都要维护最终结构吗？ → 不，先 Batch。
3. 最终结构才能持久化吗？ → 不，记录事件即可（WAL）。
4. 所有不变量都要立刻维护吗？ → 不，L0 延迟维护。
5. 所有数据都值得同样维护吗？ → 不，数据有冷热、有生命周期，应区别对待。

从第一性原理推导 LSM Tree
Goal

我们的目标是设计一个支持 put(key, value) 和 get(key) 的存储结构，希望同时满足两个目标：

查询快（Fast Read）
写入快（Fast Write）

然而，这两个目标天然存在冲突。LSM Tree 的发展过程，本质上就是不断放宽约束，在查询性能和写入性能之间寻找新的平衡。

第一阶段：为什么需要有序？

最朴素的实现方式是把所有 Key-Value 顺序追加到一个文件中：

apple -> 1
dog   -> 3
cat   -> 2
...

每次 put() 都只需要 Append 到文件末尾，因此写入效率极高。

但查询却非常低效。当执行 get(cat) 时，只能从文件头开始逐条扫描：

apple ❌
dog   ❌
cat   ✅

每访问一条记录，只知道它不是目标，却无法利用这条信息缩小后续的搜索范围，因此最坏情况下仍然需要扫描整个文件。

查询真正需要的，不是更多的数据，而是能够快速缩小搜索空间（Search Space Reduction）的结构。

最简单的方法就是让数据保持有序。有序之后，每次比较都能够排除一大片数据，从而把查询复杂度降低到对数级别。

设计原则一：查询想要快，就必须能够快速缩小搜索空间，因此数据需要具有一定的有序性。

第二阶段：为什么放弃全局有序？

虽然有序提高了查询效率，但维护全局有序的代价非常高。

例如当前数据为：

apple
cat
dog

插入一个新的 bird 后，为了保持顺序，需要把后面的所有数据整体后移：

apple
bird
cat
dog

对于写密集场景，这种维护成本几乎无法接受。

于是我们开始思考：真正必须保持的是"全局有序"吗？

答案是否定的。查询真正依赖的是局部范围能够快速搜索，而不是整个数据库始终保持一个有序文件。因此，我们可以放宽约束：已有数据保持不可修改（Immutable），新写入的数据形成新的有序段（Run）。

数据库逐渐演变成多个局部有序的 Run：

Run1 (Sorted)
Run2 (Sorted)
Run3 (Sorted)

每个 Run 内部保持有序，而 Run 之间互相独立，新数据永远只生成新的 Run。

设计原则二：维护全局有序过于昂贵，可以放宽为多个局部有序的 Immutable Run。

第三阶段：为什么需要 Compaction？

Immutable Run 带来了新的问题。

假设同一个 Key 被不断更新：

cat = 1
cat = 2
cat = 3

由于旧 Run 永远不会修改，磁盘实际上保存了三个版本，而真正有效的只有最后一个。

旧版本已经失效，却无法直接删除，因为删除本身也是一种修改。

因此，需要后台周期性地执行垃圾回收（Garbage Collection）：将多个 Run 合并，只保留最新版本，并删除已经失效的数据。这就是 Compaction。

设计原则三：Immutable 数据天然会产生垃圾，因此需要后台 Compaction 回收失效版本。

第四阶段：为什么需要 MemTable？

继续观察，我们会发现另一个问题。

如果每次 put() 都立即生成一个新的 Run，那么每个 Run 都需要维护 Header、Footer、索引、元数据等固定开销，文件系统本身也需要维护 inode、打开关闭文件等成本。

这些固定成本远远超过了一条 Key-Value 本身。

因此，更合理的方法是先积累一批数据（Batch），等达到一定规模后，再一次性生成一个新的 Run，将固定成本均摊到大量数据上。

但是 Batch 在不断积累的过程中，需要频繁插入、维护有序结构，它属于一个不断变化的 Mutable State，因此最适合放在修改成本最低的介质——内存。

于是，MemTable 出现了。

设计原则四：最终结构无需立即维护，Mutable State 可以先在内存中积累，再批量生成 Immutable Run。

第五阶段：为什么需要 WAL？

MemTable 全部位于内存中，因此断电会导致所有未 Flush 的数据全部丢失。

那么，为了恢复 MemTable，是否需要把整个内存状态都保存下来？

其实并不需要。

真正需要保存的，只是能够重新构建状态的最小事件（Event），例如：

put(cat, 100)

系统重启后，只要顺序 Replay 所有 Event，就能够重新恢复 MemTable。

因此，我们不需要保存整个状态，而只需要记录导致状态变化的事件，这就是 Write-Ahead Log（WAL）。

同时，日志必须先于 MemTable 写入：

Append WAL
      ↓
Update MemTable

如果顺序反过来，在更新 MemTable 后、写 WAL 前发生崩溃，数据库将永远失去恢复这条数据的机会。

设计原则五：最终状态无需立即持久化，只需先记录能够恢复状态的最小事件。

第六阶段：为什么会产生 Write Amplification？

Compaction 清除了旧版本，但同时又带来了新的问题。

例如：

SST1
apple
cat
dog

SST2
bird
cat
fish

真正需要处理的只有 cat，但由于 SST 是 Immutable，Compaction 无法修改已有文件，只能重新生成整个新的 SST。

因此，即使 apple、bird、dog、fish 完全没有发生变化，也必须跟着一起重新写入磁盘。

Write Amplification 的根本原因并不是 Compaction，而是 Immutable SST 无法原地修改。

设计原则六：Immutable 简化了写路径，却不可避免地引入后台重写（Write Amplification）。

第七阶段：为什么需要 L0？

进一步分析 Compaction，我们会发现：只有存在**潜在版本冲突（Potential Version Conflict）**的数据才值得 Merge。

如果两个 SST 的 Key Range 完全没有重叠：

[a, f]
[g, m]

那么它们根本不存在重复版本，也没有 Merge 的必要。

于是，一个新的目标出现了：每一层内部的 Key Range 不重叠。

这样，查询时每一层最多只需要访问一个 SST。

然而，新的 MemTable Flush 是按照时间产生的，而不是按照 Key Range 组织的，因此一个新的 SST 很可能同时覆盖多个已有 SST。如果直接放入 L1，就需要重写大量文件来维持"Key Range 不重叠"这一约束。

维护成本再次变得不可接受。

因此，我们再次放宽约束，引入一个缓冲层——L0。

L0 允许 SST 之间发生重叠，等积累到一定规模后，再统一执行 Compaction，把数据整理进入 L1。

L0 本质上并不是一个存储层，而是一个延迟维护全局不变量（Deferred Maintenance）的缓冲区。

设计原则七：昂贵的不变量无需立即维护，可以通过缓冲层延迟维护。

第八阶段：为什么不能所有数据采用同一种策略？

真实业务的数据更新分布极不均匀。

例如：

用户 Session、点赞数可能每天更新数万次；
国家名称、商品信息可能几年都不会变化。

但传统 LSM 会把它们统一放入 Compaction 中，不断重复搬运。

于是现代 LSM 系统开始意识到：不是所有数据都应该采用相同的维护策略。

不同生命周期、不同更新频率、不同 Value 大小的数据，可以采用不同的 Compaction 策略，这也是后续 RocksDB、WiscKey、Universal Compaction 等工作的出发点。

设计原则八：不同生命周期、不同更新频率的数据，应采用不同维护策略。

总结：LSM Tree 的八条设计哲学

回顾整个推导过程，我们会发现，LSM Tree 的演化并不是不断增加新的组件，而是在不断放宽那些维护成本过高的约束。

查询需要快速缩小搜索空间，因此数据需要保持一定的有序性。
全局有序过于昂贵，可以放宽为多个局部有序的 Immutable Run。
Immutable 会产生垃圾，因此需要后台 Compaction 清理失效版本。
固定成本应该批量摊薄，因此引入 MemTable 和 Batch 写入。
状态无需立即持久化，只需记录能够恢复状态的事件（WAL）。
Immutable 带来了后台重写，因此不可避免地产生 Write Amplification。
昂贵的不变量无需立即维护，可以通过 L0 延迟维护。
数据并不均匀，应根据生命周期和更新模式采用不同策略。
## Introduction

## LSM Tree Architecture

## Compaction Strategies

## Trade-offs and Optimization

## References
