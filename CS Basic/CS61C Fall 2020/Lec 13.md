# Compilation, Assembly, Linking, Loading

*November 25, 2024*

## Interpretation vs. Translation

### Great Idea #1: Abstraction (Levels of Representation/Interpretation)

- High Level Language Program (e.g., C)  ==**<--**==
- Assembly Language Program (e.g., RISC-V)  ==**<--**==
- Machine Language Program (RISC-V)  ==**<--**==
- Hardware Architecture Description (e.g. block diagrams)
- Logic Circuit Description (Circuit Schematic Diagrams)

### Interpretation

- Interpreter: 直接执行原语言程序
	- 但效率不太重要时，解释高级语言
- translate: 将程序从源语言转化为另一种语言的等效程序
	- 翻译成低级语言来提高性能

python解释器是一个程序，它读取一个python程序并执行这个python程序的功能

为什么用软件解释机器代码：
- 为了防止所有的程序需要重新从高级语言翻译
- 让可执行文件同时包含新的和旧的机器代码，必要的时候在软件中解释旧代码

### Interpretation vs. Translation

| 特性       | 解释（Interpret）      | 翻译（Translate，编译）      |
| -------- | ------------------ | --------------------- |
| 执行方式     | 逐行解释并执行            | 一次性将源代码翻译成机器码或字节码     |
| 执行效率     | 较慢                 | 较快                    |
| 是否生成独立文件 | 不生成独立文件，直接运行代码     | 生成独立的可执行文件            |
| 调试和修改    | 更易于调试，修改后无需重新编译    | 编译后需要重新生成可执行文件        |
| 例子       | Python, JavaScript | C, C++, Java (编译为字节码) |

- 一般来说，写解释器程序更容易
- 解释器更高级，所以可以给出好的错误信息
	- 翻译器的响应：添加额外信息以帮助调试（行号、名称等）
- 解释器更慢（约10倍），代码量更小（约2倍）
- 解释器提供了指令集独立性：可以在任何机器上运行
- 翻译/编译后的代码几乎总是更有效，因此性能更高
	- 这对很多应用很重要，尤其是操作系统
- 翻译/编译帮助想用户隐藏程序的源代码
	- 一种在市场中创造价值的模式
	- 另一种模式，“开源”，通过发布源代码和发展开发者社区来创造价值

## Compiler

![[CALL.png]]

- 输入：高级语言代码（如 foo.c）
- 输出：汇编语言代码（如用于 RISC-V 的 foo.s）
- 注意：输出*可以*包含伪代码
- 伪代码(*pseudo-instructions*)：编译器能理解但机器不理解的代码
	- 例如 `mv t1 t2 = addi t1, t2, 0`

## Assembler

- 输入：汇编语言代码（如用于RISC-V 的 foo.s）
- 输出：对象代码，信息表（仅限汇编代码）（如用于RISC-V的foo.o）
- 读取和使用指示(*Directives*)
- 替换伪代码
- 生成机器语言
- 生成对象文件(Object File)

### Assembler Direcitives

给汇编器提供指示，但不生产机器指令：

- `.text`：后续项放入用户文本段（机器代码）  
- `.data`：后续项放入用户数据段（源文件数据以二进制形式存储）  
- `.globl sym`：声明 `sym` 为全局变量，可以在其他文件中引用  
- `.string str`：将字符串 `str` 存储到内存中，并以空字符('\0')终止  
- `.word w1…wn`：将 n 个 32 位数据存储在连续的内存字中

### Pseudo-instruction Replacement

汇编器把方便的机器语言指令变体当作真正的指令来处理

| Pseudo        | Real                                              |
| ------------- | ------------------------------------------------- |
| mv t0, t1     | addi t0, t1, 0                                    |
| neg t0, t1    | sub t0, zero, t1                                  |
| li t0, imm    | addi t0, zero, imm                                |
| not t0, t1    | xori t0, t1, -1                                   |
| beqz t0, loop | beq t0, zero, loop                                |
| la t0, str    | lui t0, str\[31:12\] <br>addi t0, t0, str\[11:0\] |

### Producing Machine Language

- 简单情况
	- 算数运算、逻辑运算、位移等等
	- 所有必要的信息都包含在指令中
- 分支和跳转
	- PC相对的（例如beq/bne和jal）
	- 因此一旦伪指令被真实指令所取代，我们就知道有多少指令的距离需要分支/跳转

前向引用("Forward Reference")问题，分支指令可以引用程序中先前出现的指令。解决方案是两次通过整个程序：

- 第一次记住标签的位置
- 第二次用标签的位置产生机器代码
- 当然这之前需要先替换伪代码

PC相对的跳跃(jal)和分支(beq, bne)：

- `j offset` 是 `jal zero, offset` 的伪代码
- 只需计算目标和跳转位置之间的*半字*指令数(2字节)，就可以确定偏移量：位置无关代码（PIC）

引用静态数据需要数据的全部32位内存地址，使用 `la` 指令。但是这些还不能确定，所以我们创造了两个表：

