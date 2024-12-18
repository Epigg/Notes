# RISC-V Instruction Formats I

*November 24, 2024*

## RISC-V Instruction Representation

### Great Idea #1: Abstraction (Levels of Representation/Interpretation)

- High Level Language Program (e.g., C)
- Assembly Language Program (e.g., RISC-V)  
- Machine Language Program (RISC-V)  ==**<--**==
- Hardware Architecture Description (e.g. block diagrams)
- Logic Circuit Description (Circuit Schematic Diagrams)

### Consequence

1. 所有东西都有内存地址
	- 因为所有的指令和数据都存储在内存中，所以所有的东西都有一个内存地址，包括指令和数据字。
		- 分支和跳跃都需要这些
	- C指针只是内存地址：它们可以指向内存中的任何东西。
		- 不受约束地使用地址可能会导致严重的错误
		- 避免C语言中的错误
	- 一个寄存器保存正在执行的指令的地址：“程序计数器”（PC）。本质
		- 上就是一个指向内存的指针
		- 英特尔称之为指令指针（IP）
2. 二进制兼容性
	- 程序以二进制形式分发
		- 程序捆绑到特定的指令集
		- 手机和个人电脑有不同的程序版本
	- 新机器希望运行旧的程序（“二进制文件”），以及为新指令集编译的程序
	- 这导致了“向后兼容”的指令集随着时间的推移不断演进

### Instructions as Numbers

- 我们处理的大部分数据都是字
	- 每个寄存器是一个字
	- lw 和 sw 每次都访问内存中的一个字
- 如何表达指令
	- 记住：计算机只理解1和0，所以汇编字符串 `add x10,x11,x0` 对硬件是没有意义的
	- RISC-V 追求简单性：因为数据是用字表示的，所以指令也是固定大小的32位字
		- 相同的32位指令被RV32，RV64，RV128使用
- 一个字是32位，所以把指令字拆分为字段(*fields*)
- 每个字段告诉处理器一些关于指令的信息
- 我们可以为每条指令定义不同的字段。RISC-V 寻求简单性，定义了六种基本类型的指令格式：
	- R-format: 寄存器-*寄存器*算术运算
	- I-format: 寄存器-*立即数*算数运算、加载
	- S-format: *存储*操作
	- B-format: *分支*（S格式的小变体）
	- U-format: 20位*上位*立即数指令
	- J-format: *跳转*（U格式的变体）

## R-Format Layout

`31 25|24 20|19 15|14 12|11 7|6 0`

| funct2 | rs2 | rs1 | funct3 | rd  | opcode |
| :----: | :-: | :-: | :----: | :-: | :----: |
|   7    |  5  |  5  | 3      | 5   | 7      |

表格的第一行为字段的名称，第二行为字段的字节数，例如：

- opcode(operation code)是一个7位字段，位于指令的6-0位
- rs2是一个5位字段，位于指令的24-20位

1. opcode: 部分决定该机器代码是什么指令
	- 对于所有R-Format的寄存器-寄存器算术指令，该字段等于0110011
