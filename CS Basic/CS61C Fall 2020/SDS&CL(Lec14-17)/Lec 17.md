# Combinational Logic Blocks

*November 28, 2024*

## Data Multiplexors

多路复用器，或 "mux" 

1位复用器：

| s(select) | c   |
| --------- | --- |
| 0         | a   |
| 1         | b   |

![[1-bit-wide mux.png|300]]

4多路复用器：

![[4-to-1 Multiplexor.png|300]]

## Arithmetic Logic Unit(ALU)

- 大部分处理器包含一个特殊的逻辑模块，称为算数逻辑单元
- 能实现加法、减法、按位与和按位或的简单ALU
	- S为0~3依次对应这四个运算

![[simple ALU.png|400]]

## Adder/Subtractor

1 位全加器

-  $s_i=xor(a_i, b_i, c_i)$
-  $c_{i+1}=maj(a_i,b_i, c_i)=a_ib_i+a_ic_i+b_ic_i$

![[full-adder.png]]

一位全加器依次排列得到 n 位全加器  

![[n-bit adder.png]]

- 对于补码加法 $overflow=c_n XOR c_{n-1}$ 
- 均为0时，显然不溢出，为两正数相加后结果为正，或一正一负相加结果为负
- 均为1时，为两负数相加后结果为负，或一正一负相加结果为正，不溢出
- $c_n=0, c_{n-1}=1$ 时，为两正数相加后超出上限
- $c_n=1, c_{n-1}=0$ 时，为两负数相加后超出下限

## Subtractor Design

$A-B=A+(-B)$ 
一个有符号二进制数的相反数等于它的补码加一

![[subtractor.png]]

## In conclusion

- 使用多路复用器选择输入 ú 
	- S 个选择输入位可以选择 $2^S$ 个输入 
	- 每个输入可以是 n 个位宽，与 S 无关 § 
- 可以分层地实现多路复用器 
- 可以使用多路复用器实现 ALU 
- 使用 N 个 1 位全加器和输入端的 XOR 门实现 N 位加法器-减法器 
	- XOR 用作条件反相器