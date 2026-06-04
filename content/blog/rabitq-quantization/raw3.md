1. 索引构建阶段
1.1 Base 向量归一化
- 动机：
 原始数据向量可能分布在无限的欧式空间中，距离值范围很大，难以直接分析误差。为了让距离的计算和误差分析具有理论保证，RaBitQ 先将所有数据向量归一化到单位球面上。
- 过程：
 设原始向量为 $$o_r$$，首先计算其与一个全局中心 $$c$$（通常是用所有数据的 centroid 或者每个聚类的 centroid）之间的偏差，然后归一化：
$$o = \frac{o_r - c}{\|o_r - c\|}$$
其中$$\|o_r - c\|$$为 L2 范数。
  同样，对查询向量 $$q_r$$ 也做同样的归一化处理，得到 $$q$$。归一化后的向量均在 D 维单位超球面上分布，从而大大减少了数据的 skewness，并为后续距离估计提供了统一尺度。
在归一化操作中直接对原始向量进行 L2 归一化，只会将每个向量拉伸或压缩到单位长度，但不会消除数据集中存在的均值偏移或全局偏差。减去全局中心的主要作用有：
1. 消除全局偏移（Centering）
 对所有向量都减去一个共同的全局中心 c（例如所有数据的 centroid 或者某个聚类的 centroid），可以使得数据在空间中围绕原点分布，而不是呈现某种整体偏移。这样做后，数据分布更加均匀，有助于减少数据的 skewness（偏斜性）。
2. 提高距离估计的一致性
 当所有向量都“中心化”后，再进行归一化，就保证了每个向量都落在单位超球面上。这样，在后续距离计算（例如角度距离或余弦距离）中，所有向量都处于相同的尺度上，使得距离对比变得更加一致和稳定。如果直接归一化，由于整体存在偏移，不同向量之间的方向差异可能包含了全局偏移信息，而不是纯粹的相对方向信息。
3. 消除噪音和偏差
 在很多实际应用中，全局中心 c 可能携带了一些不需要的共性信息（例如背景信息、全局噪音等），减去这个中心之后，剩下的向量更能反映出各个数据的独特性。这对于后续的距离估计、聚类或量化处理都有显著帮助。

---
1.2 码表构建（Codebook Construction）
- 目标：
 构建一个“理想”的 codebook，使得码本中每个码（codeword）都是单位向量，并且在单位球面上均匀分布。这样一来，任何数据向量与码本中各个码的距离都有明确的几何解释，也便于理论误差分析。
- 原始构造：
 直接构造一个由 D 维二值化向量组成的集合：
$$C = \left\{ x \in \mathbb{R}^D : x_i \in \left\{-\frac{1}{\sqrt{D}}, \frac{1}{\sqrt{D}}\right\} \right\}$$
这样构成的集合对应于超立方体的顶点，所有向量都是单位向量，且理论上均匀分布在球面上。
- 引入随机性：
 为了解决直接使用 C 会对某些数据有偏的问题，RaBitQ 对 C 进行随机旋转：
$$C_{\text{rand}} = \{P x \mid x \in C\}$$
  其中 $$P$$ 是一个随机正交矩阵（即随机采样的旋转矩阵，这实际上就是 Johnson-Lindenstrauss 变换的一种形式）。这种随机旋转能使得 codebook 中的码不再偏向于某些固定的方向，从而使得任意单位向量更有可能获得一个更“公平”的近似表示。同时 JL 变换的另一个性质是，变换后的向量的相对距离和变换前是相近的。

---
1.3 Base 向量量化
- 过程：
 对于归一化后的数据向量 $$o$$，目标是找到 codebook $$C_{\text{rand}}$$ 中与之最近的码（按欧氏距离计），用这个最近的码来替换它。
 由于所有向量都是单位向量，问题可以转化为寻找使内积 $$\langle o, P x \rangle$$ 最大的 $$x\in C$$（因为距离 $$\|o - P x\|^2 = 2 - 2\langle o, P x\rangle$$）。
