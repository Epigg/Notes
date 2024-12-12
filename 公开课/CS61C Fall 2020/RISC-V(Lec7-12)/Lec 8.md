# RISC-V lw, sw, Decisions I

*November 21, 2024*

## Storing Data in Memory

数据传输：

- **Load from** memory: **从**内存加载
- **Store to** memory: 存储**到**内存

### Memory Addresses are in Bytes

数据通常小于32位，但很少小于8位，因此如果所有内容都是8位的倍数，则可以正常工作。

8个位称为1个字节，4个字节称为1个字。内存地址实际上是以字节为单位，而不是以字为单位。

字地址间隔为4字节。字地址与最右字节，即最低有效字节，的地址相同（即小端法(**Little-ending**)惯例）。

绝大部分机器采用的是小端法(Little Endian)，有少部分采用大端法(Big Endian)。

## Data Transfer Instructions

寄存器的速度比动态随机存取存储器(Dynamic Random Access Memory, DRAM)快 50~500 倍。就一次的访问而言相差数十纳秒，但接下来的字每隔几纳秒就会出现一次。

### Load from

C 代码

```C
int A[100];
g = h + A[3];
```

RISC-V, 使用 `lw` (Load Word)

```
lw x10, 12(x15)    # Reg x10 gets A[3]
add x11, x12, x10  # g = h + A[3]
```

其中， `x15` 是基址寄存器(base register)，指向 `A[0]` ；12 是字节偏移量(**offset**)，偏移量必须是在汇编时就已知的常数

数据流是从右到左的

### Store to

C 代码

```C
int A[100];
A[10] = h + A[3];
```

RISC-V, 使用 `sw` (Store Word)

```
lw x10, 12(x15)    # Temp reg x10 gets A[3]
add x10, x12, x10  # Temp reg x10 gets h + A[3]
sw x10, 40(x15)    # A[10] = h + A[3]
```

其中， `x15` 是基址(base)寄存器；12 和 40 是字节偏移量，x15+12 和 x15+40 必须是 4 的倍数

数据流是从左到右的

### Loading and Storing Bytes

除了字数据传输（lw, sw）之外，RISC-V 还具有*字节*数据传输：

- 加载字节： `lb` 
- 存储字节： `sb` 

和 `lw sw` 是相同的格式，例如：

`lb x10, 3(x11)` 

寄存器x11指向的地址加 3 的总地址对应的内容，被复制到寄存器x10的*低字节位置*。但是一个寄存器存储大小是字而不是字节，所以需要对加载的字节进行符号扩展(*sign-extend*)，即将该字节的*最高位*扩展到其他的 24 位。

RISC-V 还具有“无符号字节”的加载指令，即 `lbu` ，这时候不是使用符号扩展，而是使用零扩展(*zero-extend*)来填充寄存器。为什么没有 `sbu` ，因为这没有意义。

### Substituting addi

我们可以用两个指令

```
lw x10, 12(x15)         # Temp reg x10 gets A[3] 
add x12, x12, x10       # reg x12 = reg x12 + A[3]
```

代替 `addi` 

```
addi x12, value         # value in A[3]
```

但是涉及到从内存加载数据！立即加法非常常见，所以它值得有自己的指令！

## Decision Making

决策过程，即根据计算结果，做不同的事情。在编程语言中即为 if 语句，在 RISC-V 中类似 if 语句的指令是：

`beq reg1, reg2, L1`

它的含义是，如果寄存器1的值等于寄存器2的值，那么跳转到标记为 L1 的语句，否则就继续下一个语句。

`beq` 指 "branch if equal"，其中分支(branch)指控制流的改变。

条件分支(Conditional Branch)指根据比较结果更改控制流。

其他指令：

- `bne` : branch if not equal
- `blt` : branch if less than
- `bge` : branch if greater than or equal
- `bltu` `bgeu` : 无符号版本

无条件分支，即总是改变控制流。RISC-V 中一条指令：`j label` 

注意：没有 `bgt` 和 `ble` 两个命令，因为只需要交换 `blt` 和 `bge` 指令两个寄存器的顺序即可。对于无符号版本，将寄存器内容看作是无符号整型而不是有符号整型（补码）进行比较即可。

### if

`f->x10  g->x11  h->x12  i->x13  j->x14`

```C
if (i == j)
	f = g + h;
```

```
	bne x13, x14, Exit
	add x10, x11, x12
Exit:
```

### if-else

```C
if (i == j)
	f = g + h;
else 
	f = g - h;
```

```
	bne x13, x14, Else
	add x10, x11, x12
	j Exit
Else: sub x10, x11, x12
Exit:
```

### Loop

虽然在 RISC-V 中有多种编写循环的方法，但决策的关键是条件分支

```C
int A[20];
int sum = 0;
for (int i = 0; i < 20; i += 1)
	sum += A[i];
```

```
	add x9, x8, x0     # x9=&A[0] 
	add x10, x0, x0    # sum
	add x11, x0, x0    # i
	addi x13,x0, 20    # x13
Loop:
	bge x11, x13, Done 
	lw x12, 0(x9)      # x12 A[i] 
	add x10, x10, x12  # sum
	addi x9, x9, 4     # &A[i+1] 
	addi x11, x11, 1   # i++ 
	j Loop 
Done:
```
