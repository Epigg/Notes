# Data Structures

*December 12, 2024*

## Data Structure Interfaces

- 数据结构是存储数据的方式，并提供支持数据操作的算法
- 接口(Interface)是对支持操作的集合(collection)，也称为API或ADT
- 接口是规范(**specification**)：说明支持什么操作（问题）
- 数据结构是表现(**representation**)：实现如何支持操作（解决方案）

## Sequence Interface

维护项目的**外在**顺序
### Basic Operations

1. 容器操作：
    - `build(X)`：从可迭代对象X构建序列
    - `len()`：返回存储项目数量

2. 静态操作：
    - `iter_seq()`：按序返回所有项目
    - `get_at(i)`：获取第i个项目 
    - `set_at(i,x)`：用x替换第i个项目

3. 动态操作：
    - `insert_at(i,x)`/`delete_at(i)`：在第i位置插入/删除
    - `insert_first(x)`/`delete_first()`：在开头插入/删除
    - `insert_last(x)`/`delete_last()`：在末尾插入/删除

### Special Interfaces

- 栈(Stack)：仅支持末尾的插入和删除
- 队列(Queue)：支持末尾插入和开头删除

## Set Interface

维护项目的**内在**顺序(基于键)

### Basic Operations

1. 容器操作：
    - `build(X)`和`len()`：同序列接口

2. 静态操作：
    - `find(k)`：返回键为k的项目

3. 动态操作：
    - `insert(x)`：插入x（如存在相同键则替换）
    - `delete(k)`：删除并返回键为k的项目

4. 顺序操作：
    - `iter_ord()`：按键顺序返回项目
    - `find_min()`/`find_max()`：返回最小/最大键项目
    - `find_next(k)`/`find_prev(k)`：返回键大于/小于k的最近项目

### Special Case

- 字典(Dictionary)：不包含顺序操作的集合

## Array Sequence

- 静态操作高效：`get_at(i)`和`set_at(i,x)`是$\Theta(1)$
- 动态操作低效，为$\Theta(n)$
	- 需要重新分配空间和移动元素

## Linked List Sequence

- 头部动态操作高效：`insert_first`和`delete_first`是$\Theta(1)$
- 随机访问低效：`get_at(i)`和`set_at(i,x)`是$O(n)$

## Dynamic Array Sequence

- 综合了数组和链表的优点
- 关键思想：**预分配额外空间**
- **填充率**(r)：$0 \leq r \leq 1$
	- 当数组填满时，分配$\Theta(n)$的额外空间到末尾使填充率降至给定的 $r_i$(如0.5)
	- 下次分配前需要插入$\Theta(n)$个项目
- 平摊分析：单次操作可能是$\Theta(n)$，但平均是 Θ(1)
- Python列表就是动态数组的实现

### Amortized Analysis

- 平摊分析是数据结构的一种分析技术：将操作代价分摊到多次操作上
- 平摊代价$T(n)$：k次操作总代价小于 $kT(n)$
- 动态数组**末尾插入**的平摊时间是$\Theta(1)$

### Dynamic Array Deletion

- 从数组末尾删除元素的时间复杂度为$\Theta(1)$
- 但需要注意空间利用效率的问题,数据结构的大小应保持在$\Theta(n)$
- 数组收缩条件:
    - 当使用率 $r < r_d$ 时,将数组大小调整为 $r_i$
    - 其中 $r_i < r_d$ （例如 $r_d = 1/4$，$r_i = 1/2$）
    - 这确保在下一次昂贵的调整大小操作之前,可以进行$\Theta(n)$次廉价操作
- 空间使用优化:
    - 可以将额外空间使用限制在 $(1 + \varepsilon)n$，对任意 $\varepsilon > 0$
    - 例如设置 $r_d = 1/(1+\varepsilon)$，$r_i = 1/2$
- 对Python列表：
	- `append` 和 `pop` 的均摊时间复杂度为 $O(1)$
	- 其他操作可能需要 $O(n)$ 时间

## Time Complexity Table

|              Operations               | Static Array | Linked List | Dynamic Array |
| :-----------------------------------: | :----------: | :---------: | :-----------: |
|              `build(X)`               |    $O(n)$    |   $O(n)$    |    $O(n)$     |
|     `get_at(i)`<br>`set_at(i, x)`     |    $O(1)$    |   $O(n)$    |    $O(1)$     |
| `insert_first(x)`<br>`delete_first()` |    $O(n)$    |   $O(1)$    |    $O(n)$     |
|  `insert_last(x)`<br>`delet_last()`   |    $O(n)$    |   $O(n)$    |   $O(1)^*$    |
|  `insert_at(i, x)`<br>`delete_at(i)`  |    $O(n)$    |   $O(n)$    |    $O(n)$     |

$^*$: 平摊时间
