# Introduction to Synchronous Digital Systems

*November 27, 2024*

### Great Idea #1: Abstraction (Levels of Representation/Interpretation)

- High Level Language Program (e.g., C)
- Assembly Language Program (e.g., RISC-V)  
- Machine Language Program (RISC-V) 
- Hardware Architecture Description (e.g. block diagrams)  ==**<--**==
- Logic Circuit Description (Circuit Schematic Diagrams)  ==**<--**==

### Synchronous Digital Systems

处理器硬件是一个同步数字系统。

- Synchronous: 所有操作由中央时钟协调
- Digital: 用离散值表示的所有值；电信号按1和0处理；组合在一起形成字

## Switches

pass...

## Transistor

晶体管，放大或转换信号的半导体装置。现代数字系统设计于互补金属氧化物半导体（CMOS, Complementary Metal-Oxide on Semiconductor）。MOS 晶体管充当电压控制开关。

### MOS Transistors

三个端：漏极(drain)、栅极(gate)、源极(source)。

N型（P型）：如果栅极端的电压高于（低于）源端，则在漏极和源端之间建立导通通道。

![[MOS.png]]

### Transistor Circuit Rep. vs. Block diagram

- 芯片只由晶体管和电线组成
- 一小组晶体管构成有用的块
- 块被组织成一个层次结构来构建更高级的块，如加法器
- 可以用与非门构建与门、或门和非门

![[NAND.png]]

## Signals and Waveforms

1. Clocks

![[Clock.png]]

2. Signals
	- 数字仅被视为 1 或 0 
	- 通过电缆持续传输
	- 传输几乎是即时的
	- 一根电缆在任何时候只包含一个值

3. Waveforms
4. Circuit Delay
	- 如加法器在输入和得到结果间存在延迟

---

同步数字系统由两种基本类型的电路组成：  

- 组合逻辑（Combinational Logic, CL）电路
	- *输出仅仅是输入的函数*
	- 类似于数学中的纯函数，y = f(x)
	- 在两次调用之间没有存储信息；没有副作用

- 状态元件（State Elements）
	- 用于存储信息的电路

## In conclusion

- 时钟信号是电路的心跳
- 电压是模拟信号，但被量化为 0/1  
- 电路延迟是不可避免的  
- 电路有两种类型：
	- 无状态组合逻辑（例如：与、或、非）
	- 有状态电路（例如：寄存器）
