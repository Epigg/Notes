# Single-Cycle CPU Datapath II

*November 29, 2024*

## Supporting Loads

- R+I Arithmetic/Logic Datapath

![[datapath3.png]]

### Add `lw` 

[[Lec 11#RISC-V Loads]]

12位有符号立即数与rs1中的基地址相加得到内存地址，这与立即加法操作很相似，但是得到的是地址而不是最终结果。从地址加载的数据将被保存在寄存器rs。

- 加入数据内存和多路复用器
- 执行 `lw` 指令时，`ImmSel=I, MemRW=Read, WBSel=0` 

![[datapath4.png]]

- 支持较窄的加载需要额外的逻辑，以便从内存加载的值中提取正确的字节/半字，并在将结果写回寄存器文件之前将其符号或零扩展到 32 位
	- funct3 字段编码了加载数据的大小和符号
- 它只是一个多路复用器和几个门电路

## Datapath for Stores

[[Lec 11#S-Format Layout]]

### Add `sw` 

- rs2为要保存到内存的数据
- 执行 `sw` 指令时，控制信号的输出如图所示

![[datapath5.png]]

### I+S Immediate Generation

- 只需要添加一个五位的多路复用器根据指令的类型选择立即数的最后五位
- 立即数中的其他位与指令中的固定位置联系

![[generation1.png]]

## Implementing Branches

[[Lec 12#RISC-V B-Format for Branches]]

### Add Branches

- PC的状态改变规则改变了
	- `PC = PC + 4, branch not taken` 
	- `PC = PC + immediate, branch taken` 
- 需要计算 `PC+immediate` 并且比较 `rs1` 和 `rs2` 值
	- 但是只有一个ALU - 添加其他硬件来比较
- 执行 B 格式指令时，控制信号的输出如图所示

![[datapath6.png]]

分支比较器（Branch Comparator）

- A/B: 输入
- BrUn: 是否是无符号比较
- BrLT: 如果 `A<B` 输出1
- BrEq: 如果 `A=B` 输出1

### Branch Immediates

一种做法：把分支指令的立即数看作是将保存指令的立即数左移一位，因此每个指令的立即数输入的每一位都可能出现的立即数输出的两个位其中之一，所以每一位需要一个2路复用器。

RISC-V的做法：保持立即数输入的11个位位置固定，这样相较S格式，B格式立即数只有一个位的位置改变，所以只需要一个2路复用器。

[[Lec 12#RISC-V Immediate Encoding]]

## Adding `jal` and `jalr`

[[Lec 12#J-Format]]

- 增加 `jalr` 指令（I格式）
- 加宽选择写入寄存器数据的多路复用器

![[datapath7.png]]

- 更新立即数生成器的功能，使其能根据J格式指令的生成对应的立即数
- 发现了吗，除此之外和jalr是一模一样的

![[datapath8.png]]

## Adding U-Types

[[Lec 12#U-Format for “Upper Immediate” Instructions]]

- 类似地，只需要更新立即数生成器的功能
- 但是在控制信号上与J格式不尽相同
	- 立即数作为高位写入寄存器
	- ALU不进行运算，立即数**直接**作为其输出（ALUSel=B）

![[datapath9.png]]

## Single-Cycle RV32I Datapath 

![[datapath10.png]]
