---
title: leetcode刷题记录 
date: 2026-04-20T11:47:43Z
tags: [算法 & 数据结构]
series: []
featured: true
description: "图论 DFS/BFS 刷题记录，包含 dfs 回溯框架和 bfs 层序遍历模板"
draft: true
---
这是摘要
​
<!--more-->
​
这是内容
## 动态规划

## 图论进阶

## 并查集

## 回溯

## bfs & dfs

## 二叉树

## 堆
堆的一个重要价值在于找到topK元素；
> 数组中的topK元素
很明显保存最大的K个元素，但是为了过滤其他元素，需要和堆顶最小元素对比，使用的就是小根堆。

> 对 priority_queue<T, Container, Compare>，Compare(a, b) == true 表示 a 应该排在 b 后面。
```c++
priority_queue<int, vector<int>, greater<int>> pq;

for (const auto num : nums) {
    if (pq.size() < k){
        pq.push(num);
    } else if (num > pq.top()) {
        pq.pop();
        pq.push(num);
    }
return pq.top();
}
```

> 出现频率最高的前K个元素
很明显需要hash统计出现频率，然后用堆维护频率前topK,那么关键的就是如何定义pair的cmp以及传入priority_queue。
```c++
priority_queue<T, Container, Compare>,Compare is a type
因此要么传入struct {operator()}这种要么传入decltyp(cmp);

map freq init

bool cmp = [](auto& a, auto& b) {
    return a.second > b.second;//这里cmp传入优先队列理解为优先级，当a越大时优先级越低，对应小根堆
};
priority_queue<pair<int, int>, vector<pari<int, int>>, decltype(cmp)> pq;

for(){...}

while(!pq.empty()){ans.push_back(pq.top().first);pq.pop();}
```

## 哈希
Hash 最核心的能力：

> 给我一个值，我能很快判断“之前有没有见过它”，以及找到它对应的信息。

两数之和：a[i] + a[j] == target;(i < j)遍历数组到a[j]时，需要有一个数据结构判断之前是否存在target - a[j]->hash
字母异位词分组:注意到排序后映射到同一个字符串了，直接加入对应位置即可（这里hash记录的是排序字符串和其在结果数组中的位置）
和为 K 的子数组:前缀和+哈希表

> 最长连续序列
如果排序+扫描，时间复杂度O(nlogn)
使用set维护所有元素，如果对于每个元素都往后判断，有大量重复计算，所以要从序列起点判断；观察序列起点发现不存在前一个元素

```c++
int longestConsecutive(vector<int>& nums) {
    unordered_set<int> st(nums.begin(), nums.end());
    int ans = 0;
    for (int it : st) {
        if (!st.count(it - 1)) {//序列起点
            int seq_len = 0;
            while (st.count(it + seq_len)) {
                seq_len++;
            }
            ans = max(ans, seq_len);
        }
    }
    return ans;
}
```

请你设计并实现一个满足  LRU (最近最少使用) 缓存 约束的数据结构。
实现 LRUCache 类：
LRUCache(int capacity) 以 正整数 作为容量 capacity 初始化 LRU 缓存
int get(int key) 如果关键字 key 存在于缓存中，则返回关键字的值，否则返回 -1 。
void put(int key, int value) 如果关键字 key 已经存在，则变更其数据值 value ；如果不存在，则向缓存中插入该组 key-value 。如果插入操作导致关键字数量超过 capacity ，则应该 逐出 最久未使用的关键字。
函数 get 和 put 必须以 O(1) 的平均时间复杂度运行。

```c++
class LRUCache {
public:
    LRUCache(int capacity) {
        cap = capacity;
    }

    int get(int key) {
        if (!mp.count(key)) {
            return -1;
        }
        auto it = mp[key].second;
        cache.splice(cache.begin(), cache, it);
        return mp[key].first;
    }

    void put(int key, int value) {
        if (mp.count(key)) {
            mp[key].first = value;
            auto it = mp[key].second;
            cache.splice(cache.begin(), cache, it);
            return;
        }

        cache.push_front(key);
        mp[key] = {value, cache.begin()};
        if (cache.size() > cap) {
            mp.erase(cache.back());
            cache.pop_back();
        }
    }
private:
    int cap;
    unordered_map<int, pair<int, list<int>::iterator>> mp;
    list<int> cache;//规定放在前面的是最近使用的
}
```
## 单调栈
```c++

```

