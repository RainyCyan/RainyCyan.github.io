
一道经典面试问题是这样的：我们都知道本地处理TopK问题使用最小堆，如果数据量扩大到1万亿且分布式存储，方案如何调整？
我们注意到，TopK问题是满足结合律的，即一个集合的 TopK，一定包含在各个子集合 TopK 的并集里。那整体方案设计中最小堆依然是每个节点上的局部算子，但整体框架变成 Local TopK → Merge TopK 的两阶段(MapReduce)结构。

下面是来自ai的回答：

  分布式方案:两阶段归并

  假设:1 万亿条数据分布在 M 台机器(比如 1000 台,每台 10 亿条)。

  Phase 1 — Local TopK(Map)

  - 每台机器对本地分片独立跑最小堆,各自产出本地 TopK。
  - 单机代价 O(N/M · log K),完全并行。
  - 关键正确性论断:全局 TopK 的每一个元素,必然是它所在机器的 Local TopK 成员。所以只需从每台机器上收
    K 个候选,而不是全量。

  - 网络传输从 M × (N/M) 降到 M × K,数据量塌缩到 M·K(比如 1000×100 = 10 万,而非 1 万亿)。

  Phase 2 — Merge TopK(Reduce)

  - 收集 M 组 Local TopK,共 M·K 个候选,再求一次全局 TopK。
  - 两种做法:
      - 每组已排序 → K 路归并(用一个大小为 M 的堆按组头比较),O(K log M);
      - 或直接对 M·K 个候选再跑一次大小 K 的最小堆,O(M·K · log K)。

  - 数据量大时归并本身也可以做成多层树形归并(避免单点 reducer 成为瓶颈),即先局部合并成中间层,再逐层
    上收。

## Map-Reduce
这是lx的blog:https://thisingl.github.io/%E5%88%86%E5%B8%83%E5%BC%8F%E7%B3%BB%E7%BB%9F/

map-reduce paper:https://www.usenix.org/legacy/event/osdi04/tech/full_papers/dean/dean_html/

q1:如何将原始问题建模出合适的key-value对？
q2:问题需要满足什么性质才可应用map-reduce

来自论文原文：`MapReduce 编程模型的原理是：利用一个输入 key/value pair 集合来产生一个输出的 key/value pair 集合。
MapReduce 库的用户用两个函数表达这个计算：Map 和 Reduce。
用户自定义的 Map 函数接受一个输入的 key/value pair 值，然后产生一个中间 key/value pair 值的集合。
MapReduce 库把所有具有相同中间 key 值 I 的中间 value 值集合在一起后传递给 reduce 函数。`

q3:map函数和reduce函数指代的key-value含义可能不同

## 分布式/并行结构
树状结构

## raft