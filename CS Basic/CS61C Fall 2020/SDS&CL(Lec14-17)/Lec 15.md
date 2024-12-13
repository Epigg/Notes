# State

*November 28, 2024*

## Register Details, Flip-flops

### State Elements

- 作为存储值的地方
	- 寄存器文件（如 RISC-V 的 x0-x31）
	- 内存（缓存和主内存）
- 帮助控制组合逻辑块之间的信息流
	- 状态元件用于保持输入到组合逻辑块的信息的流动，允许信息有序地传递

### Registers and FIip-flops

![[register.png]]

由n个触发器构成，也称作D型触发器。图中Q为quiescent，静态的

当时钟的上升沿到来，输入 d 被采样并作为输出，注意存在延迟(*delay*)。除此之外的任何时间，输入 d 都被无视。

![[FF waveform.png]]

## Accumulator

- 寄存器用于暂存数据，控制数据向加法器的传输
- 寄存器的重置（Reset）输入用于强制其输出为0（优先级高于输入d）
	- 当时钟的上升沿到来，如果 reset 为1，则输出重置为0

![[Accumulator.png]]

- 在实践中，输入 x 通常不会和输出 $S_{i-1}$ 同时到达加法器
- 所以 $S_i$ 在一段时间内是错误的，但是寄存器总会捕获正确的值
- 在好的电路中，时钟的上升沿附近不会不存在不稳定的状态

![[Accumulator waveform.png]]

## Pipelining for Performance

- 时钟的周期要小于电路中的最大延迟
- 时钟周期由加法器/移位器的传播延迟所限制
- 通常会添加额外的寄存器来提高时钟频率
	- 相同时间内的输出更多

### Recap of Timing Terms

- 时钟（CLK） - 稳定的方波，用于同步系统
- 设定时间（Setup Time） - 输入必须在时钟上升沿*之前*保持稳定的时间
- 保持时间（Hold Time） - 输入必须在时钟上升沿*之后*保持稳定的时间
- “CLK到Q”延迟（CLK-to-Q Delay） - 从时钟上升沿开始到输出发生变化所需的时间
- 触发器（Flip-flop） - 一位状态，每次时钟上升沿时进行采样（正沿触发）
- 寄存器（Register） - 多位状态，在时钟上升沿或加载信号（LOAD）时进行采样（正沿触发）

## Finite State Machines

有限状态机（FSM）的功能可以由*状态转换图*表示。结合组合逻辑和寄存器，可以在硬件上实现任何有限状态机。

![[state transition diagram.png]]

假设状态转换由时钟控制：在每个时钟周期内，机器检查输入，进入新的状态并生成新的输出。

### Hardware Implement

- 一个寄存器来保存机器当前所处的状态
	- 每个状态使用唯一的位模式
- 组合逻辑电路用于实现将输入和当前状态（PS）输入映射到下一状态（NS）和输出的功能

### General Model for Synchronous Systems

![[Synchronous Systems.png]]

- 由寄存器分隔的组合逻辑（CL）模块构成的集合
- 寄存器可以是紧接的，组合逻辑模块（CL）也可以是紧接的
- 可以存在反馈
- 时钟信号只连接到寄存器的时钟输入端

## In conclusion

- 状态元件用于
	- 构建内存
	- 控制其他状态元件和组合逻辑之间的信息流动
- D触发器（D-flip-flops）用于构建寄存器
- 时钟信号决定了D触发器何时发生变化
- 对于长延迟的组合逻辑，使用流水线技术以实现更快的时钟
- 有限状态机