## 二分查找
二分：利用单调性，每次判断中间位置，排除一半不可能的搜索空间，把 O(n) 降到 O(log n)。
```c++
int l = 0, r = n - 1;

while (l <= r) {
    int mid = l + (r - l) / 2;

    if (nums[mid] == target) {
        return mid;
    } else if (nums[mid] < target) {
        l = mid + 1;
    } else {
        r = mid - 1;
    }
}

return -1;
```
> 变体:lower_bound和upper_bound
```c++
template<class ForwardIt, class T>
ForwardIt lower_bound(ForwardIt first, ForwardIt last, const T& target) {
    while (first != last) {
        auto mid = first + (last - first) / 2;

        if (*mid < target) {
            first = mid + 1;
        } else {
            last = mid;
        }
    }

    return first;
}

template<class ForwardIt, class T>
ForwardIt upper_bound(ForwardIt first, ForwardIt last, const T& target) {
    while (first != last) {
        auto mid = first + (last - first) / 2;

        if (*mid <= target) {
            first = mid + 1;
        } else {
            last = mid;
        }
    }

    return first;
}
```

## 双指针
判别：题目存在两个“位置”，并且当其中一个位置移动后，另一个位置不需要回退。
双指针一般可以将暴力解法的 $O(n^2)$ 降低至 $O(n)$。
- 对撞指针
- 快慢指针
- 同向双指针（滑动窗口）
- 多数组/区间的双指针

```c++
//对撞双指针一般要求数组有序，利用单调性，找两数字之和，两数之差，判断回文；
//核心是利用有序性一次排除一批不合法状态，剪枝
int left = 0, right = nums.size() - 1;
while (left < right) {
    //注意如果condition传递到内部，外部的条件不要忘记
    if (condition) {
        left++;
    } else {
        right--;
    }
}
```
> 盛最多水的容器
思考：面积 = （r - l）* min (h[l], h[r]);最宽的情况就是从两边出发，向中间移动，宽度变小，想要面积变大，只能高度增加；高度由较小的那个决定，因此需要较小的移动；
```c++
...
while (left < right) {
    //update ans every loop
    ans = max(...)
    if (h[left] < h[right]) {
        left++;
    } else {
        right--;
    }
}
```

快慢指针：两个指针同向，但是速度不同或者承担不同职责。例如删除数组中的重复元素、判断链表有没有环
```c++
// 删除有序数组中的重复项
void removeDuplicates(vector<int>& nums) {
    int n = nums.size();
    int slow = 1;
    for (int fast = 1; fast < n; ++fast) {
        if (nums[fast] == nums[fast - 1]) {
            fast++;
        } else {
            nums[slow++] = nums[fast++];
        }
    }
}
```

滑动窗口：略
区间合并/多序列
> 两个有序数组找交集。
```c++

```
三数之和
```c++
//三数之和还需要叠加一层循环
sort(nums.begin(), nums.end());
for (int i = 0; i < n - 2; ++i) {
    //跳过重复状态
    if (i > 0 && nums[i] == nums[i - 1]) {
        continue;
    }
    int left = i + 1, right = n - 1;
    while (left < right) {
        long long sum = nums[i] + nums[left] + nums[right];
        if (sum == 0) {
            left++,right--;
        } else if (sum < 0) {
            left++;
        } else {
            right--;
        }
    }
}
```

>接雨水
当前位置能接多少雨水 = min(leftMax[i], rightMax[i]) - height[i]
但是没必要计算出leftMax和rightMax数组，在l位置时，假设leftMax< rightMax,左边向前走
## 前缀和
判别特征：连续区间+和区间和有关（或者可以转换为区间和）
前缀和用于解决连续区间和问题，通过预处理数组，将时间复杂度从 O(n^2) 降低到 O(n)。
举例：（多次）区间求和/和为k,数组可正可负（前缀和+hash）/和%k==0(前缀和+hash)、 有负数 + 求和满足大小限制（前缀和+单调队列）

区分：非负数组 + 和的大小限制、非负数组 + 求最长/最短（滑动窗口）
### 板子
```c++
//初始化prefix[0] = 0的目的是为了避免处理边界情况，变体如果有记录也别忘记初始化
//prefix[i] = nums[0] + nums[1] + ... + nums[i-1];
vector<long long> prefix(n + 1, 0);

for (int i = 0; i < n; i++) {
    prefix[i + 1] = prefix[i] + nums[i];
}

// 查询 [l, r]
long long sum = prefix[r + 1] - prefix[l];
```
### 前缀和+hash
区间和 = K
    ↓
pre[j] - pre[i] = K
    ↓
pre[i] = pre[j] - K
    ↓
