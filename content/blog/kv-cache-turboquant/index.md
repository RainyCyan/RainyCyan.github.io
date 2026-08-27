---
title: KV Cache 量化：TurboQuant 如何在低比特下保留模型需要的几何结构
date: 2026-08-05T15:00:00+08:00
lastmod: 2026-08-27T15:00:00+08:00
tags: [LLM, Inference, KVCache, Quantization]
series: []
featured: false
description: TurboQuant 通过随机旋转、最优标量量化和 QJL 残差补偿，在无需数据校准的前提下压缩 KV Cache，并兼顾重建误差与内积估计偏差。
draft: true
ShowToc: true
TocOpen: true
---

大语言模型的上下文越长，推理时需要保留的 KV Cache 就越大。它保存了历史 token 在每一层注意力计算中的 key 和 value 表示，规模同时受层数、注意力头数、隐藏维度和上下文长度影响。对长上下文服务而言，KV Cache 往往先于模型权重成为显存和带宽的限制因素。

于是问题看起来很直接：把 KV Cache 从 FP16 或 BF16 压到更低的 bit-width。但真正困难的地方不在于把浮点数改写成整数，而在于确定哪些信息不能丢。注意力计算和向量检索都依赖内积；如果量化误差让相似度出现系统性偏移，即使逐坐标的均方误差不大，最终的注意力权重或近邻排序仍然可能改变。

TurboQuant 解决的正是这个矛盾：它不依赖具体数据集训练码本，能够在线处理新到达的向量；同时分别针对重建误差和内积估计设计量化器，并尽量让计算过程适合 GPU。论文由 Google Research、Google DeepMind 和 NYU 的研究者完成，论文链接见文末。

## 先把问题说清楚：KV Cache 需要什么样的量化

压缩大致有两条路线。无损压缩利用数据中的重复模式或统计规律编码，例如 LZ4、Huffman、ANS 和 Zstandard；它不会改变数值，但压缩率受数据熵限制。低比特量化则直接减少每个数值的表示位数，压缩空间更大，却必须承受有损重建。

KV Cache 的在线场景又增加了约束。一种可用的量化方法至少要满足三点：

1. **数据无关（data-oblivious）**：量化规则不依赖预先收集的数据集或运行时校准。每个新 token 到达时，都能独立完成量化。
2. **低失真**：不仅要让重建向量与原向量的 MSE 足够小，还要尽量保留内积、距离等几何关系。
3. **硬件友好**：量化和解量化应能在 GPU 上并行执行，避免为了压缩把数据搬到 CPU，导致计算收益被传输开销抵消。

现有方法通常只能在这些目标之间取舍：依赖离线训练的 Product Quantization（PQ）可以适应特定数据，但不适合动态到达的 KV；简单的逐元素截断容易实现，却未必能控制内积误差；一些 token 剪枝方法减少了需要保存的 token 数量，但改变的是缓存内容，而不是对每个向量进行保真压缩。

TurboQuant 的关键选择是：先把“任意形状的高维向量”变成“统计性质可控的坐标”，再做逐坐标量化。

## 第一步：随机旋转，把高维问题拆成标量问题

设输入向量为 $\mathbf{x}\in\mathbb{R}^d$。在分析中，先考虑单位向量 $\|\mathbf{x}\|_2=1$；一般向量只需要额外保存其 L2 范数，解量化后再缩放。TurboQuant 先使用一个随机正交矩阵 $\mathbf{\Pi}$ 计算：

$$
\mathbf{z}=\mathbf{\Pi}\mathbf{x}.
$$

正交旋转不会改变向量的长度，却会把原本可能集中在少数坐标中的信息分散到所有维度。对固定输入而言，随机矩阵带来的随机性使 $\mathbf{z}$ 在单位球面上均匀取向；因此，每个坐标的边缘分布都已知，服从一个 Beta 型分布。在维度较高时，它进一步接近 $\mathcal{N}(0,1/d)$。不同坐标近似独立，这正是算法能够逐坐标处理的原因。

这一步不是普通的“打乱维度”。它把量化器面对的最坏输入，转换成了一个可以预先分析的标量分布。于是，原本需要在 $d$ 维空间中寻找量化码字的问题，可以近似拆成 $d$ 个相同的一维问题。

## 第二步：为已知分布求最优码本

如果每个坐标平均使用 $b$ bit，就有 $2^b$ 个量化中心。TurboQuant 不在运行时从业务数据中训练这些中心，而是根据旋转后坐标的 Beta 分布，提前求解连续的一维 k-means 问题。这个问题也就是 Lloyd–Max 标量量化：把数轴划分为若干区间，每个区间用一个中心值重建，使期望平方误差最小。

在线量化时，流程很简单：

1. 计算 $\mathbf{z}=\mathbf{\Pi}\mathbf{x}$；
2. 对每个坐标 $z_j$，查找距离最近的码本中心，并保存其索引；
3. 解量化时根据索引恢复近似的 $\tilde{\mathbf{z}}$；
4. 计算 $\tilde{\mathbf{x}}=\mathbf{\Pi}^{\mathsf T}\tilde{\mathbf{z}}$。

