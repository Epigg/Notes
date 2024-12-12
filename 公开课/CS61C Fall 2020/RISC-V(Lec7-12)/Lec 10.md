# RISC-V Procedures

*November 23, 2024*

## Function Call Example

[[Lec 9#Six Fundamental Steps in Calling a Function|调用函数的六个基本步骤]]

```C
int Leaf (int g, int h, int i, int j) { 
	int f; 
	f = (g + h) – (i + j); 
	return f; 
}
```

- 参数变量g，h，i和 j 在参数寄存器a0，a1，a2和 a3 中，f在 s0 中
- 假设需要一个临时寄存器s1

哪里保存旧的寄存器值用于函数调用后恢复？

- 在调用函数之前需要一个地方保存旧值，在返回时恢复它们，并删除它们
- 理想选择的是栈(*stack*)：后进先出（LIFO）队列 
	- push
	- pop
- 栈存储在内存中，所以需要寄存器指向它
- `sp` 是 RISC-V 中的栈指针 (x2)
- 惯例是从高地址到低地址增长栈，即压栈减少sp， 弹栈增加sp

### Stack

[[Lec 5#The Stack|Stack in C]]

在思路上是完全类似的！

### RISC-V Code for Leaf()

```
Leaf:
addi sp, sp, -8  # adjust stack for 2 items 
sw s1, 4(sp)     # save s1 for use afterwards 
sw s0, 0(sp)     # save s0 for use afterwards 

add s0, a0, a1   # f = g + h 
add s1, a2, a3   # s1 = i + j 
sub a0, s0, s1   # return value (g + h) – (i + j) 

lw s0, 0(sp)     # restore register s0 for caller 
lw s1, 4(sp)     # restore register s1 for caller 
addi sp, sp, 8   # adjust stack to delete 2 items 

jr ra            # jump back to calling routine
```

prologue -> process... -> epilogue -> return

当过程结束时，栈指针移回上一个帧，这时候调用过程的内存空间中的信息*仍然存在*，但是不再有效（或安全）了。

## Nested Calls and Register Conventions

### Nested Procedures

```C
int sumSquare(int x, int y) { 
	return mult(x,x)+ y; 
}
```

一个过程调用了 `sumSquare` ，现在 `sumSquare` 要调用 `mult` 。在ra中的值是 `sumSquare` 要跳转回去的指令地址，但是它会被 `mult` 的调用覆盖。需要在调用 `mult` 前保存 `sumSquare` 的返回地址，也是通过堆栈实现。

### Register Conventions

- CalleR: 调用者，调用其他函数的函数
- CalleE: 被调用者，被调用的函数

当被调用者从执行中返回时，调用者需要知道哪些寄存器可能已经更改，哪些寄存器保证不变。

寄存器约定(*Register Conventions*)：一组普遍接受的规则，关于在过程调用( `jal` )之后哪些寄存器将保持不变，哪些寄存器可以更改。

为了减少溢出和恢复寄存器的昂贵负载和存储，RISC-V 函数**调用约定**将寄存器分为两类：

- 函数调用后*保留*(*Preserved* across function call):
	- 调用者可以依赖于不变的值
	- sp, gp, tp
	 “saved registers” s0-s11 (s0 is also fp)
- 函数调用后*不保留*(*Not Preserved* across function call):
	- 调用者*不能*依赖于可能改变的值
	- Argument/return registers a0-a7, ra
	 “temporary registers” t0-t6

### RISC-V Symbolic Register Names

| Register | ABI Name | Desciption                       | Saver  |
| -------- | -------- | -------------------------------- | ------ |
| x0       | zero     | Hard-wired zero                  | -      |
| x1       | ra       | Return address                   | Caller |
| x2       | sp       | Stack pointer                    | Callee |
| x3       | gp       | Global pointer                   | -      |
| x4       | tp       | Thread pointer                   | -      |
| x5       | t0       | Temporary/Alternae link register | Caller |
| x6-7     | t1-2     | Temporaries                      | Caller |
| x8       | s0/fp    | Saved register/Frame pointer     | Callee |
| x9       | s1       | Saved register                   | Callee |
| x10-11   | a0-1     | Function arguments/Return values | Caller |
| x12-17   | a2-7     | Function arguments               | Caller |
| x18-27   | s2-11    | Saved registers                  | Callee |
| x28-31   | t3-6     | Tempoparies                      | Caller |

- 保存寄存器("saved register")：约定应当在函数调用返回时得到恢复的寄存器
- 参数寄存器：函数的参数通过a0~a7传递，如果多于8个，则会通过栈传递
- 临时寄存器：可以自由使用。对于Leaf的例子完全可以使用临时寄存器代替保存寄存器

## Memory Allocation

### Allocating Space on Stack

- C 有两个存储类：自动和静态
	- 自动变量是函数的局部变量，在函数退出时被丢弃
	- 静态变量存在于过程的出口和入口全过程
- 对于不能放入寄存器的自动（局部）变量，使用栈存储
- 过程帧(*Procedure frame*)或激活记录(*activation record*)：保存寄存器和局部变量的堆栈段

![[stack when calling.png]]

### Using the Stack

- sp 总是指向当前栈使用的空间的底部
- 为了使用堆栈，我们将这个指针减去我们需要的空间量，然后用内容填充它

回到 `sumSquare` ，编译的结果如下：

```
sumSquare:
	addi sp, sp, -8 # space on stack 
	sw ra, 4(sp)    # save ret addr 
	sw a1, 0(sp)    # save y 
	mv a1, a0       # mult(x, x) 
	jal mult        # call mult 
	lw a1, 0(sp)    # restore y 
	add a0, a0, a1  # mult() + y 
	lw ra, 4(sp)    # get ret addr 
	addi sp, sp, 8  # restore stack jr ra
mult: ...
```

其中，压栈步骤包括 `addi` 和 `sw` ，弹栈步骤包括 `lw` 和 `addi` 

### RV32 Memory Allocation

[[Lec 5#C Memory Managements|C语言中重要的三个内存区域]]

- RV32 约定（与RV64/RV128有不同的内存布局）
- 堆栈从高内存开始，然后逐渐减少
	- 十六进制：$bfff\_fff0_{hex}$
	- 堆栈必须在16字节边界上对齐（前面的例子没有满足这点）
- RV32程序（即*text segment*）在低端
	- $0001\_0000_{hex}$
- 静态数据段（*static data segment*）（常量和其他静态变量）上面的文本为静态变量
	- RISC-V 约定全局指针（gp）指向静态
	- RV32 gp = $1000\_0000_{hex}$
- 堆（Heap）位于静态数据区（Static Data）之上，用于存储那些大小可以动态变化的数据结构；它向高地址方向增长

![[RV32 Memory Allocation.png]]

## RV32 So Far..

1. Arithmetic/logic
	- add rd, rs1, rs2
	- sub rd, rs1, rs2
	- and rd, rs1, rs2 
	- or rd, rs1, rs2 
	- xor rd, rs1, rs2 
	- sll rd, rs1, rs2 (shift left logical)
	- srl rd, rs1, rs2 
	- sra rd, rs1, rs2 (shift right arithmetic)
2. Immediate
	- addi rd, rs1, imm 
	- andi rd, rs1, imm 
	- ori rd, rs1, imm 
	- xori rd, rs1, imm 
	- slli rd, rs1, imm 
	- srli rd, rs1, imm 
	- srai rd, rs1, imm
3. Load/store
	- lw rd, rs1, imm
	- lb rd, rs1, imm
	- lbu rd, rs1, imm
	- sw rs1, rs2, imm
	- sb rs1, rs2, imm
4. Branching/jumps
	- beq rs1, rs2, Label
	- bne rs1, rs2, Label
	- bge rs1, rs2, Label
	- blt rs1, rs2, Label
	- bgeu rs1, rs2, Label
	- bltu rs1, rs2, Label
	- jal rd, Label
	- jalr rd, rs, imm
