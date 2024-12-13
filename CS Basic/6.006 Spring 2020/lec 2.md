# Data Structures

*December 12, 2024*

## Data Structure Interfaces

- 数据结构是存储数据的方式，并提供支持数据操作的算法
- 接口(Interface)是对支持操作的集合(collection)
- 接口是**规范**：定义了"做什么"(问题)
- 数据结构是**表现**：实现了"怎么做"(解决方案)

## Sequence Interface

- 维护项目的**外在**顺序
- 操作类型：
	- 容器操作：
		- build(X): 给定可迭代X，从X中的项目创建序列
		- len(): 返回存储的项目数
	- 静态操作：
		- iter_seq(): 按序返回所有项目
	    - get_at(i): 获取第i个项目 
	    - set_at(i,x): 用x替换第i个项目
	- 动态操作：
	    - insert_at(i, x)/delete_at(i): 在第i个位置插入x/删除
	    - insert_first(x)/delete_first(): 在开头插入x/删除
	    - insert_last(x)/delete_last(): 在末尾插入x/删除
- 特例接口：
	- 栈(Stack)：仅支持末尾插入和删除
	- 队列(Queue)：支持末尾插入和开头删除

## Set Interface

- 维护项目的**内在**顺序(基于键)
- 操作类型：
	- 容器操作：build(X), len()
	- 静态操作：
		- find(k): 查找键为k的项目
	- 动态操作：
		- insert(x): 插入x(代替键为x.key的项目，如果存在的话)
		- delete(k): 删除键为k的项目
	- 顺序操作：
		- iter_ord(): 返回按键顺序的迭代器
		- find_min/find_max(): 返回存储的项目中的最小/最大键
		- find_next/find_prev(k): 返回最小/最大的键大于/小于k的项目
- 特例接口：
	- 字典(Dictionary)：没有顺序操作的集合

## Array Sequence

- 静态操作效率高：get_at(i)和set_at(i,x)是Θ(1)
- 动态操作效率低，为Θ(n)
	- 需要重新分配空间和移动元素
## Linked List Sequence

- 动态操作在开头高效：insert_first和delete_first是Θ(1)
- 随机访问效率低：get_at(i)和set_at(i,x)是O(n)

## Dynamic Array Sequence

- 综合了数组和链表的优点
- 关键思想：**预分配额外空间**
- **填充率**(r)维护：0 ≤ r ≤ 1
	- 当数组填满时，分配 Θ(n) 的额外空间到末尾以满足给定的填充率$r_i$(如0.5)
	- 下次分配前需要插入 Θ(n) 个项目
- 平摊分析：单次操作可能是 Θ(n)，但平均是 Θ(1)
- Python列表就是动态数组的实现

### Amortized Analysis

- 平摊分析是数据结构的一种分析技术：将操作代价分摊到多次操作上
- 平摊代价T(n)：k次操作总代价小于 kT(n)
- 动态数组**末尾插入**的平摊时间是 Θ(1)

### Dynamic Array Deletion

- 从数组末尾删除元素的时间复杂度为 Θ(1)
- 但需要注意空间利用效率的问题,数据结构的大小应保持在 Θ(n)
- 数组收缩条件:
    - 当使用率 $r < r_d$ 时,将数组大小调整为 $r_i$
    - 其中 $r_i < r_d$ (例如: $r_d = 1/4, r_i = 1/2$)
    - 这确保在下一次昂贵的调整大小操作之前,可以进行 Θ(n) 次廉价操作
- 空间使用优化:
    - 可以将额外空间使用限制在 $(1 + ε)n$，对任意 ε > 0
    - 例如设置 $r_d = 1/(1+ε), r_i = 1/2$
- 对Python列表：
	- append 和 pop 的均摊时间复杂度为 O(1)
	- 其他操作可能需要 O(n) 时间

## Time Complexity Table

| 操作类型                              | 静态数组 | 链表   | 动态数组   |
| --------------------------------- | ---- | ---- | ------ |
| build(X)                          | O(n) | O(n) | O(n)   |
| get_at(i)<br>set_at(i, x)         | O(1) | O(n) | O(1)   |
| insert_first(x)<br>delete_first() | O(n) | O(1) | O(n)   |
| insert_last(x)<br>delet_last()    | O(n) | O(n) | O(1)\* |
| insert_at(i, x)<br>delete_at(i)   | O(n) | O(n) | O(n)   |

\*: 均摊时间复杂度