这就是 **MSE Optimized TurboQuant**。论文给出的失真上界为：

$$
D_{\mathrm{mse}}\leq \frac{\sqrt{3}\pi}{2}\cdot 4^{-b}.
$$

对于单位向量，$b=1,2,3,4$ 时，论文给出的代表性 MSE 约为 $0.36$、$0.117$、$0.03$ 和 $0.009$。更重要的是，这个结果接近 Shannon 失真—码率下界 $4^{-b}$，两者最多只差约 $\sqrt{3}\pi/2\approx2.7$ 的常数因子。换句话说，在给定 bit-width 下，继续改进的空间已经受到信息论限制，而不是简单换一个码本就能消除。

但这里出现了一个容易被忽略的问题：MSE 小，不等于内积估计无偏。

## MSE 最优为什么还不够：内积会产生系统性偏差

考虑极低比特的情况。高维下，1-bit MSE 量化的码本近似为：

$$
\left\{\pm\sqrt{\frac{2}{\pi d}}\right\}.
$$

因此，量化后的向量近似是对旋转坐标取符号，再乘上固定幅度。对任意查询向量 $\mathbf{y}$，其期望内积不是原始内积，而近似为：

$$
\mathbb{E}\left[\langle\mathbf{y},\tilde{\mathbf{x}}\rangle\right]\approx\frac{2}{\pi}\langle\mathbf{y},\mathbf{x}\rangle.
$$

这是一种乘性偏差：相似度整体被压低。随着 bit-width 增加，偏差会减小，但在低 bit 场景下仍然可能改变排序。对于 KV Cache，这意味着某些注意力分数会被系统性地缩放；对于最近邻搜索，则可能把本应排在前面的向量推到后面。

因此，如果业务只关心重建向量本身，MSE 版本已经是合适的目标；如果业务直接依赖量化向量计算内积，就需要第二个版本。

## 第三步：用 1 bit QJL 补回残差

TurboQuant 的内积版本记作 **Inner Product TurboQuant**，总 bit 预算为 $b$ 时，它把预算拆成两部分。第一阶段使用 $b-1$ bit 的 MSE 量化器，得到 $\tilde{\mathbf{x}}_{\mathrm{mse}}$，并计算残差：

$$
\mathbf{r}=\mathbf{x}-\tilde{\mathbf{x}}_{\mathrm{mse}}.
$$

经过第一阶段后，残差的 L2 范数已经较小。第二阶段不再逐坐标保存残差，而是使用 1-bit 的 Quantized Johnson–Lindenstrauss（QJL）变换。令 $\mathbf{S}$ 为高斯随机投影矩阵，保存：

$$
\mathbf{q}=\operatorname{sign}(\mathbf{S}\mathbf{r})\in\{-1,+1\}^d,qquad \gamma=\|\mathbf{r}\|_2.
$$

量化结果因此由三部分组成：第一阶段的码字索引、残差的符号向量 $\mathbf{q}$，以及残差范数 $\gamma$。解量化时，用同一个随机投影矩阵恢复残差近似值，再与 MSE 重建结果相加。

QJL 的作用不是把残差精确还原成原向量，而是让任意查询向量 $\mathbf{y}$ 的残差内积满足无偏性：

$$
\mathbb{E}\left[\langle\mathbf{y},\tilde{\mathbf{r}}\rangle\right]=\langle\mathbf{y},\mathbf{r}\rangle.
$$

于是：

$$
\mathbb{E}\left[\langle\mathbf{y},\tilde{\mathbf{x}}_{\mathrm{mse}}+\tilde{\mathbf{r}}\rangle\right]=\langle\mathbf{y},\mathbf{x}\rangle.
$$

这里的“无偏”不是说每一次内积计算都没有误差，而是说随机误差长期平均后不会固定地偏大或偏小。QJL 用一个 bit 保存每个投影结果的符号，再利用随机投影的统计性质估计残差内积；残差越小，补偿项的方差也越小。

论文给出的内积失真上界为：

$$
D_{\mathrm{prod}}\leq \frac{\sqrt{3}\pi^2\|\mathbf{y}\|_2^2}{d}\cdot4^{-b}.
$$

这个结构解释了 TurboQuant 的两种模式：MSE 版本优先保留向量本身的信息，适用于量化结果还要被后续模块继续处理的场景；Inner Product 版本用额外的 1 bit 交换内积估计的无偏性，适用于相似度计算和排序敏感的场景。

## 理论结果是否能落到任务指标上

论文实验先在 1536 维的 DBpedia Entities embedding 上验证失真性质。结果与理论分析一致：两种版本的误差方差都会随 bit-width 增加而下降，但低 bit 时 MSE 版本的内积偏差明显，并且相似度越大，系统性偏差越明显；Inner Product 版本在不同 bit-width 下都保持无偏。

