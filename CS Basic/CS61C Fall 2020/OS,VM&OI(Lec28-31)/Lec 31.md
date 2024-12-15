# I/O

*December 15, 2024*

## I/O Devices

- 需要与键盘、网络、鼠标、显示器等设备交互
- 连接多种类型的设备
- 控制设备、响应设备、传输数据
- 向用户程序提供有用的接口

### ISA for I/O

处理器任务

- 输入：读取字节序列
- 输出：写入字节序列

接口选项
1. 特殊的输入/输出指令和硬件
2. 内存映射I/O：
    - 地址空间的一部分专用于I/O
    - I/O设备寄存器位于该空间（非内存）
	    - 即某些地址不是常规内存，它们与I/O设备的寄存器相对应
    - 使用普通的load/store指令（如lw/sw）
    - RISC-V采用这种方式
 
## I/O Polling

### Polling

轮询：处理器检查状态，然后采取行动

- 设备寄存器一般有两种功能
	- 控制寄存器：指示是否可以读/写（I/O就绪）
	- 数据寄存器：包含实际数据

轮询过程：

- 处理器循环读取控制寄存器
	- 等待设备设置就绪位(0→1)
- 处理器进行数据读取或写入
- I/O设备重置控制寄存器位(1→0)

### I/O Example (Polling)

输入：从键盘读取到a0

```
lui t0 0x7ffff       # I/O base addr
Waitloop: 
    lw t1 0(t0)      # read control
    andi t1 t1 0x1   # ready bit
    beq t1 zero Waitloop
    lw a0 4(t0)      # data
```

输出：从a1写入显示器

```
lui t0 0x7ffff
Waitloop: 
    lw t1 8(t0)      # write control
    andi t1 t1 0x1   # ready bit
    beq t1 zero Waitloop
    sw a1 12(t0)     # data
```

内存映射：

```
7ffff000 input ctrl reg
7ffff004 input data reg
7ffff008 output ctrl reg
7ffff00c output data reg
```

### Cost of Polling

处理器参数假设：

- 处理器时钟频率：1 GHz
- 每次轮询操作：400个时钟周期

示例：
1. 鼠标轮询
    - 频率：30次/秒
    - 时钟周期占用：12K cycles/s
    - CPU占用率：0.0012%
    - 结论：对处理器影响很小

2. 硬盘轮询
    - 数据速率：16 MB/s
    - 每次轮询：16 B
    - 轮询频率：1M 次/秒
    - 时钟周期占用：400M cycles/s
    - CPU占用率：40%
    - 结论：开销不可接受

## I/O Interrupts

### Alternatives to Polling: Interrupts

一种硬件机制，允许I/O设备通知处理器需要注意的事件

- 轮询浪费处理器资源
- 类比：等待客人vs门铃通知

- I/O设备需要注意时触发中断
	- 中断当前程序
	- 转移控制权到操作系统的trap处理程序

中断使用策略
- 低数据率设备（如鼠标、键盘）：
    - 使用中断
    - 中断开销较低
- 高数据率设备（如网络、磁盘）：
    - 开始时使用中断
	    - 如果没有数据就什么都不做
    - 数据传输开始后切换到DMA

# Direct Memory Access

- 允许I/O设备直接读写主存
- 新增硬件：DMA引擎
- DMA引擎包含的寄存器信息：
    - 内存地址
    - 字节数
    - I/O设备号
    - 传输方向
    - 传输单位和突发传输量

![[DMA transfer.png]]

### Incoming Data

1. 设备发送中断
2. CPU响应中断，初始化传输
3. DMA引擎执行传输
4. 传输完成后再次中断CPU

### Outgoing Data

1. CPU确认设备就绪
2. CPU初始化传输
3. DMA引擎执行传输
4. 传输完成后中断CPU

### Some New Problems

- DMA引擎在内存层次结构中的位置选择：
  1. L1缓存和CPU之间：
     - 优点：自动保持一致性
     - 缺点：传输数据污染CPU工作集
  2. 末级缓存和主存之间：
     - 优点：不影响缓存
     - 缺点：需要显式地管理一致性