哈希表
> 和为 K 的子数组
思路：子数组和 = prefix[j] - prefix[i - 1] = K;(i <= j)有prefix[j] - K = prefix[i - 1];遍历到位置j时，如果有可以记录prefix[i - 1]出现的次数的就好了，使用hash

```c++
unordered_map<long long, int> sumTable;
sumTable[0] = 1;//不要忘记初始化

for (...) {
    curSum += num;
    ans += sumTable[curSum - K];
    sumTable[curSum]++;
}
```
#### 前缀和+模运算
(prefix[j] - prefix[i - 1]) % K = 0 -> prefix[j] % K = prefix[i - 1] % K 

```c++
// 注意 C++ 取模的特殊性，当被除数为负数时取模结果为负数，需要纠正为正数
```

> 525. 连续数组
重新建模，0看作-1,求和为0的最长子数组数量,然后需要用一个hash表记录出现和最早的位置
```c++
class Solution {
public:
    int findMaxLength(vector<int>& nums) {
        //由于前缀和的范围是 [-n, n]，可以用数组代替哈希表
        unordered_map<int, int> mp;  // 前缀和 -> 第一次出现的位置
        mp[0] = -1;  // 前缀和为0在位置-1（数组开始前）
        
        int sum = 0;
        int maxLen = 0;
        
        for (int i = 0; i < nums.size(); i++) {
            sum += (nums[i] == 1) ? 1 : -1;
            
            if (mp.find(sum) != mp.end()) {
                maxLen = max(maxLen, i - mp[sum]);
            } else {
                mp[sum] = i;  // 只记录第一次出现的位置
            }
        }
        
        return maxLen;
    }
};
```
这道题无法使用滑动窗口，因为不满足单调性

> 304. 二维区域和检索 - 矩阵不可变；【二维前缀和】
```c++
class NumMatrix {
private:
    vector<vector<int>> pre;  // 前缀和数组，大小为 (m+1) × (n+1)

public:
    NumMatrix(vector<vector<int>>& matrix) {
        int m = matrix.size();
        int n = matrix[0].size();
        
        // 初始化前缀和数组，多一行一列便于处理边界
        pre.assign(m + 1, vector<int>(n + 1, 0));
        
        // 构建前缀和
        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                pre[i][j] = pre[i-1][j] + pre[i][j-1] 
                          - pre[i-1][j-1] + matrix[i-1][j-1];
            }
        }
    }
    
    int sumRegion(int row1, int col1, int row2, int col2) {
        return pre[row2+1][col2+1] - pre[row1][col2+1] 
             - pre[row2+1][col1] + pre[row1][col1];
    }
};
```

> 前缀和+单调性
如果数组不存在负数，前缀和数组很明显是升序单调增比如长度最小的子数组可以用前缀和+二分去做,时间复杂度O(nlogn),空间复杂度O(n);但是最好还是滑动窗口去做

> 前缀和+单调队列
LeetCode 862. 和至少为 K 的最短子数组
题目描述
给你一个整数数组 nums 和一个整数 k，找出 nums 中和至少为 k 的最短非空子数组，并返回该子数组的长度。如果不存在这样的子数组，返回 -1。

> 
## 滑动窗口
最难的部分肯定还是滑动窗口和字符串、hash 等结构结合的组合问题。

### 定长窗口
定长窗口只需一个指针（右边界 `r`）就能确定位置，左边界固定为 `r - k + 1`。这里把它改造成和不定长窗口类似的「进入 → 更新 → 移出」结构，方便统一记忆。
```c++
for (int r = 0; r < n; ++r) {
    // ① 进入：加入 nums[r]，更新统计量（sum / freq / cnt ...）
    if (r < k - 1) continue;   // 窗口还没攒够 k 个元素

    // ② 更新答案：此时窗口为 [r - k + 1, r]

    // ③ 移出：去掉左边界 nums[r - k + 1] 的贡献（右边界由 for 推进）
}
```
比如 LeetCode 1456 定长子串中元音的最大数目。

定长窗口问题的难点不在于滑动框架，而在于两点：

- 窗口统计量的定义——除了统计量本身，还需要什么数据结构或中间量来维护窗口信息？
- 统计量的更新逻辑——加入 / 移出元素时，统计量如何变化？

常见统计量与数据结构的对应关系：

| 统计量类型 | 数据结构 | 示例 |
| --- | --- | --- |
| 和 / 积 / 异或 | 单个变量 | 最大子数组和 |
| 最大 / 最小值 | 单调队列（deque） | 滑动窗口最大值 |
| 字符频次 | 数组[26] / HashMap | 字母异位词 |
| 不同元素个数 | HashMap + 计数变量 | 含不同整数的子数组 |

