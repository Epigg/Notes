# Virtual Memory II

*December 15, 2024*

## Hierarchical Page Tables

### Size of Page Tables

假设32位虚拟地址，页面大小为4 KiB，那么单个页表大小为$4 \times 2^{20}$ 字节 = 4 MiB，占4 GiB内存的0.1%。假设有256个进程，那么总页表大小为256 × 4 MiB = 1 GiB，占4 GiB内存的25%。64位地址系统会需要更大的页表空间。

如何让页表大小保持合理：

1. 增加页面大小
- 页面大小翻倍，页表大小减半
- 可能造成内存浪费
- 内部碎片增加

2. 层次化页表
- 页面大小逐级减小
- 大多数程序只使用部分内存空间
- 将页表分成两级或多级
- RISC-V采用这种方式

### Hierarchical Page Table

```
[VPN[1](10-bits)|VPN[0](10-bits)|offset(12-bits)]
```

- 处理器寄存器保存当前页表的基地址
	- RISC-V中的监督页表基寄存器
	- L1 index 10位寻址
- 第一级页表（Level 1）
    - L2 index 10位寻址
- 第二级页表（Level 2）
    - offset 1
- 数据页面

### Example: 32-b RISC-V

- VPN: Virtual Page Number
- PPN: Physical Page Number

```
[VPN[1](10-bits)|VPN[0](10-bits)|offset(12-bits)] (32 bits)
```

```
[PPN[1](12-bits)|PPN[0](10-bits)|offset(12-bits)] (34 bits)
```

页表条目(PTE)为32位：
- 物理页号
	- PPN\[1\] 12位
	- PPN\[0\] 10位
- 状态位 10位
	- 读(R)、写(W)、执行(X)权限
	- 有效位(V)
	- 访问位(A)
	- 脏位(D)
	- 全局位(G)
	- 用户位(U)

若R=0, W=0, X=0，该条目指向下一级页表，否则为叶页表条目。

## Translation Lookaside Buffers

- 地址转换开销大：
    - 单级页表：每次引用需要2次内存访问
    - 两级页表：每次引用需要3次内存访问
- TLB作为解决方案：
    - 缓存页表中常用的地址转换
    - TLB命中→单周期转换
    - TLB未命中→需要进行页表遍历来转换

### TLB Designs

- 典型容量：32-128个表项
- 通常采用全相联方式
    - 每个表项映射一个大页面
    - 页面间空间局部性较少
    - 表项冲突概率更高
- 大型系统特点：
    - TLB可能有256-512个表项
    - 采用4-8路组相联
    - 可能有多级TLB(L1和L2)
- 替换策略：
    - 随机替换
    - FIFO(先进先出)
- TLB覆盖范围指TLB能同时映射的最大虚拟地址空间

![[TLB.png]]

- 不同进程的TLB条目不同
- 在上下文切换时将所有TLB项设为无效

## TLBs in Datapath

### VM-related Events in Pipeline

- TLB miss
	- 可通过硬件或软件机制重填TLB
	- 通常由硬件完成
- Page fault（如页面在磁盘上）
	- 需要precise trap
		- 在trap的指令之前的指令都已完成，之后的指令都没有执行
	- 这样软件处理程序在获取页面后容易恢复
	- 保护违规可能导致进程中止

### Page-Based Virtual-Memory Machine

![[TLB datapath.png]]

### Address Translation

![[address translation.png]]

## VM Performance

|                |             Cache version              | Virtual Memory version |
| :------------: | :------------------------------------: | :--------------------: |
|      Unit      |               Block/Line               |          Page          |
|    Failure     |                  Miss                  |       Page Fault       |
|      Size      |           Block Size: 32-64B           |   Page Size: 4K-8KiB   |
|   Placement    | Direct Mapped<br>N-way Set Associative |    Full Associative    |
|  Replacement   |             LRU or Random              |   LRU, FIFO, random    |
| Write Strategy |      Write Through<br>Write Back       |       Write Back       |

- 虚拟内存是层次结构中主存之下的一层
- 适用相同的CPI，AMAT等，只不过现在将主存视为中间的缓存

### Impact of Paging on AMAT

典型性能：

|              |  Caching   | Demand paging |
| :----------: | :--------: | :-----------: |
|   hit time   |  ~1 cycle  |  ~100 cycle   |
| miss penalty | ~100 cycle |   ~5M cycle   |
|  miss rate   |   1-20%    |    <0.001%    |

假设参数：

- L1缓存：命中时间1周期，命中率95%
- L2缓存：命中时间10周期，L1未命中时命中率60%
- DRAM：200周期(~100ns)
- 磁盘：2000万周期(~10ms)

#### AMAT计算

1. 无分页情况
    - AMAT = 1 + 5%×10 + 5%×40%×200 = 5.5周期

2. 有分页情况
    - 内存命中率99%: AMAT ≈ 4005.5 (变慢728倍)
    - 内存命中率99.9%: AMAT ≈ 405.5
    - 内存命中率99.9999%: AMAT ≈ 5.9

- 页面错误对性能影响巨大
- 示例：10秒程序在99%命中率下可能需要2小时
- 需要非常高的内存命中率(>99.9999%)才能保持可接受性能
