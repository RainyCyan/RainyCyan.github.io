RabitQ
1. 公式整理
1.1. 为什么需要进行旋
转和中
1.2. RabitQ 无偏估计是如何得来的
1.3. RabitQ+ 量化过程
2. 量化-量化距离计算
2.1. 从原
始距离到归一化后距离
2.2. 从归一化距离到量化距离
2.3. 由量化码值计算量化后内积
2.3.1. 单 bit 时
2.3.2. 多 bit 时
心归一化？
2.4. 合起来
2.4.1. 单 bit 时
2.4.2. 多 bit 时
2.5. 测试
2.5.1. 单 bit
2.5.2. 单 bit code2code
2.5.3. 多 bit
2.5.4. 多 bit code2code
之前看 rabitq 的时候写的，有些地方有误没改。
●
●
●
作者开源的 rabitq 实现 里面的 doc 里面有详细的
作者讲 rabitq 的博客
作者讲 rabitq+ 的博客
●
知乎上一个文章
RabitQ
_
量化方法.pdf
●
elastic-search 的测试
推
导
1
1. 公式整理
2
3
4
1.1. 为什么需要进行旋
转和中
心归一化？
●
为什么要进行旋
转？
让我们回到文中的例子：
○
原
■
■
■
始状态：
码本 ﻿ C﻿：它的码字都长得像 ﻿ (±1/√D, ..., ±1/√D)﻿，非常规整，方向固定。
数据向量 ﻿ v1 = (1, 0, ..., 0)﻿：这个向量和码本 ﻿ C﻿ 中的任何一个码字
都“不匹配”，导致量化误差很大。
数据向量 ﻿ v2 = (1/√D, ..., 1/√D)﻿：这个向量和码本 ﻿ C﻿ 中的一个码字完美
匹配，量化误差为0。
○
旋
转之后：
■
■
■
我们把
码本 ﻿ C﻿ 随机旋
转了一下，得到了新的码本 ﻿ C_rand﻿。
现在， C_rand﻿ 的码字方向是随机的，不再和坐标轴有特殊的关系。
对于数据向量 ﻿ v1﻿ 和 ﻿ v2﻿ 来说，它们现在与 ﻿ C_rand﻿ 中码字的距离变得更
加
“公平”了。﻿ v2﻿ 不再享有特殊优势，而 ﻿ v1﻿ 也不会受到特殊惩罚。旋
转操作将
这种“好
运”和“厄运”在整个向量空间中进行了均匀的重新分配。
旋
转之后，整个向量空间中数据向量之间的相对关系（比如哪些向量聚成一簇，哪些向量彼
此远
离）是保持不变的。旋
转的目的是改变码本的方向，从而打破
始确定性
原
码本对特定方向向量
的“偏爱”，使得码本对于来自任何方向的数据向量都能提供一个性能上更稳定、更公平的量化，
最终降低整体的平均
量化误差。
5
●
为什么要进行中
心归一化？
归一化的目的是让 base 向量也对齐到超球面上，和码本向量一致；而引入中
心点进行归一化是为了在
一定程度上消
除数据集的偏差，减去中
心点能够使得向量更加均匀的分布在超球面上。类似于建立的是
以中心点为
原
点的坐标系。
1.2. RabitQ 无偏估计是如何得来的
●
高维单位向量的维度聚集特性
如上图所示，对于单位向量，随着维度升高，其每一个维度的元素就越趋向于 0。
这个挺好理解
：在 3 维时，向量模长只由 3 个维度贡献，一个维度取值大一点对其他维度影响较小。而
在 1000 维时，模长由 1000 个维度贡献；此时如果一个维度取值大了，其他维度会被
急剧压缩。因此这
种反常点的数量是很有限的。
JL 引理 证明了高维向量中的每个元素的取值不太可能偏移 0 超过 2 / sqrt(D).
●
基于几何关系建立的无偏估计
6
因为我们关心的是，o 和 q 的内积，我们可以在 o 和 q 张成的平面上建立坐标系。o 和 e1 分别为两个
坐标轴。
这时候 可以将 o_bar 和 q 的内积分解为两个坐标轴方向的内积之和。也就是说，将 o_bar 投影到 o 以
及垂直于 o（e1） 两个方向；然后这两个分量分别和 q 计算内积，然后再相加。
因此可以得到：
这个式子里面，后面一项是难以提前计算的（它和 o 以及 q 都有关）。这时候 高维向量维度聚集现象
就派上用场了。作者生成了大量的随机向量进行投影，发现 ō 在平面上的投影高度聚集在 o 方向上，
且在 0.8 附近。而在垂直于 o 方向上的投影则趋近于 0.
你可能会问，为什么 <ō, e₁> 会集中在0附近？
﻿﻿
● 想象一个三维空间。 o 和 q 定义了一个二维平面。一个随机选择的向量 ō 恰好落在这
﻿﻿﻿﻿﻿
个平面内的概率有多大？非常小。它更可能与这个平面有一个夹角。
7
● 现在想象一个 128 维的空间。 o 和 q 定义的仍然只是一个“薄薄”的二维平面。一个随
﻿﻿﻿﻿
机的 128 维向量 ō ，几乎不可能与这个特定的二维平面有任何显著的对齐。它几乎总
﻿﻿
是“指向”其他 126 个维度
中的某个方向。
● 换句话说，一个随机的高维向量 ō 几乎必然与一个给定的低维子空间（如 o 和 q 张
﻿﻿﻿﻿﻿﻿
成的平面）是近乎正交的。
● 因为 e₁ 是这个平面内的一个向量，所以 ō 也几乎必然与 e₁ 是近乎正交的。
﻿﻿﻿﻿﻿﻿
● 两个向量正交，意味着它们的内积为 0。
因此，可后面一项可以直接舍去，仍然可以有很高的概率得到精确的结果。舍去后面一项之和，就
得到了无偏估计。
1.3. RabitQ+ 量化过程
为了利
用 RaBitQ 的无偏估计，RaBitQ+ 也将 base 向量（旋
转后）进行了归一化。然后再从归
一化的后的向量出发，寻找其量化表示。也就是从高维球面上的点出发，寻找最接近它的高维网格
里面的量化表示。但是，高维网格里面的点是近乎无限多的，不可能一一遍历。
引理 3.1 表明，如果 ȳ ̄ 是与 o' 余弦相似度最高的向量，那么一定存在一个缩放因子 t > 0，使得 ȳ ̄ 同时也是与
缩放后的向量 t·o' 欧氏距离最近的网格点（这个网格点就是 t·o' 直接取整的那个点）。基于此，我么可以遍
历每一个 t，然后对 o' 进行缩放取整得到 ȳ ̄，之后再计算 ȳ ̄ 与 o' 的余弦距离。最终的量化值 ȳ ̄ 就是余弦距离最
大的那一个。
具体实现时只枚举了那些使 o' 缩放后产生变化的 t，同时在维护余弦距离时采用了增量式的计算。
2. 量化-量化距离计算
在分布式构图的场景下，我们首先对数据进行 partition，然后对每个 partition 使用构图算法进行构图。
这就得到了若干子图。之后我们需要将子图进行合并，得到最终的整图。拼接子图的方式有多种，
RaBitQG 里面采用了和 DiskANN 类似的方法：分割数据时（一般通过聚类），将一个数据点划分到多
个分片里面（我们取的副本数为 2）。这样
每个数据点都作为桥梁连接起了多个子图。进行子图合并
时，使用剪枝算法对这些邻居进行裁剪即可。
使用 HNSW 的剪枝算法有一个问题：由于构建好子图之后我们就已经没有
计算量化后的两个数据点之间的距离。而数据点的归一化中
能直接使用了。
原
始数据了，这就需要我们去
心点可能是不同的，使得原
论文里的公式不
8
我们需要
导一下不同中
推
心点的量化数据之间的距离估计公式。
2.1. 从原
始距离到归一化后距离
9
新的公式为：
∥o
−
r q ∥ =
r
2 ∥o
−
r c ∥ +
o
2 ∥q
−
r c ∥ +
q
2 ∥c
−
o c ∥ −
q
2 2⋅ ∥o
−
r c ∥⋅ ∥q
−
o r c ∥⋅ ⟨o, q⟩
q
+2⋅ ∥o
r c ∥⋅ ⟨o, c
−
o o c ⟩ −
−
q 2⋅ ∥q
r c ∥⋅ ⟨q, c
−
−
q o c ⟩
q
≈ ∥o
r c ∥ +
−
o
2 ∥q
r c ∥ +
−
q
2 ∥c
o c ∥ −
−
q
2 2⋅ ∥o
r c ∥⋅ ∥q
−
o r c ∥⋅ ⟨o, q⟩
−
q
oˉ
qˉ
+2⋅ ∥o
r c ∥⋅ ⟨ , c
−
o c ⟩ −
−
q 2⋅ ∥q
r c ∥⋅ ⟨ , c
−
o c ⟩
−
oˉ
o ⟨o, ⟩
q ⟨q, ⟩
qˉ
q
= ∥o
r c ∥ +
−
o
2 ∥q
r c ∥ +
−
q
2 ∥c
o c ∥ −
−
q
2 2⋅ ∥o
r c ∥⋅ ∥q
−
o r c ∥⋅ ⟨o, q⟩
−
q
2⋅ ∥o
− c ⟩r o oˉ
− c ∥⋅ ⟨ , c
o q
2⋅ ∥q
− c ⟩r q qˉ
− c ∥⋅ ⟨ , c
o q
+
−
oˉ
⟨o, ⟩
⟨q, ⟩
qˉ
其中的 L1、L2 范数都是可以提前计算好
的，最终的问题还是转换为了计算 <o, q>.
2.2. 从归一化距离到量化距离
构建好子图之后，
原
始向量以及归一化之后的
原
始向量我们都没有了。我们只有量化后的向量。
问题转换为了：如何使用量化后的向量来估计归一化的向量距离？
根据 RaBitQ 无偏距离公式，我们有：
oˉ ⟨ , q⟩ =
oˉ
⟨ , o⟩⋅ ⟨o, q⟩
⟨o, ⟩ =
qˉ ⟨ , q⟩⋅ ⟨o, q⟩
qˉ
的随机矩阵进行旋
无偏估计有一个前提：两个向量必须使用同样
随机旋
转矩阵要是一样
的。
我们需要用 <¯o,¯q> 来表示 ⟨o,q⟩ 。
转！也就是说构建子图时，子图使用的
10
11
12
最终的表示为：
●
使用共线性来推
导
⟨o, q⟩ =
oˉ
⟨ , ⟩
qˉ
oˉ
(⟨ , o⟩ ⋅ ⟨ , q⟩)
qˉ
●
使用算术平均进行
导
推
⟨o, q⟩ =
oˉ
⟨ , ⟩
qˉ
2
●
使用几何平均进行
导
推
⟨o, q⟩ =
⋅
1
( +
oˉ
⟨ , o⟩
oˉ
⟨ , ⟩
qˉ
oˉ
(⟨ , o⟩ ⋅ ⟨ , q⟩)
qˉ
1
⟨ , q⟩
qˉ
)
<o_bar, o> 和 <q_bar, q> 都是可以提前算好
化后的数据我们都是有的。
的。最终需要实时计算的只有 <o_bar, q_bar>，这两个量
2.3. 由量化码值计算量化后内积
2.3.1. 单 bit 时
即计算 <o_bar, q_bar>。这里假设两个向量是使用同一个随机矩阵进行旋
转的，那么：
oˉ
qˉ ⟨ , ⟩ =
⟨P X , P X ⟩ =
o q ⟨X , X ⟩o q
2X
ob
− 1D
2X
qb
− 1D
= ⟨ , ⟩
D
D
=
1 (4⟨X , X ⟩ −
⋅
ob qb 2 ⋅ SumX
o 2 ⋅ SumX +
−
q D)
D
2.3.2. 多 bit 时
13
yˉ
zˉ
oˉ
⟨ , ⟩ =
qˉ ⟨P , P ⟩
∣∣ ∣∣
yˉ
zˉ
∣∣ ∣∣
1
1
=
⟨ , ⟩
yˉ
zˉ
∣∣ ∣∣
yˉ
zˉ
∣∣ ∣∣
1
1
=
⟨
y
ˉ (2 −
−
u
B 1)/2 ⋅ 1D,
z
ˉ (2 −
−
u
B 1)/2 ⋅ 1D⟩
∣∣ ∣∣
yˉ
zˉ
∣∣ ∣∣
1
1
B
2 − 1
B
2 − 1
=
(⟨y , z ⟩ −
u u
⋅
Sumy
u
−
⋅
∣∣ ∣∣
yˉ
zˉ
∣∣ ∣∣
2
2
B
2 − 1
Sumz +
2
u D( ) )
2
注
意：这里假定了 y_u 减去 (2^B - 1) / 2 就能恢复出 o_bar 了。在实际的实现中，由于多 bit 量化编
码的不同，这里的复
方式也是不同的。
原
2.4. 合起来
这里只展示了共线性
推
导的整体公式。使用算数、几何平均
的公式是类似的，将相应项替换掉即可。
2.4.1. 单 bit 时
∥o
r q ∥ =
−
r
2 ∥o
r c ∥ +
−
o
2 ∥q
r c ∥ +
−
q
2 ∥c
o c ∥
2
−
q
1
−2⋅ ∥o
r c ∥⋅ ∥q
−
o r c ∥⋅
D
−
qˉ (4⟨X , X ⟩ − 2SumX
ob qb o q
− 2SumX + D)
oˉ
q (⟨ , o⟩ ⋅ ⟨ , q⟩)
+2⋅ ∥o
r c ∥⋅ ⟨o, c
−
o o c ⟩ −
−
q 2⋅ ∥q
r c ∥⋅ ⟨q, c
−
−
q o c ⟩
q
2.4.2. 多 bit 时
∥o
r q ∥ =
−
r
2 ∥o
r c ∥ +
−
o
2 ∥q
r c ∥ +
−
q
2 ∥c
o c ∥
2
−
q
−2⋅ ∥o
r c ∥⋅ ∥q
−
−
o r
B
B
1
1
2 −1
∣∣ ∣∣
zˉ
∣∣ ∣∣
u u 2
⋅ Sumy
−
c ∥⋅
yˉ
qˉ (⟨y , z ⟩ −
2 −1
u 2
2 −1
B 2
⋅ Sumz + D⋅ ( ) )
u 2
q (⟨ , o⟩ ⋅ ⟨ , q⟩)
oˉ
+2⋅ ∥o
−
r c ∥⋅ ⟨o, c
−
o o c ⟩ −
q 2⋅ ∥q
−
r c ∥⋅ ⟨q, c
−
q o c ⟩
q
yu、zu 分别为 o 和 q 的量化码。
2.5. 测试
随机生成 256 维的向量，经过旋
实距离比较，计算误差。
转、中
心归一化、量化之后，使用上面的三种推
1-bit 生产了十万条随机数据进行测试，多 bit 编码太慢了，只生成了一千
条数据。
导进行距离计算。和真
14
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
Python
def error_rand(zeroCentroid, diffCentroid, estimator):
global total_exper, total_error
if zeroCentroid:
c_a = np.zeros(D)
c_b = np.zeros(D)
elif diffCentroid:
c_a = rand_vec()
c_b = rand_vec()
else:
c_a = rand_vec()
c_b = c_a
a, a_norm, a_q = rand_vec_with_centoid(c_a)
b, b_norm, b_q = rand_vec_with_centoid(c_b)
expect = np.sqrt(np.dot(a-b, a-b))
estimate = 0
estimate += np.dot(a-c_a, a-c_a)
estimate += np.dot(b-c_b, b-c_b)
if diffCentroid:
estimate += np.dot(c_a-c_b, c_a-c_b)
estimate -= 2 * np.sqrt(np.dot(a-c_a, a-c_a)) * np.dot(c_b-c_a, a_
q) / np.dot(a_norm, a_q)
estimate -= 2 * np.sqrt(np.dot(b-c_b, b-c_b)) * np.dot(c_a-c_b, b_
q) / np.dot(b_norm, b_q)
# estimate -= 2 * np.sqrt(np.dot(a-c_a, a-c_a)) * np.dot(c_b-c_a,
a_q)
# estimate -= 2 * np.sqrt(np.dot(b-c_b, b-c_b)) * np.dot(c_a-c_b,
b_q)
if estimator == 'algavg':
estimate -= np.sqrt(np.dot(a-c_a, a-c_a)) * np.sqrt(np.dot(b-c_b,
b-c_b)) * np.dot(a_q, b_q) * (1 / np.dot(a_q, a_norm) + 1 / np.dot(b_q, b_
norm))
elif estimator == 'colinear':
estimate -= 2 * np.sqrt(np.dot(a-c_a, a-c_a)) * np.sqrt(np.dot(b-c
_b, b-c_b)) * np.dot(a_q, b_q) / np.dot(a_q, a_norm) / np.dot(b_q, b_norm)
elif estimator == 'geoavg':
estimate -= np.sqrt(np.dot(a-c_a, a-c_a)) * np.sqrt(np.dot(b-c_b,
b-c_b)) * np.dot(a_q, b_q) / np.sqrt(np.dot(a_q, a_norm) * np.dot(b_q, b_n
orm))
estimate = np.sqrt(estimate)
total_error = total_error + abs((expect - estimate)/expect)
15
37
total_exper += 1
2.5.1. 单 bit
在 1-bit 上测了一下三种的误差。
上面是后两项除上内积的，下面是不除的。
Python
1
2
3
4
5
6
7
在相同中
# 第一个结果
ot(a_norm, a_q)
ot(b_norm, b_q)
estimate -= 2 * np.sqrt(np.dot(a-c_a, a-c_a)) * np.dot(c_b-c_a, a_q) / np.d
estimate -= 2 * np.sqrt(np.dot(b-c_b, b-c_b)) * np.dot(c_a-c_b, b_q) / np.d
# 第二个结果
estimate -= 2 * np.sqrt(np.dot(a-c_a, a-c_a)) * np.dot(c_b-c_a, a_q)
estimate -= 2 * np.sqrt(np.dot(b-c_b, b-c_b)) * np.dot(c_a-c_b, b_q)
心点的时候，除和不除没啥区别。在不同中
心点的时候除上内积的误差会更小。
2.5.2. 单 bit code2code
单 bit 的编码很简单，没有额外的转换。因此
推
导出的公式也是直接就能够使用的。
16
2.5.3. 多 bit
多 bit 时，几何平均
●
8-bit
的劣势就很明显了。共线性
和算术平均差
别不大。
●
5-bit
●
4-bit
共线性
推
个需要进行修改。
导出的式子有一个好处是，在归一化和不归一化 o_bar 两种情况下，式子是一样
的，而另外两
17
●
5-bit 256 维
●
5-bit 64 维
2.5.4. 多 bit code2code
在上面的多 bit 结果是直接使用 o_bar 来进行计算的。在实际的距离计算中，我们肯定是不想要这个显
示的还
原
过程的（由量化码值复
原
出 o_bar）。因此要考虑直接使用量化码值来进行距离的估计。
这就需要思考一下码值和 o_bar 的关系。在多 bit 量化时，我们是将
原
始的向量（经过旋
转后）量化
到：
B−1 B−1
[−2 , 2 ]
18
这个以
原
点对称的区间上的。然后，我们再将量化后的值转换为紧凑的编码。为了避免对负数进行编
码，我们对负数进行了处理——将其映射正数范围内。此时量化码的值域变成了：
[0, 2 −
B 1]
这时候就可以很简单的进行编码了。
在计算时，我们需要根据编码值来恢复 o_bar（实际计算时不需要，但是需要以此来进行公式
推
我们现有的编码方法为：
导）。
Python
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
// Currently assume all number in oBar are positive. For negative
// values, the quantization code needs to be flipped.
uint32_t mask = (1 << (bitCount - 1)) - 1;
for (size_t i = 0; i < dimension; i++) {
codePerDimension[i] = uint32_t(expectT * (double)oBar[i] + 1e-8);
if (codePerDimension[i] > maxQuantizeValue) {
codePerDimension[i] = maxQuantizeValue;
}
if (input[i] < 0) {
oBar[i] = -(codePerDimension[i] + 0.5);
codePerDimension[i] = (~codePerDimension[i]) & mask;
} else {
oBar[i] = codePerDimension[i] + 0.5;
codePerDimension[i] &= mask;
}
}
正数部分不变，负数部分映射到正数区域的高位置（负数越接近 0，编码后越接近最大值）。这种方式
有一定的取巧，它将正数和负数的区域混合在一起了，能够在有限的位宽内编码更大的值。但是这样就
导致了一个问题：在不存储符号位的情况下，不能精准的复
原
出 o_bar（因为正数和负数的区域和在一
起了，不能简单区分出正负数）。
这使得前面的公式
推
也就没有意义了。直接套
导里面的简单减去 (2^B - 1) / 2 的方式不能恢复出编码前的 o_bar。
用上面的公式，误差炸裂。。
推
导出的公式
实际上 binary code 就是符号位，我们是有存储符号位的。
但是，计算 code2code 的内积的时候，由于我们现在两边都有两项（1 bit 和 ex-bit），这时计算完
整的内积结果就需要 4 次内积运算。。
4 次内积和先恢复出 o_bar 再进行一次 float- float 内积哪个更高效呢？
因此我在测试时做了一个简单的改动，编码时将 o_bar 的值域整体右移，使得负数和正数分别占据正数
区域的低位置
和高位置。这样就能很简单的恢复出 o_bar 了。前面的公式也就成立了。
19
Python
1
2
3
4
5
6
7
8
9
10
11
12
13
# 编码
o_bar = np.floor(t * o_abs + eps)
mask = (1 << (bit_count - 1)) - 1
o_bar_code = np.where(o_bar > mask, mask, o_bar)
o_bar_code = np.where(o < 0, (mask - o_bar_code), o_bar_code + mask)
.astype(np.uint32)
#计算
CB = (1 << (bit_count - 1)) - 1
sum_aq = np.sum(a_q_code)
sum_bq = np.sum(b_q_code)
dot_aq_bq = np.dot(a_q_code, b_q_code) - CB * (sum_aq + sum_bq) + D * CB *
CB
dot_aq_bq /= norm_a_q
dot_aq_bq /= norm_b_q
在 5 bit 下进行验证：
●
直接使用 o_bar 进行计算
●
直接使用 o_bar 进行计算，o_bar 不加上 0.5 的补偿
●
使用量化后的码值进行计算
20
简单修改一下编码方式后，是能够恢复出 o_bar 的，
推
导出的公式也能保证很小的误差。
同时，对 o_bar 加上 0.5 的补偿项是能够提升精度的，但是这也使得恢复 o_bar 时会多几步计算（判断
正负，然后分别加上或减去 0.5）。具体实现时需不需要还要权衡一下