2. Funct7 +funct3: 与opcode结合，这两个字段描述要执行的操作
3. rs1(Source Register #1): 指定包含第一个操作数的寄存器
4. rs2: 指定第二寄存器操作数
5. rd(Destination Register): 指定接收计算结果的寄存器
6. 每个寄存器字段保存一个5位无符号整数（0-31），对应于寄存器编号（x0-x31）

### All RV32 R-format Instructions

| operation | funct7  | rs2 | rs1 | funct3 | rd  | opcode  |
| :-------: | :-----: | :-: | :-: | :----: | :-: | :-----: |
|    add    | 0000000 |     |     |  000   |     | 0110011 |
|    sub    | 0100000 |     |     |  000   |     | 0110011 |
|    sll    | 0000000 |     |     |  001   |     | 0110011 |
|    slt    | 0000000 |     |     |  010   |     | 0110011 |
|   sltu    | 0000000 |     |     |  011   |     | 0110011 |
|    xor    | 0000000 |     |     |  100   |     | 0110011 |
|    srl    | 0000000 |     |     |  101   |     | 0110011 |
|    sra    | 0100000 |     |     |  101   |     | 0110011 |
|    or     | 0000000 |     |     |  110   |     | 0110011 |
|    and    | 0000000 |     |     |  111   |     | 0110011 |

通过funct7 + funct3中不同的编码选择不同的操作

其中 `slt` `sltu` 是两个新指令，意味着"set on less than"，如果r1小于r2，则将rd设置为1，否则为0

## I-Format Layout

5位字段只表示至多到31的数字，而立即数可能比这个大得多。理想地，为了简单性 RISC-V 应该只有一中指令格式，但是我们不得不妥协。

注意，如果指令具有立即数，则最多使用2个寄存器（一个源，一个目标）。I格式定义与R格式基本一致：

`31 20|19 15|14 12|11 7|6 0`

| imm\[11 : 0\] | rs1 | funct3 | rd  | opcode |
| :-----------: | :-: | :----: | :-: | :----: |
|      12       |  5  |   3    |  5  |   7    |

- 用12位的有符号立即数 imm\[11:0\] 代替 funct7 和 rs2
- 其余字段与原先相同
- 立即数的范围为\[-2048, 2047\]
- 在用于算术运算之前，立即数总是先被*符号扩展*到32位
- 对于大于12位的立即数

### All RV32 I-format Arithmetic Instructions

| operation |  imm\[11 : 0\]   | rs1 | funct3 | rd  | opcode  |
| :-------: | :--------------: | :-: | :----: | :-: | ------- |
|   addi    |                  |     |  000   |     | 0010011 |
|   slti    |                  |     |  010   |     | 0010011 |
|   sltiu   |                  |     |  011   |     | 0010011 |
|   xori    |                  |     |  100   |     | 0010011 |
|    ori    |                  |     |  110   |     | 0010011 |
|   andi    |                  |     |  111   |     | 0010011 |
|   slli    | 0000000 \| shamt |     |  001   |     | 0010011 |
|   srli    | 0000000 \| shamt |     |  101   |     | 0010011 |
|   srai    | 0100000 \| shamt |     |  101   |     | 0010011 |

shamt: shift amount, 只使用立即数的低5位作为移位量

立即数的一个高阶位用于区分“右移逻辑”（SRLI）和“右移算术”（SRAI）

## RISC-V Loads

加载指令也是 I 格式。其中，12位立即数作为偏移量(offset)加到rs1中的基地址得到内存地址。从内存中加载的值存储在寄存器rd中。

`31 20|19 15|14 12|11 7|6 0`

| imm\[11 : 0\]  | rs1  | funct3 |  rd  | opcode |
| :------------: | :--: | :----: | :--: | :----: |
|       12       |  5   |   3    |  5   |   7    |
| offset\[11:0\] | base | width  | dest |  LOAD  |

- LOAD opcode: 0000011

### All RV32 Load Instructions

| operation | imm\[11 : 0\] | rs1 | funct3 | rd  | opcode  |
| :-------: | :-----------: | :-: | :----: | :-: | ------- |
|    lb     |               |     |  000   |     | 0000011 |
|    lh     |               |     |  001   |     | 0000011 |
|    lw     |               |     |  010   |     | 0000011 |
|    lbu    |               |     |  100   |     | 0000011 |
|    lhu    |               |     |  101   |     | 0000011 |

funct3 中最高位决定加载的数据是否有符号，其余两位决定加载的数据大小：

- \_00: b
- \_01: h
- \_10: w

lbu表示加载无符号字节。lh表示加载半个字，即加载16位后作符号扩展填充32位。此外，lwu是没有意义的。

## S-Format Layout

`31 25|24 20|19 15|14 12|11 7|6 0`

| imm\[11 : 5\]  | rs2 | rs1  | funct3 | imm\[4 : 0\]  | opcode |
| :------------: | :-: | :--: | :----: | :-----------: | :----: |
|       7        |  5  |  5   |   3    |       5       |   7    |
| offset\[11:5\] | scr | base | width  | offset\[4:0\] | STORE  |

- STORE opcode: 0100011
- 存储需要读取两个寄存器，rs1为基内存地址，rs2为要存储的数据，以及立即偏移量
- 注意，存储不会向寄存器写入值，所以没有rd
- RISC-V 的设计决策是将低五位的立即数放在rd字段所在的位置，为了保持 rs1/rs2 字段在相同的位置
	- 在硬件设计中，寄存器名比立即数的位更重要

### All RV32 Store Instructions

| operation | imm\[11 : 5\] | rs2 | rs1 | funct3 | imm\[4 : 0\] | opcode  |
| :-------: | :-----------: | :-: | :-: | :----: | :----------: | :-----: |
|    sb     |               |     |     |  000   |              | 0100011 |
|    sh     |               |     |     |  001   |              | 0100011 |
|    sw     |               |     |     |  010   |              | 0100011 |