在 Llama-3.1-8B-Instruct 的 Needle-in-a-Haystack 测试中，上下文长度从 4k 扩展到 104k token，并将所有方法的 KV Cache 都压缩到原始内存的 25%。召回分数如下：

| 方法 | 召回分数 |
| --- | ---: |
| SnapKV | 0.858 |
| PyramidKV | 0.895 |
| KIVI | 0.981 |
| PolarQuant | 0.995 |
| Full Precision | 0.997 |
| TurboQuant | 0.997 |

在这个设置下，TurboQuant 达到与全精度基线相同的分数。这个结果的意义不只是“压缩率高”：它说明对长上下文检索而言，保留每个向量的几何关系，可能比直接删除部分 token 更稳妥。

LongBench 进一步覆盖单文档问答、多文档问答、摘要、few-shot、合成任务和代码补全等生成任务。论文测试了 2.5 bit 和 3.5 bit 两种非整数位宽。其做法是对少量离群通道使用更高位宽，对普通通道使用更低位宽，再取平均值。例如 128 个通道中 32 个使用 3 bit、96 个使用 2 bit，平均位宽就是：

$$
\frac{32\times3+96\times2}{128}=2.5.
$$

需要注意，离群通道的选择可能需要感知上层模型或业务分布。这一策略虽然提高了极限压缩下的质量，但它不再是完全不依赖数据的纯量化流程。

TurboQuant 也被用于高维最近邻搜索。论文在 200、1536 和 3072 维数据上比较 PQ、RabitQ 和 TurboQuant。4-bit 量化时间如下：

| 方法 | 200 维 | 1536 维 | 3072 维 |
| --- | ---: | ---: | ---: |
| Product Quantization | 37.04 s | 239.75 s | 494.42 s |
| RabitQ | 597.25 s | 2267.59 s | 3957.19 s |
| TurboQuant | 0.0007 s | 0.0013 s | 0.0021 s |

PQ 的主要成本来自数据集相关的 k-means 码本训练，RabitQ 在实验实现中也没有充分利用向量化和 GPU。TurboQuant 只需固定维度、位宽和随机矩阵即可生成全局码本，因此量化阶段几乎不需要索引构建预处理。论文报告其在各维度上的 top-k 召回率也持续优于对比方法。

## 对 GravityDB 的启发：能用，但边界必须先划清

如果 GravityDB 面向的业务能够固定向量维度和数据类型，例如所有输入都是 $d$ 维 BF16 向量，那么存储层可以在写入路径上解析向量并执行在线量化。维度与 bit-width 固定后，码本和随机旋转矩阵可以作为共享配置，单个向量主要需要保存量化索引；Inner Product 版本还需要保存 QJL 符号和残差范数。

落地时至少需要验证以下问题：

- **归一化**：论文的主要分析以单位向量为前提。非单位向量需要保存 L2 范数，或者在业务侧保证输入已经归一化。
- **离群值**：如果少数通道的数值范围显著更大，统一位宽可能让这些通道承担过大的误差。2.5/3.5 bit 的离群通道策略可以缓解问题，但会引入业务感知。
- **解量化位置**：如果读取后必须把完整向量恢复到 CPU，再交给 GPU 计算，传输开销可能抵消量化节省的空间和带宽。更理想的路径是让解量化和后续算子在同一 GPU 侧完成。
- **模式选择**：如果 GravityDB 只是存储层，无法知道上层如何使用向量，优先考虑 MSE 版本；如果读取结果会直接用于内积、相似度或召回排序，则应评估 Inner Product 版本。
- **随机矩阵管理**：量化与解量化必须使用一致的矩阵和码本版本。它们应作为带版本的系统配置管理，而不能依赖进程内临时生成的状态。

因此，TurboQuant 更适合作为“固定规格向量的在线压缩格式”，而不是对所有业务数据自动启用的通用开关。上线前应以真实 KV 分布分别测试 MSE、注意力输出和端到端任务指标，并测量量化、解量化及跨设备传输的总成本。

## 结语

TurboQuant 的价值不在于提出了又一种低比特编码，而在于它把三个通常相互牵制的要求串成了一个完整方案：随机旋转消除输入形状带来的不确定性，Beta 分布和 Lloyd–Max 码本把高维量化转化为可并行的标量量化，QJL 则针对 MSE 版本无法保证内积无偏的问题补上残差。

这条链路也给系统设计提供了一个清晰的判断标准：先明确业务真正要保留的是向量重建，还是内积与排序；再选择对应量化器，而不是只看名义上的压缩倍数。对于长上下文推理和高维检索，这种按下游目标定义失真的方式，比单纯追求更低 bit-width 更可靠。

参考资料：

- 论文：[TurboQuant: Arbitrary-Precision Post-Training Quantization for Vector Databases and Large Language Models](https://arxiv.org/pdf/2504.19874)
- PyTorch 实现：[tonbistudio/turboquant-pytorch](https://github.com/tonbistudio/turboquant-pytorch)
- vLLM 社区实现：[vLLM Pull Request #38280](https://github.com/vllm-project/vllm/pull/38280)
