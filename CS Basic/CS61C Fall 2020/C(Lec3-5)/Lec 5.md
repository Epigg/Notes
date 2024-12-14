# C Memory Managements

*November 18, 2024*

## Memory

### Dynamic Memory Allocation

##### `sizeof()`

C 语言有 `sizeof()` 运算符，它以字节为单位给出（类型或变量的）大小。假设对象的大小可能会造成误导，并且是糟糕的风格，所以要使用 `sizeof(type)` 。

`sizeof` 知道数组（尽管是运行时才确定大小的数组）的大小：

```C
int n = 3;
int ar[n]; // Or: int ar[3]; int ar[] = {54, 47, 99};  
sizeof(ar) -> 12
```

注意：数组名并不是一个变量，对其取地址，即 `&ar` ，将得到*指向数组第一个元素的指针* 。数组名作为函数参数传递后，实际上传递的是 `&ar` ，失去了数组大小的信息，这时候 `sizeof(ar)` 将返回数组元素的类型的字节大小。

##### `malloc()` 

要为指向的新对象分配空间，可以使用 `malloc()` （在类型转换和 `sizeof` 的帮助下）：

`ptr = (int *) malloc (sizeof(int));`

- `ptr` 指向一个大小为 `sizeof(int)` 个字节的内存空间
- `(int *)` 告诉编译器该空间包含什么类型

分配一个包含 n 个整数的数组的空间：

`ptr = (int *) malloc (n * sizeof(int));`

当 `malloc()` 被调用时，分配的内存位置仍存储着无用值，所以知道你将其设置值，否则不要使用它。

如果内存分配失败，将会返回 `NULL` ，始终要检查返回的指针是否为 `NULL` 。

##### `free()`

`free(ptr);` - 在动态分配空间后，必须动态地释放它。尽管程序会在退出后释放所有内存，但是别偷懒！你永远不知道什么时候你的主程序会被转换成子程序！

下面两件事将导致你的程序崩溃或行为怪异，并导致非常很难找出 bug：

- `free()` 相同的内存空间两次
- 对不是 `malloc()` 返回的空间调用 `free()` 

运行时不会检查这些错误：

- 内存分配对性能至关重要，因此没有时间做这个
- 通常的结果是损坏了内存分配器的内部结构
- 直到很久以后，在代码中完全不相关的部分，错误才会被发现

##### `realloc(p, size)` - Managing the Heap

- 将先前分配给 `p` 的块调整大小为新的 `size`
- 如果 `p` 是 `NULL` ，`realloc` 和 `malloc` 一样
- 如果 `size` 是 0 ，`realloc` 和 `free` 一样，从堆中释放块
- 返回内存块的新地址；注意：它很可能已经移动了！  

## Linked List Example

```C
struct Node { 
	char *value; 
	struct Node *next; 
}; 
typedef struct Node *List; 

/* Create a new (empty) list */ 
List ListNew(void) { 
	return NULL; 
}
```

```C
/* add a string to an existing list */ 
List list_add(List list, char *string) { 
	struct Node *node = (struct Node*) malloc(sizeof(struct Node)); 
	node->value = (char*) malloc(strlen(string) + 1); 
	strcpy(node->value, string); 
	node->next = list; 
	return node; 
}
```

## Memory Locations

结构声明不分配内存，变量声明分配内存。

两种为数据分配内存的方法：

- 局部变量声明
	- `int i; struct Node list; char *string; int ar[n];`
- 调用分配函数(alloc)动态分配
	- `ptr = (struct *Node) malloc(sizeof(struct Node)*n);`

全局(*global*)变量：在所有过程之外声明的量；和局部变量很像，但是作用域是全局

### C Memory Management

C 有三部分内存内存池：

- **The Stack** 
- **The Heap** 
- **Static storage**

C 要求知道对象在内存中的位置，否则事情不会像预期的那样工作（相对地，Java 隐藏对象的位置）。

一个程序的地址空间包含4个区域（自上到下）：

- stack: 局部变量，自高地址向低地址增长
- heap: 动态分配给指针的空间，自低地址向高地址增长
- static: 全局变量，不会增长或减小
	- uninitialized data(bss, Block Started by Symbol): 未初始化过的全局变量和静态变量，在程序开始执行之前，内核将此段中的数据初始化为0或者NULL
	- initialized data(data): 程序中明确地赋初值的全局变量
- code: 程序启动时加载，不会改变

#### The Stack

堆栈帧(*frames*)包括：

- 返回后指令的地址 ("instruction" address)
- 参数
- 其他局部变量的空间

多个堆栈帧是连续的内存块；*堆栈指针*告诉堆栈帧的底部在哪里

堆栈帧以后进先出（LIFO）的顺序创建和销毁

当过程结束时，堆栈帧被抛出堆栈（事实上**只是将堆栈指针移回上一个帧**），为以后的堆栈帧释放内存。

## Memory Management

#### The Heap(Dynamic memory)

内存池很大，没有按连续顺序分配，对堆内存的连续请求可能导致块之间相距很远。

堆内存管理的要求：

- 快速运行 `malloc()` 和 `free()` 
- 最小的内存开销
- 要避免碎片化(*external fragmention*)
	- 当我们的大部分空闲内存都零碎、不连续地分布在许多小块中，我们可能有很多空闲字节，但无法满足大请求

堆内存是 C 代码最大的 bug 来源！

#### `malloc()` / `free()` Implementation

每个内存块(*memory block*)之前都有一个头，头包含元数据(metadata)：

- 块的大小
- 分配标志
- 指向下一个块的指针

所有空闲块都保存在一个循环链表中，在已分配块中的指针字段（*pointer field*，通常指数据结构中的一个成员指针）是未使用的。

##### `malloc()` 

- 在空闲列表中搜索一个足够大的块。如果没有找到，则从操作系统请求更多内存。如果它得到的结果不能满足请求，它将失败。
- 搜索的原则：
	- best-fit: 选择对请求来说足够大的最小块
	- first-fit: 选择发现的第一个足够大的块
	- next-fit: 就像 first-fit ，但记住我们在哪里结束搜索，然后从那里继续搜索

##### `free()`

- 检查与被释放的块相邻的块是否也是空闲的
- 如果是这样，相邻的空闲块将合并成一个更大的空闲块
- 否则，被释放的块只是被添加到空闲列表中

## When Memory Goes Bad

C 指针的[[CS Basic/CS61C Fall 2020/C(Lec3-5)/Lec 4#Dangers|好处和危险]]

1. 改写并未分配的空间
	- 将破坏程序的其他部分，之后可能会导致崩溃
2. C 中的指针允许访问已释放的内存，这将导致难以发现的 bug ！

```C
int *ptr () { 
	int y; 
	y = 3; 
	return &y; 
};
main () { 
	int *stackAddr, content; 
	stackAddr = ptr(); 
	content = *stackAddr; 
	printf("%d", content); /* 3 */ 
	content = *stackAddr; 
	printf("%d", content); /* 13451514 */ 
};
```

3. 使用`free` 后的指针
	- *Read*: 别的东西占据了指向的内存，程序可能会得到错误的信息
	- *Write*: 损坏其他数据，之后可能会导致崩溃
4. 忘记 `realloc` 会移动数据，
	- 原先的指针将指向无效内存
5. `free` 错误的对象
	- 不是 `malloc` 返回的指针
	- 将损坏其内部存储或擦除其他数据
6. 重复 `free` 
	- 可能会导致 `free` 之后使用的空间或损坏 `malloc` 的数据
7. 丢失最初的指针
	- 如果我们没有在其他地方保留原始 malloed 指针的副本，这可能造成内存泄漏