下面几道变体只保留相对上面框架发生变化的逻辑。

> lc239. 滑动窗口最大值（区间最值）

单调队列维护统计量，滑动窗口提供移动动力：队列里存下标、保证对应值递减，队头即窗口最大值。
```c++
// ① 进入：弹出队尾所有 <= nums[r] 的下标，保持队列递减
while (!dq.empty() && nums[dq.back()] <= nums[r]) dq.pop_back();
dq.push_back(r);

// ② 更新答案：队头下标对应窗口最大值
ans = max(ans, nums[dq.front()]);

// ③ 移出：越界的左边界一定在队头（按下标判断）
if (dq.front() == r - k + 1) dq.pop_front();
```

> LeetCode 438. 找到字符串中所有字母异位词（频次 + 匹配计数）

维护 `match`（频次已对齐的字母数）与 `win_freq`（窗口内各字母频次）。窗口长度固定为 `p` 的长度，所以不必关心非法状态——`match == require`（`p` 的不同字母数）即命中一个异位词。
```c++
// ① 进入 s[r]：频次 +1，若恰好等于目标频次，match +1
int ir = s[r] - 'a';
if (++win_freq[ir] == p_freq[ir]) ++match;

// ② 更新答案：match 等于 p 的不同字母数 require 时命中
if (match == require) ans.push_back(r - k + 1);

// ③ 移出 s[r-k+1]：若移出前恰好对齐则 match -1，再把频次 -1
int il = s[r - k + 1] - 'a';
if (win_freq[il]-- == p_freq[il]) --match;
```

> 2841. 几乎唯一子数组的最大和

用 HashMap 维护频次，统计量为窗口和 `win_sum` 与不同元素个数（即 `win_freq.size()`）；不同元素数 ≥ m 才算「几乎唯一」。
```c++
// ① 进入 nums[r]：更新窗口和与频次
win_sum += nums[r];
win_freq[nums[r]]++;

// ② 更新答案：不同元素数 >= m 时才是几乎唯一子数组
if ((int)win_freq.size() >= m) ans = max(ans, win_sum);

// ③ 移出 nums[r-k+1]：减和、减频次，频次归零则删除键
int l = r - k + 1;
win_sum -= nums[l];
if (--win_freq[nums[l]] == 0) win_freq.erase(nums[l]);
```

### 不定长窗口
与定长窗口不同，窗口长度是动态变化的，通过移动左右指针来寻找满足（或不满足）某个条件的最优区间。
```c++
int left = 0;
for (int right = 0; right < n; ++right) {
    add(nums[right]);                    // 1. 加入右边界
    
    while (shrink_condition) {           // 2. 收缩左边界
        // [可选] 更新答案（最短窗口）
        remove(nums[left]);
        left++;
    }
    
    update_answer();                     // 3. 更新答案（最长/计数）
}
// int slidingWindow(vector<int>& nums, int k) {
//     int left = 0, ans = 0;  // 或 INT_MAX / INT_MIN
//     // 定义窗口统计量：sum, freq, matched, distinct...
    
//     for (int right = 0; right < n; ++right) {
//         // ★ 填入：加入 right 的更新逻辑
//         add(nums[right]);
        
//         // ★ 填入：收缩条件和移除逻辑
//         while (/* 收缩条件 */) {
//             // ★ 填入：最短窗口更新（如需要）
//             remove(nums[left]);
//             left++;
//         }
        
//         // ★ 填入：最长/计数更新（如需要）
//         ans = max(ans, right - left + 1);  // 或 ans += right - left + 1
//     }
    
//     return ans;
// }
```
- 移动 right 加入新元素；

- 根据目标（长/短）决定 while 条件，移动 left 收缩；

- 找最长在 while 后记答案，找最短在 while 里记答案。

> 例子1. 无重复字符的最长子串
```c++
vector<int> freq(128, 0);//窗口统计量，作用是判断是否出现重复
for(){
    freq[s[r]]++;
    while (freq[s[r]] > 1) {
        freq[s[l]]--;
        l++;
    }
    ans = max(...)
}
```
> 最大连续1的个数 III
        //最长连续子数组问题，前缀和或者是滑动窗口
        //翻转k个0，实际上是找到连续子数组中0的数量<=k
        //是否满足单调性，当窗口中0个数大于k，那么更大的窗口也是不合法的
        //统计量是0的个数


