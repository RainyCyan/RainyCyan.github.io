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
如何识别动态规划问题，

## 图论进阶

## 并查集

## 回溯

## bfs & dfs

## 递归与回溯
回溯的本质就是在搜索一棵决策树。比如1,2,3的全排列

递归解决的是：规模变小之后，重复解决同一个问题。
```c++
void dfs(int x) {
    if (x == n /*end condition*/) {
        return;
    }

    dfs(x + 1);//缩小状态空间
}
```
回溯是在递归基础上增加：

做选择 → 递归 → 撤销选择
```c++
void backtrack(状态参数) {

    // 如果当前状态是一个答案
    if (满足答案条件) {
        ans.push_back(path);
    }
//这里的关键就在于判断答案是否需要和结束条件合并
    // 如果需要结束递归
    if (满足结束条件) {
        return;
    }

    // 枚举选择
    for (每一个选择) {
        // 做选择
        path.push_back(...);
        // 递归
        backtrack(下一状态);
        // 撤销选择
        path.pop_back();
    }
}
```

>例如子集，所有中间状态都是答案，因此都需要push_back

```c++
//子集2，去重版
vector<int> path;
vector<vector<int>> ans;
void backtracking(vector<int>& nums, int start) {
    int n = nums.size();
    ans.push_back(path);
    if (start == n) {
        return;
    }

    for (int i = start; i < n; ++i) {
        if (i > start && nums[i] == nums[i - 1]) {
            continue;
        }
        path.push_back(nums[i]);
        backtracking(nums, i + 1);
        path.pop_back();
    }
}

void subsets(vector<int>& nums) {
    //先sort
    sort(nums.begin(), nums.end());
    backtracking(nums, 0);
}
```
> 组合问题，path.size() == k时，就是答案，终止递归

> 组合总和问题1，可以重复选择，因此递归就从当前元素开始(不从0开始是为了避免状态重复)
```c++
combination1: 无重复元素+重复选择；进入下一层时，从当前元素开始；结束条件target <0

2.去重复：if (i > start && candidates[i] == candidates[i - 1]) {
    continue;
}

3. k个元素，if (path.size() == k) {
    ans.push_back(path);
    return;
}

4.排列总和，递归+动态规划
```

> 排列问题
组合是“从后面继续选”，排列是“每一层都可以选任意没用过的元素”。因此我们定义排列 = 每一层都可以从所有元素中选，但一个元素只能用一次。为了避免重复，我们需要记录已经用过的元素，

```c++
全排列，vecotr<bool> used
去重复，if (i > 0 && nums[i] == nums[i - 1] && !used[i - 1]) {
    continue;
}
```

> 字符串/分割类回溯
- LC17 电话号码的字母组合
```c++
void backtrack(string& digits) {
    if (path.size() == digits.size()) {
        ans.push_back(path);
        return;
    }

    string letters = mp[digits[path.size()]];

    for (char c : letters) {
        path.push_back(c);

        backtrack(digits);

        path.pop_back();
    }
}
```
## 二叉树
一个经典的二叉树节点定义如下：
```c++
struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode *right;

    TreeNode() : val(0), left(nullptr), right(nullptr) {}
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
    TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
};
```
观察得知，二叉树有一个很优雅的递归结构：`一棵树入口 = 根节点(root) +左右子树`

> 迭代遍历二叉树(染色法)
```c++
vector<int> traverse(TreeNode* root, int type) {
    vector<int> result;
    if (root == nullptr) {
        return result;
    }

    stack<pair<TreeNode*, bool>> stk;
    stk.push({root, false});

    while (!stk.empty()) {
        auto [node, visited] = stk.top();
        stk.pop();

        if (node == nullptr) {
            continue;
        }

        if (visited) {
            result.push_back(node->val);
        } else {
            if (type == 0) {
                //前序根-左-右，对应入栈顺序为：右-左-根
                stk.push({node->right, false});//右
                stk.push({node->left, false});//左
                stk.push({node->val, true});//根
            } else if (type == 1) {
                //中序左-根-右，对应入栈顺序为：右-根-左
                stk.push({node->right, false});//右
                stk.push({node->val, true});//根
                stk.push({node->left, false});//左
            } else {
                //后序左-右-根，对应入栈顺序为：根-右-左
                stk.push({node->val, true});//根
                stk.push({node->right, false});//右
                stk.push({node->left, false});//左
            }
        }
    }
}
```
> 递归的结构
```c++
void recur() {
    if (end_condition) {
        return;
    }
    recur();
    return;
}
```
>