因为所有向量都被归一化成单位向量，所以每个向量的范数都是 1。这使得两个单位向量 $$o$$ 和 $$P x$$ 之间的欧氏距离可以用它们的内积来表示。具体地，我们有：
1. 对于单位向量 $$o$$ 和 $$P x$$ ，有
$$\|o\| = \|P x\| = 1.$$
2. 根据欧氏距离的公式：
$$\|o - P x\|^2 = \|o\|^2 + \|P x\|^2 - 2\langle o, P x \rangle.$$
3. 可以得到：
$$\|o - P x\|^2 = 1 + 1 - 2\langle o, P x \rangle = 2 - 2\langle o, P x \rangle.$$
4. 由此可见，距离越小（即 $$\|o - P x\|^2$$ 越小），对应的 $$2 - 2\langle o, P x \rangle$$ 就越小。这等价于 $$2\langle o, P x \rangle$$ 越大，也就是内积 ⟨o,Px⟩ 越大。
- 高效实现：
 注意到 $$P x$$ 的内积与 $$o$$ 之间的比较可以通过逆变换来实现，即：
$$\langle o, P x \rangle = \langle P^{-1} o, x \rangle$$
  因此，为避免对整个 codebook 进行昂贵的旋转计算（也就是计算 $$P x$$ ），我们可以先计算 $$q' = P^{-1} o$$，然后对 $$q′$$ 的各个分量取符号（正负），形成一个 D 位的二进制码 $$\bar{x}_b$$。
- 编码表示：
 将 $$\bar{x}_b$$ 作为量化码存储，实际上一个 D 位的比特串（每一位对应 $$x_i$$ 的正负，映射为 0/1）。由此，可以用一个很短的二进制码来表示原始向量的近似。
- 几何意义：
 这样，归一化后的数据向量 $$o$$ 就被近似表示为 $$\bar{o} := P \bar{x}$$，其中 xˉ 对应的二进制码  $$\bar{x}_b$$可以通过简单的位操作还原出来。
- 实现时是先旋转然后再归一化的

---
2. 查询阶段（Query Phase）
2.1 查询向量的预处理
- 归一化和逆变换：
 对于输入的查询向量 $$q_r$$，同样先归一化得到 q；然后计算 $$q' = P^{-1} q$$（旋转）。
- 实现时是先旋转然后再归一化的
2.2 距离计算与估计
- 传统方法问题：
 PQ 及其变体通常直接用查询向量与量化后的数据向量之间的距离作为近似距离，这种做法存在偏差，也缺乏理论上误差界。
- RaBitQ 的改进：
  RaBitQ 设计了一个新的距离估计器，核心是基于以下关系：
$$\|o_r - q_r\|^2 = \|o_r - c\|^2 + \|q_r - c\|^2 - 2 \cdot \|o_r - c\|\cdot\|q_r - c\| \langle q, o \rangle$$
其中 ⟨q, o⟩ 为归一化向量之间的内积。因为 $$\|o_r - c\|$$ 可以在索引构建时预计算，而 $$\|q_r - c\|$$ 的计算非常轻量，所以关键就转化为估计 ⟨q, o⟩。
这个公式利用余弦定理表达了两个原始向量 $$o_r$$（数据向量）和 $$q_r$$（查询向量）之间的欧氏距离，但不是直接计算，而是通过它们与一个中心点 c 的关系来表示。具体解释如下：
1. 归一化与中心化
 假设我们有一个中心点 c，通常是整个数据集或一个聚类的中心。我们把原始向量 $$o_r$$ 和 $$q_r$$ 都减去 c 后归一化成单位向量，分别记为
$$o = \frac{o_r - c}{\|o_r - c\|}, \quad q = \frac{q_r - c}{\|q_r - c\|}.$$
这样，它们的内积 ⟨q, o⟩ 就表示两向量之间的余弦相似度。
2. 余弦定理的应用
 根据余弦定理，对于任意三角形，其两边的长度和两边夹角可以用来计算第三边的平方。这里把 $$o_r, q_r, c$$ 看成三角形的三个顶点，则有：
$$\|o_r - q_r\|^2 = \|o_r - c\|^2 + \|q_r - c\|^2 - 2 \cdot \|o_r - c\| \cdot \|q_r - c\| \cos\theta,$$
其中 cos⁡θ = ⟨q, o⟩（因为 q 和 o 都是单位向量）。
3. 公式意义
  - $$\|o_r - c\|$$ 和 $$\|q_r - c\|$$ 分别是数据向量和查询向量与中心 c 的距离；
  - ⟨q, o⟩ 表示两个归一化向量之间的内积，也就是它们夹角的余弦值。
