# Combinational Logic

*November 28, 2024*

## Truth Table

pass...

## Logic Gates

- 与、或、非
- 异或、与非、或非

将2输入门拓展到n输入门，只有异或是不直观的：当输入有奇数个1时，输出为1

## Boolean Algebra

- 基本函数：与（AND）、或（OR）和非（NOT）
- 布尔代数的强大之处：
    - 使用与门（AND）、或门（OR）和非门（NOT）构成的电路与布尔代数中的方程之间存在一一对应关系
- 在布尔代数中，+ 表示“或”（OR），• 表示“与”（AND），$\bar{x}$ 表示“非”（NOT）

## Laws of Boolean Algebra

| AND                                   | OR                                     |
| ------------------------------------- | -------------------------------------- |
| $x\cdot\bar{x}=0$                     | $x+\bar{x}=1$                          |
| $x\cdot 0=0$                          | $x+1=1$                                |
| $x\cdot 1=x$                          | $x+0=x$                                |
| $x\cdot x=x$                          | $x+x=1$                                |
| $x\cdot y = y\cdot x$                 | $x+y=y+x$                              |
| $(xy)z=x(yz)$                         | $(x+y)+z=x+(y+z)$                      |
| $x(y+z)=xy+xz$                        | $x+yz=(x+y)(x+z)$                      |
| $xy+x=x$                              | $(x+y)x=x$                             |
| $\overline{x\cdot y}=\bar{x}+\bar{y}$ | $\overline{x + y}=\bar{x}\cdot\bar{y}$ |

- 最后一条为 DeMorgan's Law (德摩根定律)

## Canonical forms

利用最大项可将真值表转化为布尔表达式，再通过布尔代数运算化简，最简结果不唯一。

pass...

## In conclusion

![[transform table.png]]
