# RISC-V Instruction Formats II

*November 24, 2024*

## B-Format Layout

例如条件分支 `beq x1, x2, Label` ，分支读取两个寄存器，但不写入寄存器（类似于存储）。

- 分支通常用于循环
	- 循环通常较小（小于50条指令）
	- 函数调用和非条件跳转用跳转指令处理（J-Format）
- 指令存储在内存的局部区域，即 Code/Text
	- 最长分支距离收代码的大小限制
	- 当前指令的地址存储在程序计数器（PC）

PC间接寻址(PC-Relative Addressing)，即使用立即数字段作为PC的补码偏置

- 分支通常在小范围内改变PC
- 可以指定自PC开始的 $±2^{11}$ 内的“单元”地址
- 将12位偏移量编码为立即数

有一个想法是，为了增长单个分支指令覆盖的范围，在把偏移量加到PC前先乘一个系数

所有关于分支的计算：

- 如果没有采取分支：
	  PC = PC + 4
- 如果采取分支：
	 PC = PC + immediate \* ~~4~~ **2** 

系数取为32位指令的大小，即4，是合适的，吗？

- RISC-V 基本指令集体系结构（ISA）的扩展支持 16 位压缩指令，以及长度为 16 位倍数的可变长度指令
- 为了实现这点，RISC-V 将分支的偏移量扩大到两倍，及时不涉及到16位指令
- 减小一半的分支范围，意味着一半的可能目标在只支持32位指令的RISC-V处理器上会出错
- 所以，RISC-V条件分支能到达自PC开始的 $±2^{10}$ 条32位的指令

### RISC-V B-Format for Branches

`31 |30 25|24 20|19 15|14 12|11 8| 7 |6 0`

| imm\[12\] | imm\[10 : 5\] | rs2 | rs1 | funct3 | imm\[4 : 1\] | imm\[11\] | opcode |
| :-------: | :-----------: | :-: | :-: | :----: | :----------: | :-------: | :----: |
|     1     |       6       |  5  |  5  |   3    |      4       |     1     |   7    |

- B格式和S格式很相似，都有两个源寄存器 rs1/rs2 和12位的立即数 imm
- 但是B格式的立即数的范围为 -4096 到 4094
- 机器指令中存储的是立即数的1-12位，第0位总是为0

Example: 

```
Loop:
	beq x19,x10,End 
	add x18,x18,x10
	addi x19,x19,-1 
	j Loop 
End: # target instruction
```

|     0     |   000000    | 01010  | 10011  | 000 |    0100    |     0     | 1100011 |
| :-------: | :---------: | :----: | :----: | :-: | :--------: | :-------: | :-----: |
| imm\[12\] | imm\[10:5\] | rs2=10 | rs1=19 | BEQ | imm\[4:1\] | imm\[11\] | BRANCH  |

分支的偏移量为 4 × 32-bit，即16字节，不考虑imm\[0\]，指令中的imm相当于8

### RISC-V Immediate Encoding

指令的编码，inst\[31:0\]：

| Type |     31  25      |   24  20   | 19  15 | 14  12 |     11  7      |  6  0  |
| :--: | :-------------: | :--------: | :----: | :----: | :------------: | :----: |
|  R   |     funct7      |    rs2     |  rs1   | funct3 |       rd       | opcode |
|  I   |   imm\[11:5\]   | imm\[4:0\] |  rs1   | funct3 |       rd       | opcode |
|  S   |   imm\[11:5\]   |    rs2     |  rs1   | funct3 |   imm\[4:0\]   | opcode |
|  B   | imm\[12\|10:5\] |    rs2     |  rs1   | funct3 | imm\[4:1\|11\] | opcode |

32位立即数，imm\[31:0\]：

| Type |    31  12    | 11         |     10  5     |     4  1      |     0      |
| :--: | :----------: | :--------: | :-----------: | :-----------: | :--------: |
|  I   | -inst\[31\]- | inst\[31\] | inst\[30:25\] | inst\[24:21\] | inst\[20\] |
|  S   | -inst\[31\]- | inst\[31\] | inst\[30:25\] | inst\[11:8\]  | inst\[7\]  |
|  B   | -inst\[31\]- | inst\[7\]  | inst\[30:25\] | inst\[11:8\]  |     0      |

- 其中，12到31为算数扩展的结果
- 比较S格式和B格式只有指令的第7位改变了角色

### All RISC-V Branch Instructions

| operation | imm\[12\|10 : 5\] | rs2 | rs1 | funct3 | imm\[4 : 1\|11\] | opcode  |
| :-------: | :---------------: | :-: | :-: | :----: | :--------------: | :-----: |
|    beq    |                   |     |     |  000   |                  | 1100011 |
|    bne    |                   |     |     |  001   |                  | 1100011 |
|    blt    |                   |     |     |  100   |                  | 1100011 |
|    bge    |                   |     |     |  101   |                  | 1100011 |
|   bltu    |                   |     |     |  110   |                  | 1100011 |
|   bgeu    |                   |     |     |  111   |                  | 1100011 |