## 贪心问题
贪心问题的性质是“局部最优解 => 全局最优解”，但是根据这一性质还是很难判断某一问题能否用贪心算法解决。

leetcode 45/55 跳跃游戏1/2
```c++
//题目限定了一定可以到达，所以可以直接遍历到最后减少一次判断
int jump(vector<int>& nums) {
    int n = nums.size();
    int step = 0;
    int maxLast = 0, maxMost = 0;
    for (int i = 0; i < n - 1; i++) {
        maxMost = max(maxMost, i + nums[i]);
        if (i == maxLast) {
            maxLast = maxMost;
            step++;
        }
    }
    return step;
}
```

> 

# 图论

## 理论知识

> 存储结构

> 有向图/无向图

> 连通分量

## dfs 与图
dfs的理论思想就是先沿着一个方向探索，到头或碰壁后返回上一个节点探索其他方向，更换方向的过程需要撤销原先的探索（回溯）
因此代码框架实际就是回溯的结构
```c++ 
void dfs(参数) {
    if (终止条件) {
        存放结果;
        return;
    }

    for (选择：本节点所连接的其他节点) {
        处理节点;
        dfs(图，选择的节点); // 递归
        回溯，撤销处理结果
    }
}
```
例题，求start->end的所有路径


## bfs 与图

## 排序
### 快速排序（链表）
- 阿里polardb二面
```c++
ListNode* qSort(ListNode* head) {
    if (!head || !head->next) {
        return head;
    }
    int pivot = head->val;
    ListNode* left = nullptr;
    ListNode* leftTail = nullptr;
    ListNode* right = nullptr;
    ListNode* rightTail = nullptr;

    ListNode* cur = head->next;
    while(cur) {
        ListNode* next = cur->next;
        cur->next = nullptr;
        if (cur->val < pivot) {
            if (!left) {
                left = leftTail = cur;
            } else {
                leftTail->next = cur;
                leftTail = cur;
            }
        } else {
            if (!right) {
                right = rightTail = cur;
            } else {
                rightTail->next = cur;
                rightTail = cur;
            }
        }
        cur = next;
    }
    left = qSort(left);
    right = qSort(right);
    //合并
    // left -> pivot -> right
    if (left) {
        ListNode* tail = left;
        while (tail->next) {
            tail = tail->next;
        }
        tail->next = head;
    } else {
        left = head;
    }

    head->next = right;

    return left;
}
```

## 二分查找
```c++

```
## 高精度计算
### 高精度加法
```c++
string add(string num1, string num2) {
    int i = num1.size() - 1, j = num2.size() - 1, carry = 0;
    string ans;
    while (i >= 0 || j >= 0 || carry) {
        int x = i >= 0 ? num1.at(i) - '0' : 0;
        int y = j >= 0 ? num2.at(j) - '0' : 0;
        int sum = x + y + carry;
        ans += char(sum % 10 + '0');
        carry = sum / 10;
        i--;
        j--;
    }
    reverse(ans.begin(), ans.end());
    return ans;
}
```
### 高精度乘法
43. 字符串相乘
```c++
    string multiply(string num1, string num2) {
        //乘法有两个操作 加法和乘法，加法又分为竖加和进位，优化思路是分开做两部分操作，第一部分只处理乘法和竖加，第二部分处理进位加法；使用一个数组存起来乘法的中间结果
        //加法具有交换律，最后一起做就行
        //而且可以推导出乘法的位数 < m + n + 1
        //加上正负号处理
        //处理一下0
        if (num1 == "0" || num2 == "0") {
            return "0";
        }
        bool neg = false;
        int start1 = 0, start2 = 0;
        if (num1.at(0) == '-') {
            neg = !neg;
            start1 = 1;
        }
        if (num2.at(0) == '-') {
            neg = !neg;
            start2 = 1;
        }
        int m = num1.size(), n = num2.size();
        vector<int> ansArr(m + n, 0);
        for (int i = m - 1; i >= start1; --i) {
            int x = num1.at(i) - '0';
            for (int j = n - 1; j >= start2; --j) {
                int y = num2.at(j) - '0';
                ansArr[i + j + 1] += x * y;
            }
        }
        //处理进位加法
        for (int i = m + n - 1; i > 0; i--) {
            ansArr[i - 1] += ansArr[i] / 10;
            ansArr[i] %= 10;
        }

        //拼接出答案
        string ans;
        if (neg) {
            ans = '-';
        }
        int index = ansArr[0] == 0 ? 1 : 0;
        while (index < m + n) {
            ans += char(ansArr[index] + '0');
            index++;
        }
        return ans;
    }
```
