# Reading 1: Static Checking

#### Objectives

- static typing
- the big three properties of good software

## Static typing

 - TypeScript 是静态类型(**statically-typed**)语言。变量可以在编译(**compile time**)时(在程序运行之前)分配类型，因此编译器可以使用这些变量推断表达式的类型。
- 相比之下，JavaScript 和 Python 是动态类型化(**dynamically-typed**)的语言，因为这种检查会推迟到运行(**runtime**)时(程序运行时)。
- 静态类型是一种特殊的静态检查(**static checking**)，这意味着在编译时检查 bug。

### Static types at compile time, dynamic types at runtime

> 静态类型声明在编译时用于执行静态检查。添加静态类型不会改变程序在运行时的行为是由实际值驱动的事实。

### Support for static typing in dynamically-typed languages

Python 3.5及更高版本允许您在代码中声明类型提示，例如:

```python
# Python function declared with type hints
def hello(name: str) -> str:
    return 'Hi, ' + name
```

声明的类型可以被像 Mypy 这样的检查器用来静态地查找类型错误，而不必运行代码。

向 Python 和 JavaScript 这样的动态类型语言添加静态类型，反映了软件工程师普遍的信念，即使用静态类型对于构建和维护大型软件系统也是必不可少的。

## Static checking, dynamic checking, no checking

- **静态检查**: 在程序运行之前就会自动发现 bug。
- **动态检查**: 执行代码时会自动发现 bug。
- 没有检查: 语言根本不能帮助您找到错误。
 
静态检查可以捕捉：

1. 语法错误
2. 拼写错误 - `Math.sine(2)`
3. 错误的参数数量 - `Math.sin(30, 20)`
4. 错误的参数类型 - `Math.sin("30")`
5. 错误的返回类型

动态检查可以捕捉：

（均为Python中的例子）

1. 特定的非法参数值 - `ZeroDivisionError`
2. 非法的类型转换 - `int("hello")` 抛出 `ValueError`
3. 超出范围的索引或不存在的键 - `"hello"[13]` 抛出 `IndexError` ; `{'a': 1}['b']` 抛出 `KeyError`

TypeScript 执行静态检查，但是它的运行时行为完全由 JavaScript 提供，而且 JavaScript 的设计者决定不检查（**no checking**）上面的许多情况。

- `ZeroDivisionError` => `Infinity` (or `-Infinity`)
- `IndexError` or `KeyError` => `undefined`

没有检查会使 bug 更难被发现，因为这些特殊的值可以通过进一步的计算传播，直到最终发生故障，远离代码中的原始错误。

### Surprise: `number` is not a true number

- **Limited precision for integers**.

	TypeScript 中的所有数字都是浮点数，这意味着大数量级的整数只能近似地表示。可以精确地表示 $-2^{53}$ 和 $2^{53}$ (不包含)之间的整数，但是超过这个范围，浮点表示只保留数字的最高有效二进制数字。
	
	对可表示整数的界限可作为内置常量使用:
	- `Number.MAX_SAFE_INTEGER` (约 $2^{53}$ or $10^{36}$) 是最大的精确可表示整数
	- `Number.MIN_SAFE_INTEGER` (约 $-2^{53}$ or $-10^{36}$ ) 是最小的精确可表示整数
	
	在这些安全范围内的整数的算术保证保持精确。

- **Special values**.

	`nuumber` 类型有几个非真实数字的特殊值:
	- `Number.NaN` (表示“不是一个数字”)
	- `Number.POSITIVE_INFINITY` (以 `"Infinity"` 打印)
	- `Number.NEGATIVE_INFINITY` (以 `"-Infinity"` 打印)

- **Overflow and underflow**.

	TypeScript 也无法表示过大(远离零)或过小(接近零)的数字:
	- `Number.MAX_VALUE` (约 $10^{308}$ ) 是可以安全地表示的最大数
	- `Number.MIN_VALUE` (约 $10^{-328}$ ) 是可以安全地表示的最大数
	
	如果计算的结果太大或太小，无法满足有限的范围，结果将溢出(成为`Number.POSITIVE_INFINITY` 或 `Number.NEGATIVE_INFINITY`)或下溢(成为零)。

