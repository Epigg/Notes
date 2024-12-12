# RISC-V Decisions II

*November 22, 2024*

## Logical Instructions

### RISC-V Logical Instructions

逻辑运算：

| Logical operations | C operators | Java operators | RISC-V instructions |
| :----------------: | :---------: | :------------: | :-----------------: |
| 按位与                |      &      |       &        |         and         |
| 按位或                |     \|      |       \|       |         or          |
| 按位异或               |      ^      |       ^        |         xor         |
| 逻辑左移               |     <<      |       <<       |         sll         |
| 逻辑右移               |     >>      |       >>       |         srl         |
 
每个运算都含有两个变体：

- Register: `and x5, x6, x7  # x5 = x6 & x7`
- Immediate: `andi x5, x6, 3  # x5 = x6 & 3` 

> [!note]
>- *"masks"* 掩码的使用，即与部分位为1的立即数进行逻辑运算
>- 在 RISC-V 中没有“非”，因为可以用与 $11111111_{two}$ 进行按位异或代替

### Shifting

右移分为逻辑右移和算术右移两种，其扩展高位规则与 `sb` 类似

-  `srl, srli` 逻辑右移：始终用0扩展最高位
-  `sra, srai` 算数右移：使用原始最高位扩展最高位

左移在没有溢出的情况下，结果相当于是乘2的幂次。右移的结果相当于是除以2的幂次并向负无穷舍入。

## A Bit About Machine Program

汇编程序源文件(foo.s)经过汇编器(*Assembler*)，被转换为指令位模式，得到机器代码对象代码文件(foo.o)，再通过链接器(*Linker*)与预构建的对象代码文件文件库(lib.o)链接，得到机器代码可执行文件(a.out)。

> [!note] Program Execution
>- 程序计数器（program counter, PC）是处理器内部的一个寄存器，保存下一条要执行的指令的内存地址
>- 从内存中获取指令，然后控制单元利用数据路径和内存系统执行指令，并更新PC（默认添加4字节到PC，移动到下一个顺序指令，此外有分支，跳转指令）

有用的 RISC-V 汇编器特性：

1. 符号化的(Symbolic)寄存器名称
	- 例如， `a0-a7` 代表寄存器 `x10-x17` 用于参数调用
	-  `zero` 代表 `x0` 
2. 伪指令(Psedu-instrucions)
	- 即常用的汇编命令的简化语法：
	-  `mv` (move): `mv rd, rs = addi rd, rs, 0`
	-  `li` (load immediate): `li rd, 13 = add rd, x0, 13`
	-  `nop` (no oparation): `nop = addi x0, x0, 0`

## RISC-V Function Calls

### Six Fundamental Steps in Calling a Function

1. 把*参数*放在函数可以访问的地方（寄存器）
2. 将控制传递给函数( `jal` )
3. 获取函数所需的（局部，或者说本地）存储资源
4. 执行该函数所要求的任务  
5. 将*返回值*放在调用代码可以访问的地方，恢复使用的所有寄存器，并释放局部存储
6. 将控制返回到调用的起始点( `ret` )，因为一个函数可以从程序中的几个地方被调用

### RISC-V Function Call Conventions

- 寄存器比内存快，所以使用它们
- a0-a7(x10-x17): 8个参数寄存器用于传递参数，其中两个用于返回值(a0-a1)
- ra(x1): 存储返回地址(*return address*)的寄存器
- s0-s1(x8-x9)和s2-s11(x18-x27): 保存寄存器

### Instruction Support for Functions

```C
	...
	sum(a,b); /* a, b: s0, s1 */ 
	... 
} 
int sum(int x, int y) {
	return x + y; 
}
```

在 RISC-V 中，所有指令都是4字节，并像数据一样存储在内存中。因此，这里我们显示了存储程序的地址。

```
1000 mv a0, s0       # x = a 
1004 mv a1, s1       # y = b 
1008 addi ra, zero, 1016     # ra=1016 
1012 j sum           # jump to sum    
1016 …               # next inst.
… 
2000 sum: add a0, a0, a1 
2004 jr ra           # new instr."jump reg"
```

为什么在这里使用 `jr` ，而为什么不用 `j` 呢？因为 `sum` 可能被很多地方调用，所以我们不能返回到一个固定的地方。调用 `sum` 的进程必须能够以某种方式“返回起点”。

单指令跳转和保存返回地址：jump and link ( `jal` )

```
1008 addi ra, zero, 1016     # ra=1016 
1012 j sum          # goto sum

1008 jal sum        # ra=1012, goto sum
```

`jal` 的好处：

- 使常见的情况快速：函数调用非常常见  
- 减小程序大小  
- 不需要知道指令在内存中的位置

总的来说，需要两个指令：

- 调用函数： `jal` 
	- “链接”是指形成指向调用站点的地址或链接，以允许函数返回到正确的地址  
	- 跳转到地址，同时将接下来的指令的地址保存在寄存器ra中
- 返回： `jr`
	- 无条件跳转到寄存器中指定的地址: `jr ra` 
	- 汇编程序简化语法: `ret = jr ra`

> [!summary]
> 事实上标准指令只有两个：
> - `jal rd, Label` - jump-and-link
> - `jalr rd, rs, imm` - jump-and-link register
> - 其中 `rd` 保存返回地址，会跳转到 `rs` 存储的地址加 `imm`  
> 
> `j`, `jr` 和 `ret` 都是伪指令，或者说简化语法：
> - `j`: `jal x0, Label` 