- Symbol Table: 文件中可以被其他文件使用的“物品”列表
	- Labels: 函数调用
	- Data: `.data` 部分的所有东西；可跨文件访问的变量
- Relocation Table: 此文件需要其地址的“物品”列表
	- 所有要跳转到的绝对标签：jal, jalr
		- 内部的(Internal)
		- 外部的(External)（包括库文件）
		- 如la指令加载标签地址，jalr指令的基寄存器
	- 静态段中的任何数据
		- 比如la指令加载静态数据地址，如lw/sw指令的基寄存器

### Object File Format

- 对象文件头(object file header)：对象文件其他部分的大小和位置  
- 文本段(text segment)：机器代码  
- 数据段(data segment)：源文件中静态数据的二进制表示  
- 重定位信息(relocation information)：标识需要稍后修复的代码行  
- 符号表(symbol table)：该文件的标签和可引用的静态数据的列表
- 调试信息(debugging information)
- 标准格式为ELF （MS除外）

## Linker

- 输入：对象代码文件，信息表（如用于RISC-V 的 foo.o, libc.o）
- 输出：可执行代码（如用于 RISC-V 的a.out）
- 链接(*linking*): 将多个对象（.o）文件合并为单个可执行文件的过程
- 允许文件的独立编译
	- 对一个文件的更改不需要重新编译整个程序

![[Linker.png]]

三个步骤：

- 从每个.o文件中取出文本片段并将它们放在一起
- 从每个.o文件中取出数据段，将它们放在一起，并将其链接到文本段的末尾
- 解决引用
	- 查看重定位表；处理每个条目
	- 即，填入所有的绝对地址

四种寻址类型：

- 不需要重定位(PIC, Position-Independent Code)
	- pc相对寻址(beq, bne, jal；auipc/addi)
- 需要重新定位
	- 绝对函数寻址(auipc/jalr)
	- 外部函数寻址(auipc/jalr)
	- 静态数据寻址(通常是lui/addi)


> [!NOTE] Absolute Addresses in RISC-V
> 哪些指令需要重定位编辑：
> - J格式：跳转/跳转并链接
> - I格式，S格式：加载和存储静态区域的变量，相对于全局指针
> - B格式 条件分支：即使代码移动，也保留pc相对寻址

### Resolving References

- 链接器假设RV32文本段的第一个字在地址 0x10000
- 链接器知道：
	- 每个文本和数据段的长度  
	- 文本和数据段的排序
- 链接器计算：
	- 要跳转到的每个标签和每个被引用的数据的绝对地址（内部或外部）
- 处理引用：
	- 在所有“用户”符号表中搜索引用（数据或标签）
	- 如果没找到，搜索库文件（如 printf）
	- 一旦确定了绝对地址，就正确填写机器代码
- 链接器的输出：包含文本和数据（加上头）的可执行文件

### Static vs. Dynamically Linked Libraries

- 前面所描述的是传统的方法：静态链接(*statically-linked*)方法
	- 库是可执行文件的一部分，所以如果库更新，程序不会得到修复（如果我们有源代码，必须重新编译）
	- 需要包括整个库，尽管不是全部被使用
	- 可执行文件是自包含的，即包含了所有运行所需的代码和数据，不依赖任何外部文件或库
- 另一种方法是动态链接库(*dynamically-linked* libraries, DLL)，在Windows和UNIX平台上很常见

### Dynamically Linked Libraries

- 时间/空间问题
	- 存储程序需要更少的磁盘空间  
	- 发送程序需要更少的时间  
	- 执行两个程序需要更少的内存（如果它们共享一个库）
	- 在运行时，需要时间开销来做链接
- 程序升级
	- 替换一个文件(libXYZ.so)会升级所有使用库“XYZ”的程序
	- 只拥有可执行文件已经不够了
- 流行的动态链接方法使用机器代码作为“最小公约数”
	- 链接器不使用有关程序或库如何编译的信息（即，什么编译器或语言）
	- 可以被描述为“机器代码级的链接”

总的来说，动态链接给编译器、链接器和操作系统增加了相当多的复杂性。然而，它提供的许多好处往往超过这些。

## Loader

- 输入：可执行代码（如用于 RISC-V 的a.out）
- 输出：运行中的程序
- 可执行文件存储在磁盘上
- 当一个程序运行时，加载器的工作是将它加载到内存中并开始运行
- 实际上，加载器就是操作系统(OS)，加载是OS的任务之一

步骤：

- 读取可执行文件的头文件以确定文本和数据段的大小  
- 为程序创建一个新的地址空间，空间足够容纳文本和数据段，以及堆栈
- 将可执行文件复制指令和数据到新的地址空间  
- 将传递给程序的参数复制到堆栈上  
- 初始化机器寄存器  
	- 大多数寄存器被清除，但是堆栈指针分配了第一个空闲堆栈位置的地址  
- 跳转到启动例程，将程序的参数从堆栈复制到寄存器并设置PC  
	- 如果主程序返回，启动例程通过 `exit` 系统调用终止程序