因此对于二叉树的问题可以问自己：问题 A：空节点怎么办？问题 B：当前节点要干什么？问题 C：左右子树要不要递归？问题 D：当前节点的操作是在子树之前还是之后？
比如对于树的深度
```c++
int maxDepth(TreeNode* root) {
    if (!root) {//空节点返回0
        return 0;
    }
    //当前节点深度+1
    //计算左右子树深度，取较大者，当前节点操作在后面
    return 1 + max(maxDepth(root->left), maxDepth(root->right));
}
```
又比如翻转二叉树
```c++
TreeNode* invertTree(TreeNode* root) {
    if (!root) {
        return nullptr;
    }
    //交换左右子树
    swap(root->left, root->right);
    //递归翻转左右子树
    invertTree(root->left);
    invertTree(root->right);
    return root;
}
```
再比如判断对称二叉树，实际需要对两颗子树分析判断
包含结束情况：1.两个空节点，true;2.一空一非空，false;3.不想等，false;4.递归判断子树需要判断(a->left,b->right)和(a->right,b->left)
```c++
bool isSymmetric(TreeNode* a, TreeNode* b) {
    if (!a && !b) {
        return true;
    }
    if (!a || !b) {
        return false;
    }
    return a->val == b->val && isSymmetric(a->left, b->right) && isSymmetric(a->right, b->left);
}
bool isSymmetric(TreeNode* root) {
    if (!root) {
        return true;
    }
    return isSymmetric(root->left, root->right);
}
```
因此对于树的遍历(DFS)可以根据对根节点的处理顺序分为前中后序

前面都是深度遍历dfs,下面来说层序遍历
```c++
void bfs(TreeNode* root) {
    if (!root) return;

    queue<TreeNode*> q;
    q.push(root);

    while (!q.empty()) {
        int size = q.size();  // 当前层节点数量

        for (int i = 0; i < size; i++) {
            TreeNode* node = q.front();
            q.pop();

            // ===== 处理当前节点 =====


            // ===== 加入下一层 =====
            if (node->left) {
                q.push(node->left);
            }

            if (node->right) {
                q.push(node->right);
            }
        }
    }
}
```
> LC199 二叉树的右视图
这道题即可以用层序bfs也可以用dfs
bfs的话就是经典模板+输出控制，dfs实际先处理右子树，再处理左子树，每层访问的第一个元素进入答案，需要一个参数标记层数

> LCR045. 找树左下角的值
bfs每层第一个元素更新答案；dfs需要一个全局参数标记当前访问最大深度，然后第一次进入新层就更新答案

#### 二叉树路径和问题

> 路径和1/2/3
```c++
hasPathSum()：最简单，确定dfs含义是判断是否存在路径，当前节点处理直接减去cur->val,递归返回左右子树结果取或

```
pathSum三是任意节点开始，任意子节点结束，想到用前缀和+hash(当然不能忘记递归回溯结构)

迭代方式前中后序实现

- 返回值设计
> LC110 平衡二叉树
平衡二叉树指的是树的高度相差不超过1，因此想到左右子树高度->当前节点高度，有`height(node) = max(node->left, node->right) + 1`,但是如果每个节点都判断的话，会有很多重复计算；返回值可以设计成如果不平衡 -1,我们让height完成了1.判断平衡2.如果平衡，返回高度两个功能
```c++
int height(TreeNode* root) {
    if (!root) {
        return 0;
    }

    int leftHeight = height(root->left);
    if (leftHeight == -1) {
        return -1;
    }
    int rightHeight = height(root->right);
    if (rightHeight == -1) {
        return -1;
    }
    if (abs(leftHeight - rightHeight) > 1) {
        return -1;
    }
    return max(leftHeight, rightHeight) + 1;
}

bool isBalanced(TreeNode* root) {
    return height(root) != -1;
}
```

