# Sorting

*December 14, 2024*

# Lecture 3: Sorting

## Set Interface
### Basic Operations

1. 容器操作
    - `build(X)`：从可迭代对象X构建集合
    - `len()`：返回存储项目数量

2. 静态操作
    - `find(k)`：返回键为k的存储项

3. 动态操作
    - `insert(x)`：添加x到集合（如果已存在相同键则替换）
    - `delete(k)`：移除并返回键为k的存储项

4. 顺序操作
    - `iter_ord()`：按键顺序逐个返回存储项
    - `find_min()`：返回最小键的存储项
    - `find_max()`：返回最大键的存储项
    - `find_next(k)`：返回大于k的最小键存储项
    - `find_prev(k)`：返回小于k的最大键存储项

### Array Based Implemment

1. 无序(unordered)数组（无序存储）
    - 实现了基本的集合功能但效率不高
    - 所有操作时间复杂度均为$O(n)$

2. 有序(sorted)数组优势
    - 支持快速查找最大/最小值（在数组首尾）
    - 通过二分查找加速查找操作：$O(\log n)$
    - 但是构建操作需$O(n\log n)$时间

|            Operations            | Array  | Sorted Array  |
| :------------------------------: | :----: | :-----------: |
|            `build(X)`            | $O(n)$ | $O(n \log n)$ |
|            `find(k)`             | $O(n)$ |  $O(\log n)$  |
|    `insert(x)`<br>`delete(k)`    | $O(n)$ |    $O(n)$     |
|   `find_min()`<br>`find_max()`   | $O(n)$ |    $O(1)$     |
| `find_prev(k)`<br>`find_next(k)` | $O(n)$ |  $O(\log n)$  |

## Sorting

1. 排序问题定义
    - 输入：n个数字的静态数组A
    - 输出：A的一个有序排列B
    - 要求：
        - B是A的排列(**permutation**)（相同元素不同顺序）
        - B有序(**sorted**)：对所有$i \in \{1,...,n\}$，有$B[i-1] \leq B[i]$
    - 示例：`[8, 2, 4, 9, 3] → [2, 3, 4, 8, 9]`

2. 排序方法分类
    - 破坏性(**destructive**) 排序：直接覆盖原数组A
    - 原地(**in place**) 排序：使用$O(1)$额外空间（隐含是破坏性的）

### Permutation Sort

1. 算法思想
    - 尝试所有可能的排列（n!种）
    - 检查每个排列是否有序（$\Theta(n)$时间）
2. 实现示例
```python
def permutation_sort(A):
    for B in permutations(A):    # O(n!)
        if is_sorted(B):         # O(n)
            return B             # O(1)
```

