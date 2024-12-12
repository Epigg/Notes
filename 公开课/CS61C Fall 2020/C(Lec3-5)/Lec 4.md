# C Intro: Pointers, Arrays, String

*November 17, 2024*

## Bugs and Pointers

#### Variable Declarations 

- 所有变量必须在它们被使用前声明
- 所有变量必须在代码块的开始声明
- 每个变量必须在声明时初始化，否则它将是无用值(**garbage**)

#### Undefined Behavior

很多 C 程序具有”未定义行为“，指的是行为不可预测，例如

- 在不同设备上，行为不同
- 甚至不同时间运行，行为不同

被称为“Heisenbugs”，即随机或难以重现、似乎在调试时消失或改变了的bugs。相对的，“Bohrbugs” 是可重复的。

#### Address vs. Value

将内存(memory)看作一个巨大的数组，每个房间具有一个特定的地址，并且存储值。

### Pointers

一个地址引用一个特定内存位置，换句话说，它指向一个内存位置。

指针：保存一个变量地址的变量。

#### Syntax

- `int *p;` - 告诉编译器变量 `p` 是 `int` 型的地址
- `p = &y;` - 告诉编译器将 `y` 的地址赋值给 `p`
- `z = *p;` - 告诉编译器将在 `p` 中地址的值赋值给 `z`

- `&` 是取地址运算符(address operator)
- `*` 是解引用运算符(dereference operator)

创建指针：

```C
int *p, x;
x = 3;
p = &x;
```

改变指针指向的变量：

```C
*p = 5;
```

#### Pointers and Parameter Passing

```C
void addOne (int x) { 
	x = x + 1;
} 
int y = 3; 
addOne(y);
```

`y` 仍然 = 3

```C
void addOne (int *p) {
	*p = *p + 1; 
} 
int y = 3; 
addOne(&y);
```

`y` 现在 = 4

#### Dangers

C 语言中的局部变量是不初始化的，它们可以包含任何东西。声明一个指针只是分配空间来保存指针——它并没有分配指针要指向的东西！

好处：

- 便于传递结构体或数组
- 通常来说，指针允许更简洁、紧凑的代码

指针可能是 C 程序中最大的 bug 来源，所以任何时候处理他们都要小心。

- 动态内存管理，最成问题的
- 悬空引用：在malloc之前使用
- 内存泄漏：迟到的free；丢失最初的指针导致无法 `free`

## Using Pointers Effectively

指针用于指向任何数据类型，通常一个指针只能指向一个类型。

`void *` 是可以指向任何数据类型的的通用指针。但是，谨慎使用以避免程序错误和安全问题等坏事。

指针也可以指向函数，例如 `int (*fn) (void *, void *) = &foo` ，其中  `fn` 是一个以两个 `void *` 为参数，返回 `int` 的函数指针，并且它初始化指向函数 `foo` 。通过 `(*fn)(x, y)` 来调用函数。

#### Pointers and Structures

```C
typedef struct {
    int x;
    int y;
} Point;
Point p1;
Point p2;
Point *paddr*;

/* dot notation */
int h = p1.x;
p2.y = p1.y;
/* arrow notation */
int h = paddr->x;
int h = (*paddr).x;
/* This works too */
p1 = p2
```

#### `NULL` pointer

如果你读或写 `NULL` ，程序会崩溃。

因为 “0 is false” ，所以：

- `if (!p) { /* P is a null pointer */}`
- `if (q) { /* Q is not a null pointer */}`

#### Pointing to Different Size Objects

现代机器是字节可寻址的(*byte-addressable*)，硬件的内存由8位存储单元组成，每个存储单元都有一个唯一的地址。

C 指针是一个抽象的内存地址。类型声明告诉编译器指针每次访问读取多少字节。

但是实际上我们想要的是内存对齐(*alignment*)。有些处理器不允许在没有4字节边界对齐的情况下处理32位值；其他处理器将会是很慢，如果试图访问“未对齐”的内存。

## Array

- `int ar[2];` - 声明一个有两个元素的数组。数组实际上只是一块内存
- `int ar[] = {795, 635};` - 声明并初始化。数组的大小为2
- `ar[num]` - 访问元素。得到第 num 个元素

数组*几乎*等同于指针。`char *string` 和 `char string[]` 是几乎相同的声明。

关键概念：数组变量是一个指向第一个元素的 “指针”（在很多方面相似）。

-  `ar[0]` 和 `*ar` 一样
-  `ar[2]` 和 `*(ar+2)` 一样

可以使用指针*计算*更方便地访问数组

- `pointer + n` - 得到 `pointer` 指向的内存地址加上 `n * sizeof(“指针指向的类型”)` 

仅在作用域有效时，声明的数组的内存才有效。

```C
char *foo() { 
char string[32];
...; 
return string; 
} // is incorrect
```

使用一个表示数组大小的变量声明数组

```C
// Wrong
int i, ar[10];
for(i = 0; i < 10; i++){ ... }
// Right
int ARRAY_SIZE = 10;
int i, a[ARRAY_SIZE];
for(i = 0; i < ARRAY_SIZE; i++){ ... }
```

使用间接的方式，避免保留两个数字10的副本

- 缺陷：C语言中的数组不知道它自己的长度，也不检查边界！
- 结果：可能会意外地访问数组的末尾。我们必须把数组和它的大小传递给一个遍历它的过程。

- Segmentation faults 访问的内存不属于当前程序所分配的内存空间
- bus errors 无法获取到所需要的内存，几乎都是因为未对齐的读写

### Function Pointer Example

```C
#include <stdio.h>
int x10(int), x2(int);
void mutate_map(int [], int n, int(*)(int)); 
void print_array(int [], int n); 

int x2 (int n) { return 2*n; } 
int x10(int n) { return 10*n; } 

void mutate_map(int A[], int n, int(*fp)(int)) {
	for (int i = 0; i < n; i++) 
		A[i] = (*fp)(A[i]);
} 
void print_array(int A[], int n) { 
	for (int i = 0; i < n; i++) 
	printf("%d ",A[i]); printf("\n"); 
}

int main(void) { 
	int A[] = {3,1,4}, n = 3; 
	print_array(A, n); 
	mutate_map (A, n, &x2); 
	print_array(A, n); 
	mutate_map (A, n, &x10); 
	print_array(A, n); 
}
```

```shell
% ./map 
3 1 4 
6 2 8 
60 20 80
```
