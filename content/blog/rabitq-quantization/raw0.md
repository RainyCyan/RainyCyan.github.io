- 论文题目：《RaBitQ: Quantizing High-Dimensional Vectors with a Theoretical Error Bound for Approximate Nearest Neighbor Search》
- 论文链接：https://dl.acm.org/doi/10.1145/3654970
- 论文代码：https://github.com/gaoj0017/RaBitQ
- 其他：公式推导可以见https://github.com/VectorDB-NTU/RaBitQ-Library/tree/main/docs
摘要
问题背景和动机
在高维空间里进行近似最近邻（ANN）搜索，计算原始向量与查询向量的距离是性能瓶颈，忘记从哪看的，说距离计算和比较占了ANNs时间的80%-90%；
虽然已经有了比较经典的量化方法比如虽然 PQ/变种
these methods do not have a theoretical error bound and are observed to fail disastrously on some real-world datasets.
这些方法能压缩和加速，但缺少理论误差边界（error bound），在某些真实数据上会出现很差的结果甚至灾难性的失败
符号表

设计思路
高维向量测度集中现象的观察：（from author's blog）

对于归一化向量，随着数据维度的增加，x[i] $i\in \{0,...d-1\}$,的分布呈现出集中在0的周围的趋势，这个信息可以用于量化
这个比较好理解：随着维度增加，“反常点”（大于平均值）会导致其他维度取值范围急剧压缩，因此随着维度增加，这种“反常点”出现的概率也会随之下降
JL定理证明了高维向量中每个唯独的取值不太可能超过$\frac{2}{\sqrt{D}}$
索引阶段
归一化
通过归一化削弱向量长度对于距离计算的影响,距离计算推导：$o_r和q_r是原始数据向量和查询向量，c是数据向量中心，c=\frac{\sum_{i=0}^{n}{o_i}}{n},o=\frac{o_r-c}{|o_r-c|},q=\frac{q_r-c}{|q_r-c|}$,
$||q_r-o_r||^2 = ||q_r-c||^2+||o_r-c||^2-2|o_r-c||q_r-c|<q,o>$
$o_r-c可以预先计算，q_r-c只需要计算一次，那么问题的关键就转化成了计算<q,o>$
为什么需要归一化
直观地说，归一化可以将一组向量置于空间中心，并进一步将每个向量对齐到单位球面上。
- 引入中心点是为了在一定程度上消除数据集的倾斜，让向量可以更加均匀的分布在超球面上
码本构建
鉴于原始数据向量已转换为均匀分布在单位球面上的单位向量，直观上，我们也应该构造一个均匀分布在单位球面上的码本。这里我们自然地构造一个嵌套在单位球面上的超立方体，如下图所示,即$C:=\{+\frac{1}{\sqrt{D}},-\frac{1}{\sqrt{D}}\}^D,$这个codebook的表达空间是$2^D$。但是可能会导致两个并不相似的向量被量化到同一个点。

请记住，我们的目标是利用随机性带来的“自由信息”。这里，我们通过随机旋转码本，向其中注入一些随机性，随机旋转的操作避免了原始码本对于某些方向的偏好导致的不公平

只需要存储旋转的正交矩阵P
$C_{rand}=\{Px|x\in C\}$
对于数据向量o,可以找到码本中最近的向量o',又因为o'是超立方体的顶点，可以用二值向量表示
为什么需要旋转
原始码字类似$(\pm{\frac{1}{\sqrt{D}}},...,\pm{\frac{1}{\sqrt{D}}},)$,非常规整，方向固定，这种固定可能导致“不公平”现象的出现，对某些靠近码字方向向量存在“偏好”，而对远离的向量不公平
- 随机旋转 在统计意义上 消除了对特定方向的系统性偏好。 
- 但在某一个具体随机旋转矩阵下，个别方向仍然可能存在偏向，只是这种偏向 不再系统性、不再固定在某些固定方向上。
- 多次不同随机 PPP 会使这种偏差“平均掉”，也就是我们在数学理论中所说的“无偏性”
旋转器 官方库提供了两种实现
- 默认情况下，该库使用该FFHT + Kac’s Walk方法,这种方法以更低的空间和时间复杂度实现了近似随机正交变换
- 随机正交变换，随机采样后QR分解，时间和空间复杂度都很高
量化过程（寻找距离最近的码本向量）
设$P\overline{x}是o对应的量化向量，等效于x=argmin ||o-Px||^2 = argmin (||o||^2+||Px||^2-2<o,Px>) = argmin(2-2<o,Px>) =argmax <o,Px>$
这样还是不好计算，我们可以把o逆旋转后再计算与x的内积，即
$<o,Px>=<P^{-1}o,x>,\because x=\{\pm\frac{1}{\sqrt{D}}\}^D,所以只需要取和P^{-1}o同号的x即可$，最后存储二值向量 $\overline{x}_b=\{0,1\}^D$,
$\overline{x}=(2\overline{x}_b-1)/\sqrt{D}$,记号：$量化向量\overline{o}:=P\overline{x}$
查询阶段（无偏距离估计）