4. 整个公式说明了，原始向量 $$o_r$$ 与 $$q_r$$ 之间的距离可以分解为两部分：它们分别与中心 c 的距离，以及它们之间夹角（或内积）所引入的修正项。
总的来说，这个公式通过引入一个中心点 c 把原始距离分解成归一化后的内积部分，方便我们在量化和误差分析时使用统一的尺度和理论工具。

- 无偏估计器：
 论文通过对归一化后的数据向量 o（用其量化表示 $$\bar{o}$$）和查询向量 q 之间的内积进行分析，构造出一个无偏估计器：
$$\langle q, o \rangle \approx \frac{\langle \bar{o}, q \rangle}{\langle \bar{o}, o \rangle}$$
  并证明该估计器的误差上界是 $$O\left(\frac{1}{\sqrt{D}}\right)$$，且是理论上最优的（在 D 趋于无限时）。
  有了这个无偏估计之后，距离计算变成了：
  $$\|o_r - q_r\|^2 = \|o_r - c\|^2 + \|q_r - c\|^2 - 2 \cdot \|o_r - c\|\cdot\|q_r - c\| \langle q, o \rangle$$
  $$=  \|o_r - c\|^2 + \|q_r - c\|^2 - 2 \cdot \|o_r - c\|\cdot\|q_r - c\| \cdot \frac{\langle \bar{o}, q \rangle}{\langle \bar{o}, o \rangle}$$ 
  无偏估计的误差为：
  
$$\left|\frac{\left\langle\bar{\mathbf{o}}_0, \mathbf{q}\right\rangle}{\left\langle\bar{\mathbf{o}}_0, \mathbf{o}\right\rangle}-\langle\mathbf{o}, \mathbf{q}\rangle\right| \leq \sqrt{\frac{1-\left\langle\bar{\mathbf{o}}_0, \mathbf{o}\right\rangle^2}{\left\langle\bar{\mathbf{o}}_0, \mathbf{o}\right\rangle^2}} \cdot \frac{\epsilon_0}{\sqrt{D-1}}$$
  也就是说：
  $$\frac{\left\langle\bar{\mathbf{o}}_0, \mathbf{q}\right\rangle}{\left\langle\bar{\mathbf{o}}_0, \mathbf{o}\right\rangle} -  \sqrt{\frac{1-\left\langle\bar{\mathbf{o}}_0, \mathbf{o}\right\rangle^2}{\left\langle\bar{\mathbf{o}}_0, \mathbf{o}\right\rangle^2}} \cdot \frac{\epsilon_0}{\sqrt{D-1}} \leq 
\langle\mathbf{o}, \mathbf{q}\rangle \leq 
\frac{\left\langle\bar{\mathbf{o}}_0, \mathbf{q}\right\rangle}{\left\langle\bar{\mathbf{o}}_0, \mathbf{o}\right\rangle} +
\sqrt{\frac{1-\left\langle\bar{\mathbf{o}}_0, \mathbf{o}\right\rangle^2}{\left\langle\bar{\mathbf{o}}_0, \mathbf{o}\right\rangle^2}} \cdot \frac{\epsilon_0}{\sqrt{D-1}}$$
2.3 查询向量量化
使用完整的查询向量来进行计算是浪费的。可以将查询向量进行标量量化，将其量化成 𝐵𝑞-bit 的整型。
- 设：
  - $$v_l = \min_i q'[i]$$ （最小值）
  - $$v_r = \max_i q'[i]$$ （最大值）
- 定义分段大小：
  - $$\Delta = \frac{v_r - v_l}{2^{B_q} - 1}$$，其中 $$B_q$$ 是你要用多少 bit 来表示量化结果，比如 4 bits。