是否有符号体现在 funct3 字段的第二位

## Long Immediates

当我们移动代码时，分支的立即数字段是否会改变。如果移动单独的代码行，答案是是；如果移动所有的代码，那么答案是否

如果目标指令与分支指令的距离大于 $2^{10}$ 条指令，需要使用跳转指令：

```
  beq x10, x0, far
  # next instr
```

```
  beq x10, x0, next
  j far
next:
  # next instr
```

### U-Format for “Upper Immediate” Instructions

`31 12|11 7|6 0`

|    imm\[31 : 12\]    |  rd  |    opcode     |
| :------------------: | :--: | :-----------: |
|          20          |  5   |       7       |
| U-immediate\[31:12\] | dest |  LUI/0110111  |
| U-immediate\[31:12\] | dest | AUIPC/0010111 |

- 32位的指令的上20位存储20位的立即数
- 一个目标寄存器rd
- U格式用于两种指令
	- lui: load upper immediate
	- auipc: add upper immediate to PC

#### LUI

- LUI将立即数写入目标寄存器的上20位，并清除下12位
- 用addi设置下12位，结合二者将任意32位值写入一个寄存器

如何设置0xDEADBEEF

```
lui x10, 0xDEADB      # x10 = 0xDEADB000 
addi x10, x10, 0xEEF  # x10 = 0xDEADAEEF
```

注意，addi的12位立即数总是符号扩展的。应将lui的立即数设为 0xDEADC

汇编器使用伪代码实现设置的操作：

`li x10, 0xDEADBEEF` （若imm没有超出12位，那么不会被lui和addi两个指令替换）

#### AUIPC

- 将立即数直接添加到PC并将结果放入目标寄存器  
- 用于PC相对寻址

```
Label: AUIPC x10, 0  # Puts address of Label in x10
```

## J-Format

`31 |30 21| 20 |19 12|11 7|6 0`

| imm\[20\] | imm\[10 : 1\] | imm\[11\] | imm\[19 : 12\] |  rd  |   opcode    |
| :-------: | :-----------: | :-------: | :------------: | :--: | :---------: |
|     1     |      10       |     1     |       8        |  5   |      7      |
|           |               |           |                | dest | JAL/1101111 |

上20位存储 offset\[20:1\]

### JAL

`jal rd, Label`

- 将 PC+4 保存在寄存器 rd 中
- 赋值 PC = PC + offset (PC相对的跳转PC-relative jump)
- 目标在 $±2^{19}$ 个位置之内，每个位置2字节
	- 能到达 $±2^{18}$ 条 32 位的指令
- 类似于B格式，立即数的0位固定为0

> [!note] Uses
> - j 是伪指令，`j Label = jal x0, Label` 
> - 可以用 jal 调用距离PC $2^{18}$ 条32位指令内的函数

### JALR Instruction(I-Format)

`31 20|19 15|14 12|11 7|6 0`

| imm\[11 : 0\]  | rs1  | funct3 |  rd  |    opcode    |
| :------------: | :--: | :----: | :--: | :----------: |
|       12       |  5   |   3    |  5   |      7       |
| offset\[11:0\] | base |   0    | dest | JALR/1100111 |

`jalr rd, rs1, imm`

- 将 `PC+4` 保存在寄存器 rd 中
- 赋值 `PC = rs1 + imm` 
- 使用与算术操作和加载相同格式的立即数
	- **没有**2倍的扩大
	- 不同于分支和 `jal` 

> [!note] Uses
> - ret 和 jr 是伪指令
> 	`ret = jr ra = jalr x0, ra, 0` 
> - 用32位绝对地址调用函数
> 	`lui x1, <hi20bits>`
> 	`jalr ra, x1, <lo12bits>`
> - 根据32位偏移量相对跳转PC
> 	`auipc x1, <hi20bits>`
> 	`jalr x0, x1, <lo12bits>`

## Summary of RISC-V Instruction Formats

| Type |     31  25      |   24  20   | 19  15 | 14  12 |     11  7      |  6  0  |
| :--: | :-------------: | :--------: | :----: | :----: | :------------: | :----: |
|  R   |     funct7      |    rs2     |  rs1   | funct3 |       rd       | opcode |
|  I   |   imm\[11:5\]   | imm\[4:0\] |  rs1   | funct3 |       rd       | opcode |
|  S   |   imm\[11:5\]   |    rs2     |  rs1   | funct3 |   imm\[4:0\]   | opcode |
|  B   | imm\[12\|10:5\] |    rs2     |  rs1   | funct3 | imm\[4:1\|11\] | opcode |

| Type |       31  20        |    19  12    | 11  7 |  6  0  |
| :--: | :-----------------: | :----------: | :---: | :----: |
|  U   |    imm\[31:20\]     | imm\[19:12\] |  rd   | opcode |
|  J   | imm\[20\|10:1\|11\] | imm\[19:12\] |  rd   | opcode |
