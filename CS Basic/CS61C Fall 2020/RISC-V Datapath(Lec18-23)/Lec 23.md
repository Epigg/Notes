# Pipelining III

*December 6, 2024*

## Control Hazards

处理器无法确定分支指令的下一条指令，直到分支判断执行完毕。流水线越长，处理器等待的时间便越长，因为它必须等待分支指令处理完毕，才能确定下一条进入流水线的指令。

**分支预测**：猜测程序分支的进行方向，即分支指令的下一个 PC ，直接继续运行预测的后续指令，加快运算速度。

- 如果预测正确，则后续运行的指令是正确的
- 如果预测错误，则需要通过转换为 nop 从流水线中清除不正确的指令

## Superscalar Processors

提高处理器的性能：

1. 时钟频率
	- 受技术和散热限制
2. 流水线
	- 指令的交叠执行
	- 更深的流水线：5 ⇒ 10 ⇒ 15 阶段
		- 每阶段工作量更少，意味着时钟周期更短
		- 但是有更多潜在的冒险（CPI > 1）
3. 多发射 "超标量" （Multi-issue "superscalar"）处理器

多发射"超标量"处理器：

- 复制流水线阶段 ⇒ 形成多条流水线
- 每个时钟周期可以启动多条指令
- CPI(每指令周期数)小于1，因此使用IPC(每周期指令数)来衡量
- 例如，4GHz的4路多发射处理器
    - 峰值性能为16 BIPS(每秒十亿条指令)
    - 峰值CPI = 0.25
    - 峰值IPC = 4
- 在实际运行中，由于依赖关系会降低这个性能（冒险）

"乱序执行"（out-of-order）：

- 在硬件层面动态重排指令顺序
- 目的是减少流水线冒险带来的影响

处理器的性能，每指令周期数可以通过[[Lec 21#Processor Performance Iron Law]]求得。