# Introduction to the C Programming Language

## Computer Organization

- ENIAC：第一台电子通用计算机
- EDSAC：第一台存储程序（*Stored-Program*）计算机

> *C is not a “very high-level” language, nor a “big” one, and is not specialized to any particular area of application. But its absence of restrictions and its generality make it more convenient and effective for many tasks than supposedly more powerful languages.*

## Compilation

### Overview

- C 语言编译器（**compilers**）将 C 程序直接转换为特定架构（**architecture-specific**）的机器代码（1和0的字符串）
- Java 转换为架构无关的字节码，然后可能由即时编译器（JIT）进行编译
- Python 环境在运行时转换为字节码

这些主要区别在于你的程序**何时**被转换为低级机器指令（"解释的层次"）

对于C语言，通常是两个步骤的过程：
- 将 `.c` 文件*编译（compiling）* 为 `.o` 文件
- 将 `.o` 文件*链接（linking）* 成可执行文件

*汇编（Assembling）* 过程也会进行（但是被隐藏）。

### Advantages

- 合理的编译时间：编译程序的改进（Makefiles）允许只重新编译修改过的文件
- 出色的运行时性能：对于类似的代码，通常比 Scheme 或 Java 快得多（因为它针对特定架构进行优化）

### Disadvantages

- 编译后的文件（包括可执行文件）是特定架构的，依赖于处理器类型和操作系统
- 可执行文件必须在每个新系统上重新构建
- "修改->编译->运行（重复）"的迭代周期在开发过程中可能很慢
    - 但 `make` 只重建改变的部分，并且可以并行编译：`make -j`
    - 不过链接器是顺序的 -> 阿姆达尔定律（Amdahl’s Law）

### C Pre-processor (CPP)

C 源文件在编译器看到代码之前首先通过宏处理器 CPP。

- CPP 用单个空格替换注释
- CPP 命令以 `#` 开始
    - `#include "file.h"`
    - `#include <stdio.h>`
    - `#define PI (3.14159)`
    - `#if/#endif`
- 使用 gcc 的 `-save-temps` 选项可以看到预处理的结果

#### Macros

实际上，所有的`#define`所做的就是*字符串替换*。

- `#define min(X,Y) ((X)<(Y)?(X):(Y))`

如果`foo(z)`有副作用（*side-effect*），这可能会产生有趣的*错误*

- `next = min(w, foo(z));`
- `next = ((w)<(foo(z))?(w):(foo(z)));`

## C vs. Java

| 特性           | C语言                                                                                         | Java                                                                                                                                     |
| ------------ | ------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| 语言类型         | 面向函数                                                                                        | 面向对象（Object）                                                                                                                             |
| 编程单位         | 函数                                                                                          | 类（Class） = 抽象数据类型（ADT）                                                                                                                   |
| 编译           | `gcc hello.c`创建机器语言代码                                                                       | `javac Hello.java` 创建 Java 虚拟机语言字节码                                                                                                      |
| 执行           | `a.out`加载并执行程序                                                                              | `java Hello`解释字节码                                                                                                                        |
| hello, world | # include <stdio.h><br>int main(void) <br>{<br>    printf("Hello\n");<br>    return 0;<br>} | public class HelloWorld {  <br>   public static void main(String[] args) {  <br>       System.out.println("Hello");  <br>   }  <br>}<br> |
| 存储           | 手动（`malloc, free`）                                                                          | `new` 分配并初始化，自动（垃圾回收）释放                                                                                                                  |
| 常量           | `const`和`#define`                                                                           | `final`                                                                                                                                  |
| 预处理器         | 有                                                                                           | 无                                                                                                                                        |
| 变量命名约定       | `sum_of_squares`                                                                            | `sumOfSquares`                                                                                                                           |
| 访问库          | `#include <stdio.h>`                                                                        | `import java.io.File`                                                                                                                    |

- 运算符几乎相同

更新到 **ANSI C**：C99 C11

`main`函数：`int main(int argc, char *argv[])`

## C Syntax

- 真或假（True or False）
- 类型变量（Typed Variables）
- 整数（Integers）
- 常量和枚举（Consts and Enums）
- 类型函数（Typed Function）
- 结构体（Structs）
- 控制流（Control Flow）
    - if - else
    - while
    - for 
    - switch (break)