- 把区间 $$[v_l, v_r]$$ 均匀划分成 $$2^{B_q} - 1$$ 个小段，每段长度是 Δ。
  然后，对于 q′[i] 这样的一个数，如果它在某个小段里，会被四舍五入到左右端点最近的那个值，并用整数 m 或 m+1 来表示。也可以使用随机量化，通过加入一个随机数$$u_i$$，随机的分配到左右端点，避免量化引入的偏斜：
  $$\bar{q}_u[i] := \frac{q'[i] - v_l}{\Delta} + u_i$$
- 查询量化后的距离计算
量化之后，$$\langle \bar{o}, q \rangle$$ 的计算就可以用 $$\langle \bar{o}, \bar{q} \rangle$$ 来替代：
$$\begin{aligned}
& \langle \bar{\mathbf{o}}, \mathbf{q} \rangle = \langle \mathbf{P} \bar{\mathbf{x}}, \mathbf{q} \rangle = \mathbf{P}^{-1} \mathbf{P} \bar{\mathbf{x}}, \mathbf{P}^{-1} \mathbf{q} = \langle \bar{\mathbf{x}}, \mathbf{q'} \rangle 
\approx  \langle\mathbf{\bar{x}}, \mathbf{\bar{q}}\rangle\\
=&\left\langle\frac{2 \bar{\mathbf{x}}_b-\mathbf{1}_D}{\sqrt{D}}, \Delta \cdot \bar{\mathbf{q}}_u+v_l \cdot \mathbf{1}_D\right\rangle \\
= & \frac{2 \Delta}{\sqrt{D}}\left\langle\bar{\mathbf{x}}_b, \bar{\mathbf{q}}_u\right\rangle+\frac{2 v_l}{\sqrt{D}} \sum_{i=1}^D \bar{\mathbf{x}}_b[i]-\frac{\Delta}{\sqrt{D}} \sum_{i=1}^D \bar{\mathbf{q}}_u[i]-\sqrt{D} \cdot v_l
\end{aligned}$$
其中，$$\frac{2\Delta}{\sqrt{D}}$$、$$\frac{2 v_l}{\sqrt{D}} \sum_{i=1}^D \bar{\mathbf{x}}_b[i]$$以及 $$\frac{\Delta}{\sqrt{D}} \sum_{i=1}^D \bar{\mathbf{q}}_u[i]-\sqrt{D} \cdot v_l
$$ 这几部分都可以提取出来预先算好。
这几个参数也正是 Faiss 的 RaBitQ 实现里面的 c1、c2、c34 参数：
    query_fac.c1 = 2 * delta * inv_d;
    query_fac.c2 = 2 * v_min * inv_d;
    query_fac.c34 = inv_d * (delta * sum_qq + d * v_min);
    
    float final_dot = 0;
    // dot-product itself
    final_dot += query_fac.c1 * dot_qo;
    // normalizer coefficients
    final_dot += query_fac.c2 * sum_q;
    // normalizer coefficients
    final_dot -= query_fac.c34;

    // pre_dist = ||or - c||^2 + ||qr - c||^2 -
    //     2 * ||or - c|| * ||qr - c|| * <q,o> - (IP ? ||or||^2 : 0)
    const float pre_dist = or_c_l2sqr + query_fac.qr_to_c_L2sqr -
            2 * fac->dp_multiplier * final_dot;
- 高效实现：
 为了计算 ⟨oˉ, q⟩ 快速，论文提出两种实现方案：
  - 对于单个数据向量，使用位运算（因为 oˉ 的编码是二值码）进行内积计算；
  - 对于批量数据，则可以采用 SIMD 加速的方式，类似于 PQ Fast Scan 的实现。

---
2.3 查询步骤概述
- 预计算 LUT：
 查询阶段首先对查询向量 q 进行归一化、逆变换和量化处理，得到 q' 和相应的量化码或其更高精度表示。
- 距离估计：
对于每个数据向量，利用其预先存储的量化码以及预计算的 $$\|o_r - c\|$$ 和 ⟨oˉ, o⟩，结合查询向量的预处理结果，计算出无偏距离估计。
具体地，距离计算主要依赖于对 ⟨oˉ, q⟩ 的快速估计，然后根据公式还原出原始向量与查询向量的距离。
- 候选排序和精排：
 利用估计距离筛选候选向量，最后对于候选结果进行精排（re-ranking），以确保返回的近邻尽可能准确。

---
3. 总结
[图片]
3.1 构建查询总结
RaBitQ 的整个流程可以总结为：
1. 索引构建阶段
  - 归一化：把所有原始向量相对于一个中心 c 归一化到单位球面上；
  - 码本构建：利用固定的二值化向量（各坐标取 $$\pm \frac{1}{\sqrt{D}}$$）构造原始码本，再通过随机正交矩阵 P 进行旋转，得到均匀分布在单位球面上的代码集合；
  - 向量量化：对于每个数据向量，找到 codebook 中最近的码（即使内积最大），并将其表示为一个 D 位的二进制字符串，同时预计算一些辅助量（如 $$\|o_r - c\|$$ 和 ⟨oˉ,o⟩）。
2. 查询阶段
  - 对查询向量进行同样的归一化和逆变换（用 $$P^{-1}$$ ），得到 q'；
  - 利用高效的、无偏的距离估计器（基于 $$\langle \bar{o}, q \rangle/\langle \bar{o}, o \rangle$$），快速估计每个数据向量与查询向量的距离；
  - 根据估计距离筛选候选集，并对候选向量进行精排以返回最终结果。

3.2 需要保存或计算的量
为了实现高效搜索和反量化，RaBitQ 的码表构建阶段主要需要：
- 保存随机正交矩阵 P：用于定义一个隐式的随机旋转后的二值码本；
- 存储每个数据向量的量化码 $$\bar{x}_b$$（一个 D 位的二进制字符串），这允许用高效的位运算计算内积；
- 预计算并存储辅助信息：如 $$\|o_r - c\|$$ 和 ⟨oˉ, o⟩，这些在查询时用来校正无偏估计器；
- 设计高效的 LUT 或 SIMD 实现：帮助在查询时快速计算 ⟨oˉ,q⟩ ，从而完成无偏距离估计。

3.3 反量化流程
为了还原出原始向量，需要：
- 通过量化码 $$\bar{x}_b$$ 恢复量化后的二值向量 oˉ：
$$\bar{o}[i] = \begin{cases}  +\frac{1}{\sqrt{D}}, & \text{if } \bar{x}_b[i] = 1 \\  -\frac{1}{\sqrt{D}}, & \text{if } \bar{x}_b[i] = 0 \end{cases}.$$
- 应用正交矩阵的逆变换：
$$o \approx P^{-1} \bar{o}.$$
- 逆归一化
- 结合中心 c 和距离 $$\|o_r - c\|$$ 还原原始向量：
$$o_r = \|o_r - c\| \cdot o + c.$$
3.4 优缺点
这种设计的主要优势在于：
- 理论保证：给出无偏估计及其 $$O(1/\sqrt{D})$$ 的误差界；
- 计算高效：利用二值编码和 SIMD 优化可以极大提高查询速度；
- 鲁棒性：归一化和随机旋转能够缓解数据分布偏斜问题，使得量化更均衡，理论和实际误差都更低。
缺点也是有的，主要是：
- RaBitQ 只支持 欧氏距离 以及 余弦距离

4. RaBitQ 实现
4.1 数据生成
        # RaBitQ 论文实现需要先准备好 IVF 的 base vector X 以及数据分区的 centroids。
        X_pad         = np.pad(X, ((0, 0), (0, MAX_BD-D)), 'constant')
        centroids_pad = np.pad(centroids, ((0, 0), (0, MAX_BD-D)), 'constant')
        np.random.seed(0)

        # The inverse of an orthogonal matrix equals to its transpose. 
        # 1. 生成正交的投影矩阵 P
        P = Orthogonal(MAX_BD)
        # 对于正交矩阵，其转置等于其逆
        P = P.T
        
        # 变成一维数组
        cluster_id=np.squeeze(cluster_id)
        # 2. 对原始向量和聚类中心同时进行旋转
        XP = np.dot(X_pad, P)
        CP = np.dot(centroids_pad, P)
        # 3. 计算旋转后的残差
        XP = XP - CP[cluster_id]
        # 4. 二值化
        bin_XP = (XP > 0)
        
        # The inner product between the data vector and the quantized data vector, i.e., <\bar o, o>.
        # 5. 计算 <\bar o, o> 用于无偏估计的计算
        x0 = np.sum(XP[ : , :B] * ((2 * bin_XP[ : , :B] - 1) / B ** 0.5), axis=1, keepdims=True) / np.linalg.norm(XP, axis=1, keepdims=True)
        
        # To remove illy defined x0
        # np.linalg.norm(XP, axis=1, keepdims=True) = 0 indicates that its estimated distance based on our method has no error.
        # Thus, it should be good to set x0 as any finite non-zero number.  
        x0[~np.isfinite(x0)] = 0.8
        
        bin_XP = bin_XP[:, :B].flatten()
        uint64_XP = np.packbits(bin_XP.reshape(-1, 8, 8)[:, ::-1]).view(np.uint64)
        uint64_XP = uint64_XP.reshape(-1, B >> 6)
4.2 查询处理
生成的数据会被用来构建 IVFRN 索引，构建时主要是根据 cluster_id 将数据划分为各个 IVF。各个 IVF 之间连续存储，然后写到文件里面。后面使用时加载，主要的处理发生在加载里面：
static constexpr float fac_norm = const_sqrt(1.0 * B);
static constexpr float max_x1 = 1.9 / const_sqrt(1.0 * B-1.0);

template <uint32_t D, uint32_t B>
void IVFRN<D, B>::load(char * filename){
    std::ifstream input(filename, std::ios::binary);

    // 读取各个数据
    ......
    
    // 为每个向量计算距离估计的参数
    for(int i=0;i<N;i++){
        // ||o_r - c|| / <\bar o, o>
        long double x_x0 = (long double) dist_to_c[i] / x0[i];
        // ||o_r - c||^2
        fac[i].sqr_x = dist_to_c[i] * dist_to_c[i];
        fac[i].error = 2 * max_x1 * std::sqrt(x_x0 * x_x0 - dist_to_c[i] * dist_to_c[i]);
        fac[i].factor_ppc = -2 / fac_norm * x_x0  * ((float)space.popcount(binary_code + i * B / 64) * 2 - B);
        fac[i].factor_ip = -2 / fac_norm * x_x0;
    }
    input.close();
}
float tmp_dist = (ptr_fac -> sqr_x) + sqr_y 
        + ptr_fac -> factor_ppc * vl 
        + (result[i]-sumq) * (ptr_fac -> factor_ip) * width;
        
tmp_dist = ||o_r - c||^2 + ||q_r - c||^2 
              - 2 * ||o_r|| * ||q_r|| / <bar{o}, o>  * (ip_byte_bin * width - sumq * width / 2 + vl_correction)
[图片]

5. 推导

Hi, I was reviewing the implementation and comparing it with the equation presented in the paper:
$$\|o_r - q_r\|^2 = \|o_r - c\|^2 + \|q_r - c\|^2 - 2 \cdot \|o_r - c\| \cdot \|q_r - c\| \cdot \frac{\langle \bar{o}, q \rangle}{\langle \bar{o}, o \rangle}$$
However, in the code:
long double x_x0 = (long double) dist_to_c[i] / x0[i];
fac[i].sqr_x = dist_to_c[i] * dist_to_c[i];
fac[i].error = 2 * max_x1 * std::sqrt(x_x0 * x_x0 - dist_to_c[i] * dist_to_c[i]);
fac[i].factor_ppc = -2 / fac_norm * x_x0 * ((float)space.popcount(binary_code + i * B / 64) * 2 - B);
fac[i].factor_ip = -2 / fac_norm * x_x0;

float tmp_dist = sqr_x + sqr_y + factor_ppc * vl + (ip_term) * factor_ip * width;
float error_bound = y * error;
*ptr_res = tmp_dist - error_bound;
It seems that the term ||q_r - c|| in the last component of the formula (the cross term) is missing. In the paper, the third term is:
$$-2 \cdot \|o_r - c\| \cdot \|q_r - c\| \cdot \frac{\langle \bar{o}, q \rangle}{\langle \bar{o}, o \rangle}$$
But in the code, factor_ip and factor_ppc includes only ||o_r - c|| / <\bar{o}, o>, and it's not clear whether ||q_r - c|| is absorbed into width, approximated, or intentionally omitted.
Could you please clarify whether this term is approximated elsewhere, or if this might be an implementation issue?
Thanks!

This picture below shows the relationship between equation and code:
