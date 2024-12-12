# Single-Cycle CPU Datapath I

*November 28, 2024*

## RISC-V Processor Design

### Machine Structures

![[Machine Structures.png]]

### Great Idea #1: Abstraction (Levels of Representation/Interpretation)

- High Level Language Program (e.g., C)
- Assembly Language Program (e.g., RISC-V)  
- Machine Language Program (RISC-V) 
- Hardware Architecture Description (e.g. block diagrams)  ==**<--**==
- Logic Circuit Description (Circuit Schematic Diagrams)  ==**<--**==

### Single-Core Processor 

![[processor.png]]

### The CPU

- 处理器（CPU）：计算机的核心活动部件，负责执行所有运算工作（数据处理和决策）
- 数据通路（**Datapath**）：处理器中包含执行各种必要运算的硬件部分（可以理解为"力量"部分）
- 控制单元（Control）：处理器中的另一个硬件部分，负责告诉数据通路需要执行什么操作（可以理解为"大脑"部分）

## Building a RISC-V Processor

- 在每个时钟周期，计算机执行一条指令
- 当前状态的输出驱动组合逻辑的输入，组合逻辑的输出会在下一个时钟边沿到来之前稳定在相应的状态值
- 在时钟上升沿，所有状态元件都会根据组合逻辑的输出进行更新，然后执行进入下一个时钟周期

### Five Stages of the Datapath

存在一个问题，即“执行指令”（从获取指令开始执行所有必要的操作）的单一 “整体 ”模块过于庞大，效率低下。

解决方案是将 “执行指令 ”的过程分解成若干阶段，然后将这些阶段连接起来，创建整个数据通路。因为较小的阶段更易于设计，而且易于优化一个阶段而不会影响其他阶段，即模块化（modularity）。

执行指令的五个阶段：

- 指令获取 (Instruction Fetch, IF) 
- 指令解码 (Instruction Decode, ID)
- 执行 (Execute, EX) - ALU 
- 内存访问 (Memory Access, MEM)
- 写回寄存器 (Write Back to Register, WB)

![[Instruction Execution.png]]

### Datapath Elements: State and Sequencing

- 写入使能（Write Enable）
	- 无效电平（deasserted）： 数据输出不会发生变化
	- 有效电平（asserted）：数据输出将在时钟上升沿变为数据输入
- 寄存器文件（regfile，RF）由 32 个寄存器组成
	- 两条输出总线（buses）：总线 A 和总线 B
	- 一条输入总线：总线 W
- 寄存器选择
	- RA（编号）选择要放在总线 A（数据）上的寄存器
	- RB（编号）选择要放在总线 B（数据）上的寄存器 
	- RW（编号）选择在写入使能为 1 时通过总线 W（数据）写入的寄存器
- 时钟输入（Clk）
	- Clk 输入仅在写入操作期间起作用 
	- 在读取操作期间，行为表现为组合逻辑块
		- RA 或 RB 有效 ⇒ 总线 A 或总线 B 在“访问时间”（即上升沿）后有效

![[regfile.png]]

- 内存
	- 一条输入总线：Data In
	- 一条输出总线：Data Out
- 内存中的字
	- 读取时，地址选择要放在数据输出总线上的字 
	- 写入时，设置写使能为1：地址选择要通过数据输入总线写入的内存字

![[Memory.png]]

## State Required by RV32I ISA

每个指令在执行过程中会读取并更新寄存器、PC和内存的状态。

- 寄存器（x0 ..x31）
	- 寄存器文件 Reg 保存32个32位的寄存器 Reg\[0\] ..Reg\[31\]
	- 第一个读入的寄存器由指令中的rs1字段指定
	- 第二个读入的寄存器由指令中的rs2字段指定
	- 写入的目标寄存器由指令中的rd字段指定
	- x0总是0（写入被忽略）
- 程序计数器（PC）
	- 保存当前指令的地址
- 内存（MEM）
	- 在一个 32 位字节寻址的内存空间中同时保存指令和数据
	- 将内存分为两部分，分别为指令内存（IMEM）和数据内存（DMEM）
	- 从指令存储器读取指令（假设 IMEM 只读）
	- 通过内存加载和保存指令

## R-Type Datapath

[[Lec 11#R-Format Layout|R-type Instructions]]

### Add Datapath

指令 `add rd. rs1, rs2` 改变了两个机器状态

-  `Reg[rd] = Reg[rs1] + Reg[rs2]`
-  `PC = PC + 4` 

![[datapath1.png]]

![[add waveforms.png]]

### Sub Datapath 

sub指令和add几乎一模一样，通过 `inst[30]` 从加法和减法选择

![[datapath2.png]]

> 全部的指令都是通过解码 funct3 和 funct7 字段并选择适当的 ALU 函数来实现

## Datapath With Immediates

- 新增了立即数生成器与选择rs2和imm的多路复用器
- 执行的过程需要经过决策，这些结果由控制逻辑通过指令得到
- 不要求的行为，相关的通路怎么运作不重要

![[datapath3.png]]