# Hashing

*December 14, 2024*

## Introduction

- 思考：能否比$\Theta(\log n)$更快地实现`find(k)`？
- 理论上不能（下界定理），但实际上...可以！？

---

## Comparison Model

1. 基本假设：
    - 算法只能通过比较来区分项目
    - 可比较的项目(**Comparable items**)是一种黑盒对象
	    - 只支持成对比较
		- 无法直接访问项目的内部结构
    - 比较运算：$\lt、\leq、\gt、\geq、=、\neq$，返回True或False

2. 目标：
    - 存储$n$个可比较项目
    - 支持`find(k)`操作
    - 运行时间下界由比较次数决定

## Decision Tree

- 任何算法都可以看作操作的决策树(**decision tree**)
- 内部节点表示二元比较(**binary comparison**)，分支为True或False
- 叶节点表示算法终止，得到输出结果
- 根到叶的路径(**root-to-leaf path**)代表算法在特定输入上的执行(**execution**)
- 每个搜索结果至少需要对应一个叶节点，除n个元素外还需要一种未找到的结果，因此搜索需要$\geq n+1$个叶节点

### Comparison Search Lower Bound

1. 比较搜索算法的最坏运行时间：
    - 运行时间 ≥ 比较次数 ≥ 根到叶路径最长长度 ≥ 树高

2. 二叉树性质：
    - 含$n$个节点的二叉树最小高度在完全二叉树时达到
    - 高度 $\geq \lceil \log(n+1) \rceil - 1 = \Omega(\log n)$
    - 排序数组实现达到这个界

3. 推广：
    - 有$\Theta(n)$个叶节点、最大分支因子$b$的树高度为$\Omega(\log_b n)$

如果能用一种操作，使分支因子（或出度）逐渐大于常数$\omega(1)$，那么决策树就会更浅，从而加快算法速度（例如B树）

---

## Direct Access Array


1. 想法
	- 利用Word-RAM的$O(1)$时间随机访问
	- 为每个项目分配**唯一**整数键$k$，范围为$\{0,...,u-1\}$，存储在数组对应位置

2. 优点：
    - 如果键能放入机器字（$u \leq 2^w$），则最坏情况下find/动态操作都是$O(1)$

3. 缺点：
    - 空间复杂度$O(u)$，当$n \ll u$时非常浪费
    - 示例：10字母名字作为键，每个名字1位，需要$26^{10} \approx 17.6$ TB空间

## Hashing

- 当$n \ll u$时，将键映射到更小的范围$m = \Theta(n)$，使用更小的直接访问数组
- 哈希函数：$h(k)：\{0,...,u-1\} \rightarrow \{0,...,m-1\}$
- 直接访问数组称为哈希表(**hash table**)，$h(k)$称为键k的哈希值

冲突(**Collision**)：

- 当$m \ll u$时，由抽屉原理，任何哈希函数都不是单射
- 必然存在键a、b使得$h(a) = h(b)$ → 发生冲突！:(
- 解决方案：
	1. 存储在数组的其他位置（开放寻址，**open addressing**）
		- 分析复杂，但普遍实用
    2. 存储在支持动态集合接口的其他数据结构中（链接，**chaining**）

---

## Chaining

- 核心思想：将碰撞的元素存储在另一个数据结构（链）中
- 性能分析：
    - 理想情况：键均匀分布，链长度为$n/m = n/\Omega(n) = O(1)$
    - 最佳情况：链长度为$O(1)$时，所有操作为$O(1)$时间
    - 最差情况：多个项目映射到同一位置，如$h(k)=constant$，链长度为$\Theta(n)$

- 需要一个好的哈希函数！什么是好的哈希函数？

---

## Hash Functions

### Division (bad)

$$
h(k) = (k\ mod\ m)
$$

- 特点：
    - 启发式方法，在键**均匀分布**时效果好
    - m应避免与存储键的模式存在对称性
    - 选择远离2和10的幂（数据模式相关的数）的大素数较好
    - Python使用这种方法的改进版本

- 局限性：
    - 当$u \gg n$时，任何哈希函数都存在导致$O(n)$长度链的输入集
    - 改进思路：不使用固定哈希函数，而是随机（但要谨慎）选择

### Universal (good, theoretically)

$$
h_{ab}(k) = (((ak + b)\ mod\ p)\ mod\ m)
$$

#### Concept

- 哈希族

$$
\mathcal{H}(p,m) = \{h_{ab} | a,b \in \{0,...,p-1\} \text{ and } a \neq 0\}
$$

- 参数
    - 固定素数$p > u$
    - a和b从$\{0,...,p-1\}$中选择


- $\mathcal{H}$是通用(**Universal**)函数族：对任意$k_i \neq k_j \in \{0,...,u-1\}$，有$\text{Pr}_{h \in \mathcal{H}}\{h(k_i) = h(k_j)\} \leq 1/m$

#### Theoretical Analysis

1. 定义随机变量$X_{ij} = 1$，对于$h \in \mathcal{H}$
    - $X_{ij} = 1$ 如果$h(k_i) = h(k_j)$
    - $X_{ij} = 0$ 其他情况

2. 链长度
	- 位置$h(k_i)$的链长度为随机变量$X_i = \sum_j X_{ij}$
	- 期望链长度：

$$
\mathbb{E}_{h \in \mathcal{H}}\{X_i\} \leq 1 + (n-1)/m
$$

- 因为$m = \Omega(n)$，载荷因子(load factor) $\alpha = n/m = O(1)$，所以期望是$O(1)$！

## Dynamic

- 当$n/m \gg 1$时，使用新的大小$n$和新的随机选择的哈希函数重建表
- 分析类似动态数组，成本可以分摊到多个动态操作上
- 所以哈希表可以在期望平摊的$O(1)$时间完成动态集合操作！:)

---

## Complexity Table

|            Operations            | Array  | Sorted Array  | Direct Access Array |    Hash Table     |
|:--------------------------------:|:------:|:-------------:|:-------------------:|:-----------------:|
|            `build(X)`            | $O(n)$ | $O(n \log n)$ |       $O(u)$        | $O(n)^{\dagger}$  |
|            `find(k)`             | $O(n)$ |  $O(\log n)$  |       $O(1)$        | $O(1)^{\dagger}$  |
|    `insert(x)`<br>`delete(k)`    | $O(n)$ |    $O(n)$     |       $O(1)$        | $O(1)^{*\dagger}$ |
|   `find_min()`<br>`find_max()`   | $O(n)$ |    $O(1)$     |       $O(u)$        |      $O(n)$       |
| `find_prev(k)`<br>`find_next(k)` | $O(n)$ |  $O(\log n)$  |       $O(u)$        |      $O(n)$       |

$^*$: 平摊时间
$^\dagger$: 期望时间
