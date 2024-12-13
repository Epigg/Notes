# Pipelining I

*December 6, 2024*

## Pipelining

### Great Ideas in Computer Architecture

1. Abstraction (Layers of Representation/Interpretation) 
2. Moore’s Law 
3. Principle of Locality/Memory Hierarchy 
4. Parallelism 
5. Performance Measurement and Improvement ==**<--**==
6. Dependability via Redundancy

[[Lec 20#Instruction Timing]] 

## Processor Performance Iron Law

$$
\frac{时间}{程序}=\frac{指令}{程序}*\frac{时钟周期}{指令}*\frac{时间}{时钟周期} 
$$

决定每个程序的指令数（Instructions per Program）的因素：

- 任务
- 算法
- 编程语言
- 编译器
- 指令集架构（ISA）

决定每个指令的（平均）时钟周期数（Cycles per Instruction, CPI）的因素：

- 处理器实现（或微体系结构）
- 例如，对于单周期 RISC-V 设计，CPI = 1
- 复杂指令（如 `strcpy` ），CPI >> 1
- 超标量处理器，CPI < 1

决定每时钟周期的时间（Time per Cycle）的因素：

- 处理器微架构（通过逻辑门确定关键路径）
- 技术（如 5 纳米与 28 纳米）
- 功耗预算（较低的电压会降低晶体管速度）

## Energy Efficiency

$$
\frac{能耗}{程序}=\frac{指令}{程序}*\frac{能耗}{指令}
$$

其中每指令的能耗取决于 CMOS 中电容器的充放电，由公式 $CV^2$ 求得。所以我们想通过降低电容值和电源电压来降低能耗。

#### Energy “Iron Law”

$$
性能=功率*能效
$$

- 能效（如指令/焦耳）是所有计算设备的关键指标
- 对于功率受限的系统（如 20MW 数据中心），需要更高的能效才能在相同功率下获得更高的性能
- 对于能耗受限的系统（如 1W 的手机），需要更高的能效以延长电池使用时间
