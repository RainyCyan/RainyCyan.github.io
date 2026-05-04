---
title: B-Tree Index 核心逻辑梳理
date: 2026-05-04T09:46:00Z
tags: [数据库 & 存储]
series: []
featured: true
---
# B-Tree Index

> 版本管理：TL;DR（1min）→ Theatrical Cut（5-10min）→ Director's Cut（>30min）
> 点击展开每一层，按需深入。

<!--more-->

---

## TL;DR — 一分钟版本

{{< collapse summary="👆 展开 TL;DR（1 分钟浏览）" openByDefault=true >}}

**B-Tree 是为了解决磁盘 I/O 瓶颈而设计的数据结构。**

核心思路：BST/AVL 每个节点只存 1 个 key，树高 O(log₂N)。N=10 亿时树高约 30，查一次要 30 次随机 I/O，太慢。B-Tree 让**一个节点存多个 key**，树高降到 O(logₘN)（m 是阶数），m=500 时树高不到 4。

三个核心操作：
- **搜索**：和 BST 一样，节点内二分查找，沿路径向下
- **插入**：先插到叶子，满了就**分裂**，中间 key 上提，树从下往上生长
- **删除**：删了不够就**借**，借不到就**合并**，向上传播

B+Tree 是实际数据库用的变体：内部节点只存 key（不存数据），叶子节点链表相连，范围查询直接顺序扫描。

> 一句话：**用节点内的空间换树的高度，用树的高度换 I/O 次数。**

{{< /collapse >}}

---

## Theatrical Cut — 正文阅读

{{< collapse summary="👆 展开 Theatrical Cut（5-10 分钟阅读）" >}}

### 核心问题：为什么需要 B-Tree？

> 所有数据结构的演进都是为了解决一个核心矛盾：**内存快但小，磁盘慢但大**。

在磁盘上做查询，瓶颈是 **I/O 次数**（寻道 + 旋转延迟远大于数据传输本身）。因此目标很明确：**用最少的磁盘访问次数，找到目标数据**。

BST / AVL / Red-Black Tree 的问题：每个节点只存一个 key，树高 = O(log₂N)。当 N 很大时（比如 10 亿行），树高约 30，每次查询需要 30 次随机 I/O，太慢。

B-Tree 的核心思路：**一个节点存多个 key，让树变矮**。树高 = O(logₘN)，m 是每个节点的分支数（阶数）。m 越大，树越矮，I/O 越少。

---

### B-Tree 的结构定义

> 先看定义，再看为什么这样定义。

一个 m 阶 B-Tree 满足：

1. **每个节点最多有 m 个子节点**
2. **除根节点外，每个节点至少有 ⌈m/2⌉ 个子节点**（保证树不会退化成链表）
3. **每个节点有 k 个 key，则它有 k+1 个子节点**（key 把区间切分成 k+1 段）
4. **所有叶子节点在同一层**（绝对平衡）

> 第 4 条是最关键的特性：B-Tree 是**绝对平衡**的，所有叶子在同一深度。这意味着**任何 key 的查询路径长度相同**，最坏情况 = 最好情况。

---

### 核心操作逻辑

#### 搜索

> 和 BST 完全一样，只是每个节点内做二分查找。

```
search(node, key):
    在 node.keys 中二分查找 key 的位置 i
    if node.keys[i] == key: return node
    if node.is_leaf: return null
    return search(node.children[i], key)
```

每个节点内部用二分（O(log m)），但 m 通常很小（几百），且节点内操作在内存中，成本可忽略。**真正的成本是沿路径的节点访问次数 = 树高**。

---

#### 插入

> 插入的核心逻辑是：**先插到叶子，满了就分裂**。

```
insert(key):
    root = find_leaf_to_insert(key)
    root.keys.push(key)  // 插入到叶子节点
    while root.keys.size == m:  // 节点满了
        mid = m/2
        left = keys[0..mid-1], right = keys[mid+1..m-1]
        promote_key = keys[mid]
        将 promote_key 插入父节点
        父节点将 left/right 作为两个子节点
        root = root.parent
    if root == null: 创建新根节点
```