LC124 二叉树中的最大路径和
仍然可以用通过所有根节点的最大路径和更新答案，但是需要处理负数

```c++
int ans = INT_MIN;
int dfs(TreeNode* root) {
    if (!root) {
        return 0;
    }
    // 返回的是从当前节点向下的最大路径和
    int leftMax = dfs(root->left);
    int rightMax = dfs(root->right);

    int cur = root->val;
    cur += leftMax > 0 ? leftMax : 0;
    cur += rightMax > 0 ? rightMax : 0;

    ans = max(ans, cur);

    return root->val + max(0, max())
}

int maxPathSum(TreeNode* root) {
    dfs(root);
    return ans;
}
```

#### 二叉搜索树BST
二叉搜索树有以下性质：
- 左< 根< 右:左子树所有节点 < 根节点 < 右子树所有节点,注意是整个子树
- BST 中序遍历得到严格递增序列。
- BST 查找可以利用大小关系排除一半子树,BST插入实际也是利用搜索
- 最小值一直向左，最大值一直向右

```c++
class BST {
private:
    TreeNode* root;
public:
    TreeNode* search(int target) {
        TreeNode* cur = root;
        while (cur) {
            if (cur->val == target) {
                return cur;
            }else if (cur->val > target) {
                cur = cur->left;
            } else {
                cur = cur->right;
            }
        }
        return nullptr;
    }

    void insert(int val) {

    }
}
```
> LC98 验证二叉搜索树
利用性质1，当前节点需要判断是否合法需要传入范围[low,high]
```c++
bool dfs(TreeNode* root, long long low, long long high) {
    if (!root) {
        return true;
    }

    if (root->val <= low || root->val >= high) {
        return false;
    }

    return dfs(root->left, low, root->val) && dfs(root->right, root->val, high);
}
bool isValidBST(TreeNode* root) {
    return dfs(root, LLONG_MIN, LLONG_MAX);
}
```
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

> 数据流的中位数
动态插入 + 动态查询中间值。如果维护有序数组，插入不是O(1),注意到中位数把数组切割成了两个近似相等的部分，我们只关心中位数左边有哪些元素，以及右边有哪些元素。

左半边我们需要知道最大的元素是多少？右边我们需要知道最小的元素是多少，因此两个堆维护。约束条件就是两个堆的大小差不能超过1。

```c++
class MedianFinder {
public:
    MedianFinder() {
        
    }
    
    void addNum(int num) {
        if (maxHeap.empty() || num < maxHeap.top()) {
            maxHeap.push(num);
        } else {
            minHeap.push(num);
        }

        // 2. 保证 maxHeap 比 minHeap 多 0 或 1 个
        if (maxHeap.size() < minHeap.size()) {
            maxHeap.push(minHeap.top());
            minHeap.pop();
        } 
        else if (maxHeap.size() > minHeap.size() + 1) {
            minHeap.push(maxHeap.top());
            maxHeap.pop();
        }
    }
    
    double findMedian() {
        if (maxHeap.size() == minHeap.size()) {
            return (maxHeap.top() + minHeap.top()) / 2.0;
        }

        return maxHeap.top();
    }
private:
    priority_queue<int> maxHeap;//左半边
    priority_queue<int, vector<int>, greater<int>> minHeap;
};
```

> 合并K个有序链表
注意到每个链表都已经排好序了，所以每个链表头就是最小元素，所有链表头最小就是全局最小，那不断取链表头就好了；观察我们实际做的就是在k个元素中重复找最小的，很明显用堆

```c++
    ListNode* mergeKLists(vector<ListNode*>& lists) {
        //找最小，最小堆
        auto cmp = [](const auto&a, const auto &b) {
            return a->val > b->val;
        };
        priority_queue<ListNode*, vector<ListNode*>, decltype(cmp)>  pq;
        
        for (ListNode *head : lists) {
            if (head){
                pq.push(head);
            }
        }

        ListNode dummy(0);
        ListNode *tail = &dummy;
        while (!pq.empty()) {
            auto cur = pq.top();
            tail->next = cur;
            tail = tail->next;
            pq.pop();
            if (cur->next) {
                pq.push(cur->next);
            }
        }
        return dummy.next;
    }
```

### 拓展：建堆和堆排序



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