3. 分析
    - 正确性：通过穷举所有可能性（暴力解法）
    - 时间复杂度：$\Omega(n! \cdot n)$（**指数级**，效率很低）:(

### Solving Recurrences

1. 代入法(**Substitution**)：
    - 猜测一个解
    - 用代表函数替换
    - 验证递归式是否成立

2. 递归树法(**Recurrence Tree**)：
    - 绘制表示递归调用的树
    - 对节点的计算量求和

3. 主定理(**Master Theorem**)：
    - 用于解决多种递归式的公式

### Selection Sort

#### Basic Concept

- 核心思想：找到前缀中最大的数并将其交换到正确位置
- 工作流程：
	1. 在`A[:i+1]`中找到最大数
    2. 将其与`A[i]`交换
    3. 递归排序前缀`A[:i]`
- 示例：`[8,2,4,9,3] → [8,2,4,3,9] → [3,2,4,8,9] → [2,3,4,8,9]`

#### Implementation

```python
def selection_sort(A, i = None):      # T(i)
	'''Sort A[:i+1]'''
    if i is None: i = len(A) - 1      # O(1)
    if i > 0:                         # O(1)
        j = prefix_max(A, i)          # S(i)
        A[i], A[j] = A[j], A[i]       # O(1)
        selection_sort(A, i - 1)      # T(i-1)

def prefix_max(A, i):                 # S(i)
	'''Return index of maximum in A[:i+1]'''
    if i > 0:                         # O(1)
        j = prefix_max(A, i - 1)      # S(i-1)
        if A[i] < A[j]:               # O(1)
			return j                  # O(1)
    return i                          # O(1)
```

#### Complexity Analysis

1. `prefix_max`分析：
    - 基本情况：$i=0$时，只有一个元素，最大值索引就是$i$
    - 归纳：正确性通过比较`A[:i]`的最大值和`A[i]`确保
    - 时间复杂度：$S(n) = S(n-1) + \Theta(1)$
	    - 代入法：$S(n) = \Theta(n)$，$cn = \Theta(1) + c(n-1) \Longrightarrow c = \Theta(1)$
	    - 递归树法：$\sum_{i=0}^{n-1} 1 = \Theta(n)$

2. `selection_sort`分析：
    - 基本情况：$i=0$时，单个元素已排序
    - 归纳：每次将最大值放到正确位置，剩余部分递归处理
    - 时间复杂度：$T(1) = \Theta(1)$，$T(n) = T(n-1) + \Theta(n)$
	    - 代入法：$T(n) = \Theta(n^2)$，$cn^2 = \Theta(n) + c(n-1)^2 \Longrightarrow c(2n-1) = \Theta(n)$ 
	    - 递归树法：$\sum_{i=0}^{n-1} i = \Theta(n^2)$

### Insertion Sort

#### Basic Concept

- 核心思想：递归排序前缀，然后通过重复交换将新元素插入到正确位置
- 工作流程：
	1. 递归排序`A[:i]`
	2. 将`A[i]`插入到已排序的`A[:i]`中正确位置
- 示例：`[8,2,4,9,3] → [2,8,4,9,3] → [2,4,8,9,3] → [2,3,4,8,9]`

#### Implementation

```python
def insertion_sort(A, i = None):      # T(i)
	'''Sort A[:i+1]'''
    if i is None: i = len(A) - 1      # O(1)
    if i > 0:                         # O(1)
        insertion_sort(A, i - 1)      # T(i-1)
        insert_last(A, i)             # S(i)

def insert_last(A, i):                # S(i)
	'''Sort A[:i+1] assuming sorted A[:i]'''
    if i > 0 and A[i] < A[i-1]:       # O(1)
        A[i], A[i-1] = A[i-1], A[i]   # O(1)
        insert_last(A, i-1)           # S(i-1)
```

#### Complexity Analysis

1. `insert_last`分析：
    - 基本情况：$i=0$或元素已在正确位置
    - 归纳：通过递归交换最后两个元素将元素移动到正确位置
    - 时间复杂度：$S(1) = \Theta(1), S(n) = S(n-1) + \Theta(1) \Longrightarrow S(n) = \Theta(n)$

2. `insertion_sort`分析：
    - 基本情况：$i=0$时，单个元素已排序
    - 归纳：归纳假设保证前缀已排序，`insert_last`保证新元素正确插入
    - 时间复杂度：$T(1) = \Theta(1), T(n) = T(n-1) + \Theta(n) \Longrightarrow T(n) = \Theta(n^2)$

### Merge Sort

#### Basic Concept

- 核心思想：分治策略，递归排序两半并合并
- 工作流程：
    1. 递归排序数组的前半部分和后半部分
    2. 使用两指针算法合并两个已排序的半部分
- 示例：`[7,1,5,6,2,4,9,3] → [1,7,5,6,2,4,3,9] → [1,5,6,7,2,3,4,9] → [1,2,3,4,5,6,7,9]`

#### Implementation

```python
def merge_sort(A, a=0, b=None):       # T(b-a=n)
    '''Sort A[a:b]'''
    if b is None: b = len(A)          # O(1)
    if 1 < b - a:                     # O(1)
        c = (a + b + 1) // 2          # O(1)
        merge_sort(A, a, c)           # T(n/2)
        merge_sort(A, c, b)           # T(n/2)
        L, R = A[a:c], A[c:b]         # O(n)
        merge(L, R, A, len(L), len(R), a, b)  # S(n)

def merge(L, R, A, i, j, a, b):       # S(b-a=n)
    '''Merge sorted L[:i] and R[:j] into A[a:b]'''
    if a < b:                         # O(1)
        if (j <= 0) or (i > 0 and L[i-1] > R[j-1]):  # O(1)
            A[b-1] = L[i-1]           # O(1)
            i = i - 1                 # O(1)
        else:
            A[b-1] = R[j-1]           # O(1)
            j = j - 1                 # O(1)
        merge(L, R, A, i, j, a, b-1)  # S(n-1)
```

#### Complexity Analysis

1. `merge`分析：
    - 基本情况：$n=0$时，数组为空，正确性显然成立
    - 归纳：假设对$n$成立，`A[b-1]`必须是`L`和`R`中的最大数，由于它们已排序，取最后一项即可
    - 时间复杂度：$S(0) = \Theta(1)$，$S(n) = S(n-1) + \Theta(1) \Longrightarrow S(n) = \Theta(n)$

2. `merge_sort`分析：
    - 基本情况：$n=1$时，单个元素已排序
    - 归纳：对于$k<n$的情况成立，算法递归排序较小的半部分，`merge`正确合并
    - 时间复杂度：$T(1) = \Theta(1)$，$T(n) = 2T(n/2) + \Theta(n)$
        - 代入法：猜测$T(n) = \Theta(n\log n)$
	        $cn\log n = \Theta(n) + 2c(n/2)\log(n/2) \Longrightarrow cn\log(2) = \Theta(n)$
        - 递归树分析：完全二叉树，深度$\log_2 n$，$n$个叶节点
            - 第$i$层有$2^i$个节点，每个节点工作量为$O(n/2^i)$
            - 总工作量：$\sum_{i=0}^{\log_2 n} (2^i)(n/2^i) = \sum_{i=0}^{\log_2 n} n = \Theta(n\log n)$