`Number` 是 `number` 的封装器。声明类型时，始终使用小写的 `number` 。

## Arrays

声明一个数字数组并创建空数组值:

```ts
let array: Array<number> = [];
```

用数组编写的冰雹代码:

```ts
let array: Array<number> = [];
let n: number = 3;
while (n !== 1) {
    array.push(n);
    if (n % 2 === 0) {
        n = n / 2;
    } else {
        n = 3 * n + 1;
    }
}
array.push(n);
```

## Functions

如果我们想把这段代码包装成一个可重复使用的模块，我们可以编写一个函数:

```ts
/**
 * Compute a hailstone sequence.
 * @param n  starting number for sequence.  Assumes n > 0.
 * @returns hailstone sequence starting with n and ending with 1.
 */
function hailstoneSequence(n: number): Array<number> {
    let array: Array<number> = [];
    while (n !== 1) {
        array.push(n);
        if (n % 2 === 0) {
            n = n / 2;
        } else {
            n = 3 * n + 1;
        }
    }
    array.push(n);
    return array;
}
```

注释是函数的规范，描述操作的输入和输出。规范应该简洁、清晰和精确。

## Mutating values vs. reassigning variables

There are two aspects to immutability: whether a value is mutable or immutable, and whether a reference (such as a variable) can be reassigned or not.

变异（**Mutation**）指改变一个值的内容。不可变（immutable）类型是其值在创建后永远不能更改的类型，如字符串类型。相比之下，数组类型在 TypeScript 中是可变的，就像列表在 Python 中是可变的一样。

Python 还有不可变的序列，*tuple*，一旦创建就不能更改。TypeScript 没有不可变的元组，但是它有 `ReadonlyArray` ，这是一种省略了改变操作的数组类型。

重新赋值（**Reassignment**）指改变变量指向的位置。Python 允许重新赋值任何变量，但 TypeScript 允许我们声明常量，变量名一次性赋值且永远不能重新分配引用。

```ts
const n: number = 5;
```

## Documenting assumptions

将变量的类型记录下来是对它的一个假设: 例如，变量 `n` 总是引用一个数字。

编程时必须牢记两个目标:

- 与计算机交流。首先说服编译器，你的程序是合理的-语法正确和类型正确。然后得到正确的逻辑，以便在运行时给出正确的结果。
- 和其他人交流。使程序易于理解，以便当有人(可能是您)需要修复、改进或在将来调整它时，他们可以这样做。

## Hacking vs. engineering

Hacking is often marked by unbridled optimism:

- Bad: 在测试任何代码之前编写大量代码。
- Bad: 把所有的细节都记在脑子里，假设自己(和每个使用你的代码的人)会永远记住它们，而不是把它们写在代码里。
- Bad: 假设 bug 不存在，或者很容易找到并修复。

But software engineering is not hacking. Engineers are pessimists:

- Good: 一次写一点，边写边测试 - 测试优先(test-first)编程。
- Good: 记录代码所依赖的假设。
- Good: 保护代码不受愚蠢的影响——尤其是自己! 静态检查对此有所帮助。

## The goals of 6.102

Our primary goal in this course is learning how to produce software that is:

- **Safe from bugs**. Correctness (correct behavior right now) and defensiveness (correct behavior in the future) are required in any software we build.
- **Easy to understand**. The code has to communicate to future programmers who need to understand it and make changes in it (fixing bugs or adding new features). That future programmer might be you, months or years from now. You’ll be surprised how much you forget if you don’t write it down, and how much it helps your own future self to have a good design.
- **Ready for change**. Software always changes. Some designs make it easy to make changes; others require throwing away and rewriting a lot of code.

There are other important properties of software (like performance, usability, security), and they may trade off against these three. But these are the Big Three that we care about in 6.102, and that software developers generally put foremost in the practice of building software. It’s worth considering every language feature, every programming practice, every design pattern that we study in this course, and understanding how they relate to the Big Three.
