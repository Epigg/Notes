# Single-Cycle CPU Control

*November 29, 2024*

## Control and Status Registers

- 控制与状态寄存器（CSR）与寄存器文件（x0-x31）是分离的
	- 用于监控状态和性能
	- 使用专有的12位地址编码空间，最多可有 4096 个
- 不在基本 ISA 中，但几乎在每个实现中都使用
	- ISA 是模块化的
	- 计数器和定时器以及与外设通信所必需的

### CSR Instructions

`31 20|19 15|14 12|11 7|6 0`

|     csr     |        rs1         | funct3 | rd  | opcode |
| :---------: | :----------------: | :----: | :-: | :----: |
|     12      |         5          |   3    |  5  |   7    |
| source/dest | source/uimm\[4:0\] | instr  | rd  | SYSTEM |

- SYSTEM opcode: 1110011
- uimm 通过零扩展为32位

寄存器指令

|  instr  | rd  | rs  | Read CSR? | Write CSR? |
| :-----: | :-: | :-: | :-------: | :--------: |
|  csrrw  | x0  |  -  |    no     |    yes     |
|  csrrw  | !x0 |  -  |    yes    |    yes     |
| csrrs/c |  -  | x0  |    yes    |     no     |
| csrrs/c |  -  | !x0 |    yes    |    yes     |

立即数指令

|  instr   | rd  | uimm | Read CSR? | Write CSR? |
| :------: | :-: | :--: | :-------: | :--------: |
|  csrrwi  | x0  |  -   |    no     |    yes     |
|  csrrwi  | !x0 |  -   |    yes    |    yes     |
| csrrs/ci |  -  |  0   |    yes    |     no     |
| csrrs/ci |  -  |  !0  |    yes    |    yes     |

- CSRRW（原子读/写 CSR）指令 "原子 "交换 CSR 和整数寄存器中的值
- CSRRW 读取 CSR 的值，并将其写入整数寄存器 rd，然后将 rs1 写入 CSR
- 伪指令 `csrw csr, rs1` 是` csrrw x0, csr, rs1`
	- `rd=x0`, 只需将 `rs1` 写入 CSR
- 伪指令 `csrwi csr, uimm` 是 `csrrwi x0，csr，uimm`
	- `rd=x0`, 只需将 `uimm` 写入 CSR
- 使用于写入使能和时钟...

### System Instructions

`31 20|19 15|14 12|11 7|6 0`

|   funct12    | rs1 | funct3 | rd  | opcode |
| :----------: | :-: | :----: | :-: | :----: |
|      12      |  5  |   3    |  5  |   7    |
| ECALL/EBREAK |  0  |  PRIV  |  0  | SYSTEM |

- `ecall` -（I格式）向操作系统发出请求，如系统调用（syscalls）
- `ebreak` -（I格式）用于如调试器，将控制权转移到调试环境
- `fence` - 按照其他线程或协处理器的视图对内存（和 I/O）访问进行排序

## Instruction Timing

![[timing.png]]

|   IF   |    ID    |   EX   |  MEM   |   WB   | Total  |
| :----: | :------: | :----: | :----: | :----: | :----: |
| I-MEM  | Reg Read |  ALU   | D-MEM  | Reg W  |        |
| 200 ps |  100 ps  | 200 ps | 200 ps | 100 ps | 800 ps |

| Instr | IF  | ID  | EX  | MEM | WB  |   Total   |
| :---: | :-: | :-: | :-: | :-: | :-: | :-------: |
|  add  |  x  |  x  |  x  |     |  x  |   600ps   |
|  beq  |  x  |  x  |  x  |     |     |   500ps   |
|  jal  |  x  |  x  |  x  |     |  x  |   600ps   |
|  lw   |  x  |  x  |  x  |  x  |  x  | ==800ps== |
|  sw   |  x  |  x  |  x  |  x  |     |   700ps   |


- 最大的时钟频率取决于指令的最长执行时间
	- $f_{max}=1/800ps=1.25GHz$ 
- 大多数区块大部分时间处于闲置状态

## Control Logic Design

### Control Logic Truth Table

![[CL truthtable.png]]

### Control Realization Options

- 只读寄存器（Read-Only Memory, ROM ）
	- 结构有规则
	- 可轻松重新编程
		- 修复错误
		- 添加指令
	- 在手动设计控制逻辑时很受欢迎
- 组合逻辑（Combinatorial Logic）
	- 如今，芯片设计人员使用逻辑合成工具将真值表转换为门网络

### ROM Controller

RV32I 是一个9位的指令集，指令的类型由 `inst[30], inst[14:12], inst[6:2]` 编码

![[ROM control.png]]

十一位输入通过一个解码器，得到（选择）唯一的一个指令类型；每个指令类型都对应着一个有十五位输出的控制

### Combinational Logic Control

pass...