<$o,q$>与<$\overline{o},q$>
$构造e_1=\frac{q-<q,o>o}{|q-<q,o>o|},那么有$
- o与e1是单位向量
- o与e1正交
- {o,e1} 构成了包含 q 的二维子空间的正交基
那么就可以q用正交基表示：$q=<o,q>o+\sqrt{1-<o,q>^2}e_1$
$\therefore <\bar{o},q>=<\bar{o},<o,q>o+\sqrt{1-<o,q>^2}e_1>=<\bar{o},o><o,q>+\sqrt{1-<o,q>^2}<\bar{o},e_1>$
作者在这里生成了大量随机向量进行投影，发现\bar{o}在平面上的水平投影高度集中在o方向上，且集中在0.8附近，垂直于o方向的投影趋近于0，通过理论分析和实验得出，$E(<\bar{o},o>)\approx 0.8,E(<\bar{o},e_1>)=0$
这里用到JL定理去证明，我们直观上可以这样去理解：
无偏估计

构造无偏估计器
$\frac{<\bar{o},q>}{<\bar{o},o>}=<o,q>+\sqrt{1-<o,q>^2}\frac{<\bar{o},e_1>}{<\bar{o},o>}$,我们可以把末尾项看作误差项，其期望是0，而且在0附近集中聚集
误差界限

$P\{|\frac{<\bar{o},q>}{<\bar{o},o>}-<o,q> |\gt\}$

我们可以通过设置\eps控制误差边界，计算出的error_bound可以用于重排

高效计算估计量（标量量化）

标量量化
即使我们精确计算出了$<\bar{o},q>,<o,q>仍然是估计值，因此只需要保证量化<\bar{o},q>的误差不超过之前估计导致的误差即可$
随机标量量化

计算<$\bar{x},\bar{q}$>

bitwise 和 batch 
single

转化为二值向量计算

batch mode

RaBitQ for vector search
重排 Reranking
因为RaBitQ有误差上下界保证，重排可以实现剪枝
维护一个容量K的最大堆，对于一个候选A
1. A 的 lower bound > KNNs 中最大的 upper bound，直接丢弃
2. A 的 lower bound ≤ maxUB(KNNs)，且 KNNs 的堆顶已经 rerank 过，rerank A
3. A 的 lower bound ≤ maxUB(KNNs)，但 KNNs 的堆顶还没 rerank,先rerank 堆顶，重新判断
IVF_RaBitQ
索引构建:
数据布局:
查询:
Industry
- Milvus - IVF + RaBitQ (C++)
- Faiss - IVF + RaBitQ (C++)
- VSAG - HGraph + RaBitQ (C++)
- VectorChord - IVF + RaBitQ (Rust)
- Volcengine OpenSearch - DiskANN + RaBitQ
- CockroachDB - CSPANN + RaBitQ (Golang)
- ElasticSearch - HNSW + RaBitQ (Java - the algorithm is adopted with some minor modifications and renamed as "BBQ")
- Lucene - HNSW + RaBitQ (Java - the algorithm is adopted with some minor modifications and renamed as "BBQ")
- Oceanbase https://open.oceanbase.com/blog/21124600064/ https://zhuanlan.zhihu.com/p/1932795299459503929
实验
metrics:
量化指标：
ANN指标：
https://rte.weiyun.baidu.com/wiki/attach/image/api/imageDownloadAddress?attachId=2b1d4fcc6ace46b1877038b6a0e01d27&docGuid=wFs9QRfjxnxyvz
https://rte.weiyun.baidu.com/wiki/attach/image/api/imageDownloadAddress?attachId=c4f3e75c7ac04260882b44ca317c4abf&docGuid=wFs9QRfjxnxyvz
数学知识
JL引理：
一些参考文章或博客
入门介绍：https://zhuanlan.zhihu.com/p/7193968541
https://vectordb-ntu.github.io/RaBitQ-Library/
作者blog:https://dev.to/gaoj0017/quantization-in-the-counterintuitive-high-dimensional-space-4feg
elastic:https://bit.xiayf.cn/posts/rabitq-explainer-101.html(译文)/https://www.elastic.co/search-labs/blog/rabitq-explainer-101（原文）
JL引理：https://www.spaces.ac.cn/archives/8679/comment-page-1