**关键理解**：
- 分裂时，中间 key **上提**到父节点，左右两边成为两个独立子节点
- 上提可能导致父节点也满，继续分裂，**向上传播**
- 只有根节点分裂时，树的高度才增加 1
- 这就是 B-Tree **从下往上生长**的方式——和 BST 从上往下插入完全不同

---

#### 删除

> 删除的核心逻辑是：**删了之后不能低于最小度数，不够就借，借不到就合并**。

```
delete(key):
    node = find(key)
    if node is internal:
        用前驱/后继 key 替换，转化为删除叶子节点
    // 现在要删除的是叶子节点
    node.keys.erase(key)
    if node.keys.size < min_degree - 1:  // 不满足最小 key 数
        if 兄弟节点有富余:
            从父节点借一个 key，兄弟补一个 key 给父节点（旋转）
        else:
            合并当前节点和兄弟节点，父节点 key 下移
            递归检查父节点是否满足条件
```

**关键理解**：
- 删除内部节点时，用前驱/后继替换，**转化为删除叶子节点**——这是简化问题的核心技巧
- 借 key 的操作本质是**父节点 key 下移 + 兄弟节点 key 上移**，类似 AVL 的旋转
- 合并操作可能导致父节点 key 不足，**向上传播**，直到根节点
- 根节点 key 数变为 0 时，树高减 1

---

### B+Tree：B-Tree 的变体

> B+Tree 是 B-Tree 的改进，**几乎所有实际数据库（MySQL InnoDB、PostgreSQL）用的都是 B+Tree**。

#### 与 B-Tree 的区别

| 特性 | B-Tree | B+Tree |
|------|--------|--------|
| 数据存储位置 | 所有节点都存数据 | **只有叶子节点存数据** |
| 内部节点 | 存 key + 指针 | **只存 key（路由用）** |
| 叶子节点 | 独立 | **链表相连** |
| 范围查询 | 需要中序遍历 | **叶子链表顺序扫描** |

#### 为什么 B+Tree 更好？

1. **内部节点更小** → 不存数据只存 key → 一个节点能存更多 key → 树更矮 → I/O 更少
2. **范围查询高效** → 叶子节点链表相连 → 找到下界后顺序扫描即可，无需回溯父节点
3. **查询更稳定** → 所有查询都走到叶子节点 → 路径长度完全一致

---

### 总结：B-Tree 的设计哲学

> 一句话：**用节点内的空间换树的高度，用树的高度换 I/O 次数**。

- 每个节点存多个 key → 分支因子 m 变大 → 树高 logₘN 变小 → I/O 次数变少
- 节点大小通常对齐**磁盘页大小**（4KB / 16KB）→ 一次 I/O 读入一个完整节点 → 最大化每次 I/O 的信息量
- 绝对平衡 → 最坏情况 = 最好情况 → 查询延迟可预测

这就是 B-Tree 成为数据库索引标准选择的原因：**在磁盘 I/O 这个约束下，把查询效率推到了极致**。

{{< /collapse >}}

---

## Director's Cut — 附录与完整细节

{{< collapse summary="👆 展开 Director's Cut（>30 分钟深度阅读）" >}}

### 1. B-Tree 的数学证明

#### 树高下界

对于一个 m 阶 B-Tree，有 N 个 key，树高 h 满足：

```
h ≤ 1 + log_{⌈m/2⌉}((N+1)/2)
```

推导：根节点至少 1 个 key，2 个子节点；其他节点至少 ⌈m/2⌉ - 1 个 key，⌈m/2⌉ 个子节点。因此第 1 层至少 1 个节点，第 2 层至少 2 个，第 3 层至少 2·⌈m/2⌉ 个……

当 m=500，N=10⁹ 时，h ≤ 1 + log₂₅₀(5×10⁸) ≈ 1 + 3.6 ≈ 5。即最多 5 次 I/O 就能找到任意 key。

#### 为什么分裂/合并能保证平衡？

B-Tree 的平衡不是通过旋转（如 AVL）或染色（如红黑树）来维护的，而是通过**结构约束**：

- 插入时，节点满了就分裂，key 数始终 ≤ m-1
- 删除时，节点 key 太少就借或合并，key 数始终 ≥ ⌈m/2⌉ - 1
- 所有叶子始终在同一层（因为分裂/合并只影响兄弟节点，不改变层数）

**这是 B-Tree 最优雅的地方：平衡是结构定义的自然结果，不需要额外的平衡算法。**

---

### 2. 磁盘 I/O 的微观分析

#### 为什么磁盘随机 I/O 这么慢？

| 操作 | 延迟 | 比例 |
|------|------|------|
| L1 cache | 0.5 ns | 1x |
| L2 cache | 7 ns | 14x |
| RAM | 100 ns | 200x |
| SSD random read | 150,000 ns (150μs) | 300,000x |
| HDD random read | 10,000,000 ns (10ms) | 20,000,000x |

一次磁盘随机 I/O 的时间 ≈ 几百万次 CPU 指令。所以**减少一次 I/O 比优化节点内算法重要得多**。

#### 为什么节点大小要对齐磁盘页？

磁盘的最小读写单位是**页**（通常 4KB）。一次 I/O 读取一整页，无论你只读 1 字节还是 4KB，成本几乎一样。

因此 B-Tree 的节点大小通常设为 4KB 或 16KB（对齐 InnoDB 的页大小），让**一次 I/O 读入尽可能多的 key**，最大化每次 I/O 的信息量。

---

### 3. B+Tree 的深入分析

#### 内部节点能存多少 key？

假设 key = 8 bytes（如 bigint），指针 = 8 bytes，每个内部节点条目 = 16 bytes。

- 4KB 页：4096 / 16 = 256 个条目 → 分支因子 m = 256
- 16KB 页（InnoDB）：16384 / 16 = 1024 个条目 → 分支因子 m = 1024

N=10⁹ 时：
- m=256：树高 ≤ 1 + log₁₂₈(5×10⁸) ≈ 1 + 4.1 ≈ 5
- m=1024：树高 ≤ 1 + log₅₁₂(5×10⁸) ≈ 1 + 3.2 ≈ 4

**4-5 次 I/O 就能从 10 亿条记录中找到任意一条。**

#### 叶子节点的链表结构

B+Tree 的叶子节点通过**双向链表**相连，这是范围查询高效的关键：

```
SELECT * FROM users WHERE age BETWEEN 20 AND 30;
```

1. 从根节点沿路径找到 age=20 所在的叶子节点（4-5 次 I/O）
2. 在叶子节点内二分找到 age=20 的位置
3. 沿叶子链表顺序扫描，直到 age > 30

**整个过程只有第一步是随机 I/O，后续扫描都是顺序 I/O**（磁盘顺序读远快于随机读）。

---

### 4. 与其他索引结构的对比

| 结构 | 树高 (N=10⁹) | 范围查询 | 写放大 | 适用场景 |
|------|-------------|---------|--------|---------|
| B+Tree | 4-5 | ✅ 高效 | 中等 | 通用 OLTP |
| LSM-Tree | N/A | ❌ 慢 | 高 | 写密集 |
| Hash Index | 1 | ❌ 不支持 | 低 | 等值查询 |
| BST/AVL | 30 | ✅ 中序 | 低 | 内存数据 |

**B+Tree 是"均衡器"**：在等值查询、范围查询、写入性能之间取得了最好的平衡。

---

### 5. 关于 B-Tree 的名字

> About the name B-Tree, it's interesting to find out what's the meaning of 'B'. Actually one of B-Tree's authors, McCreight explained, the name is decided at lunch time. Maybe it means Boeing (where Bayer and McCreight worked then), or means Balanced or Bayer or other things. What Rudy likes to say is, **"the more you think about what the B in B-Tree means, the better you understand B-Trees!"**

- 参考：https://stackoverflow.com/questions/2263858/does-anybody-know-how-b-tree-got-its-name

{{< /collapse >}}
