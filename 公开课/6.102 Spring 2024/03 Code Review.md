# Reading 3: Code Review

*November 16, 2024*

#### Objectives

- code review: reading and discussing code written by somebody else
- general principles of good coding: things you can look for in every code review, regardless of programming language or program purpose

## Code review

代码审查(==Code review==)是由非代码原作者的人对源代码进行仔细、系统的研究。

两个目的：

- **Imporving the code.**
- **Improving the programmers.**

## Style standards

It’s important to be self-consistent, however, and it’s _very_ important to follow the conventions of the project you’re working on.

## Smelly example #1

程序员经常将糟糕的代码描述为具有需要去除的“难闻气味”。 “代码卫生”是这个问题的另一个词。

```ts
function dayOfYear(month: number, dayOfMonth: number, year: number): number {
    if (month === 2) {
        dayOfMonth += 31;
    } else if (month === 3) {
        dayOfMonth += 59;
    } else if (month === 4) {
        dayOfMonth += 90;
    } else if (month === 5) {
        dayOfMonth += 31 + 28 + 31 + 30;
    } else if (month === 6) {
        dayOfMonth += 31 + 28 + 31 + 30 + 31;
    } else if (month === 7) {
        dayOfMonth += 31 + 28 + 31 + 30 + 31 + 30;
    } else if (month === 8) {
        dayOfMonth += 31 + 28 + 31 + 30 + 31 + 30 + 31;
    } else if (month === 9) {
        dayOfMonth += 31 + 28 + 31 + 30 + 31 + 30 + 31 + 31;
    } else if (month === 10) {
        dayOfMonth += 31 + 28 + 31 + 30 + 31 + 30 + 31 + 31 + 30;
    } else if (month === 11) {
        dayOfMonth += 31 + 28 + 31 + 30 + 31 + 30 + 31 + 31 + 30 + 31;
    } else if (month === 12) {
        dayOfMonth += 31 + 28 + 31 + 30 + 31 + 30 + 31 + 31 + 30 + 31 + 31;
    }
    return dayOfMonth;
}
```

## Don’t repeat yourself (DRY)

重复的代码存在安全风险。如果在两个地方有相同或非常相似的代码，那么根本的风险是两个副本中都存在错误，并且某些维护人员修复了一个地方的错误，但没有修复另一个地方。

## Comment where needed

优秀的软件开发人员会在他们的代码中编写注释，并且明智地这样做。好的注释应该使代码更容易理解，更安全地避免错误（因为重要的假设被记录），并且更易于更改。

一种关键注释是规范，它出现在函数或类之上，并记录函数或类的行为。在 TypeScript 中，这通常被编写为文档注释，以`/**`开头并包括`@`语法，如函数的`@param`和`@returns` 。这是规范的示例：

```ts
/**
 * Compute the hailstone sequence.
 * See https://en.wikipedia.org/wiki/Collatz_conjecture#Statement_of_the_problem
 * @param n starting number of sequence; requires integer n > 0.
 * @returns the hailstone sequence starting at n and ending with 1.
 *         For example, hailstone(3) = [3, 10, 5, 16, 8, 4, 2, 1].
 */
function hailstoneSequence(n: number): Array<number> {
    ...
}
```

另一个重要的注释是指定从其他地方复制或改编的一段代码的出处或来源。两个原因：

- 避免侵犯版权
- 借用的代码可能存在错误或已经过时

有些注释是不好的并且没有必要。例如，将代码直译：

```ts
while (n !== 1) { // test whether n is 1   (don't write comments like this!)
   ++i; // increment i
   l.push(n); // add n to l
}
```

但应该注释晦涩的代码：

```ts
const sum = n*(n+1)/2;  // Gauss's formula for the sum of 1...n

// here we're using the sin x ~= x approximation, which works for very small x
const moonDiameterInMeters = moonDistanceInMeters * apparentAngleInRadians;
```

通常最好将代码分成*段落*，即具有特定目的的行组，并以注释作为主题句开始每个段落。

```ts
// prepare the rocket
...
...
...

// launch the rocket
...
...
...

// enter orbit
...
...
```

一旦你开始以这种方式思考你的代码，你可能会意识到这些段落中的部分或全部都应该是辅助函数，然后注释可能会自然地变成命名良好的函数调用。

```ts
prepareRocket(...);
launchRocket(...);
enterOrbit(...);
```

清晰的名称通常不需要注释。

## Fail fast

快速失败(==*Failing fast*==)意味着代码应该尽早揭示其错误。越早发现问题（越接近其原因），就越容易发现和解决。

`dayOfYear`函数不会很快失败——如果你以错误的顺序向它传递参数，它会悄悄地返回错误的答案。事实上，按照`dayOfYear`的设计方式，非美国人很可能会以错误的顺序传递参数！

## Avoid magic numbers

有一个计算机科学笑话，计算机科学家唯一理解的数字是 0、1，有时甚至是 2。

所有其他常量都称为==幻数==，因为它们就像凭空出现一样，没有任何解释。

解释数字的一种方法是使用注释，但更好的方法是将数字声明为具有良好、清晰名称的命名常量。

避免使用幻数的一些原因：

- **数字的可读性不如名称。**
- **常数在未来可能需要改变。**
- **常数可能依赖于其他常数。**
	不要对手动计算的数字进行硬编码。使用命名常量，该常量可以根据其他命名常量直观地计算关系。

是否有必要为不依赖于其他常数的数学和物理基本常数命名？答案是有必要。

- 数字本身的表达可能很复杂，因此多次重复并不安全。
- 其次，这些数字也编码了未来可能改变的设计决策。

如果代码中存在大量幻数，则可能表明你需要退后一步，将这些数字视为*数据*而不是命名的常量，并将它们放入允许更简单代码的数据结构中。在 `dayOfYear` 中，如果月份的长度（30、31、28 等）存储在列表或映射等数据结构中，那么它们将更具可读性，并且更加DRY，例如 `monthLength[month]` 。

## One purpose for each variable

不要重复使用参数，也不要重用变量。变量并不是编程中的稀缺资源。自由地声明它们，给它们起个好听的名字，当你不再需要时就停止使用它们。如果一个曾经表示一件事的变量突然开始在几行后表示不同的含义，那么会让读者感到困惑。

这不仅是一个易于理解的问题，而且还是一个避免错误(SFB)和准备改变(RFC)的问题。

特别是函数参数，通常应保持不变。对尽可能多的变量使用 `const` ，因为 `const` 关键字表示该变量永远不应该被重新分配，并且 TypeScript 编译器将静态检查它。

## Smelly example #2

`dayOfYear` 中有一个潜在的错误：它根本不处理闰年。作为解决这个问题的一部分，假设我们编写一个闰年函数。

```ts
function leap(y:number):boolean {
    let tmp=y.toString();
	if(tmp[2]==='1'||tmp[2]==='3'||tmp[2]==='5'||tmp[2]==='7'||tmp[2]==='9') {
        if(tmp[3]==='2'||tmp[3]==='6') return true; 
        else
            return false; 
    } else {
        if(tmp[2]==='0' && tmp[3]==='0') {
            return false; 
        }
        if(tmp[3]==='0'||tmp[3]==='4'||tmp[3]==='8') return true; 
    }
    return false;
}
```

## Use good names

好的函数和变量名称应该很长并且具有自我描述性。通常可以通过使代码本身更具可读性，并使用更好的名称来描述函数和变量来完全避免注释。例如：

```ts
const tmp = 86400;  // tmp is the number of seconds in a day (don't do this!)
```

```ts
const secondsPerDay = 86400;
```

一般来说，像 `tmp` 、`temp` 和 `data` 这样的变量名很糟糕——这是程序员极度懒惰的症状。这些名称通常没有意义，最好使用更长、更具描述性的名称，这样你的代码本身就可以清晰地读取。

遵循语言的词汇命名约定。在 Python 和 TypeScript 语言中，类通常以大写字母开头，变量名和函数名以小写字母开头。但这两种语言的不同之处在于如何将多词短语呈现为函数或变量名称。 Python 使用snake_case（下划线分隔短语中的单词），而 TypeScript 使用camelCase（将第一个单词后的每个单词大写）。

类 Java 语言也使用大写将全局常量与变量和局部常量区分。 `ALL_CAPS_WITH_UNDERSCORES` 用于全局常量。但是函数内部声明的局部变量用 `camelCaseNames` 。

在任何语言中，函数名称通常是动词短语，例如 `getDate` 或 `isUpperCase` ，而变量和类名称通常是名词短语。选择简短的单词，并且要简洁，但避免缩写。例如，`message` 比 `msg` 更清晰，而 `word` 比 `wd` 好得多。

完全避免使用单字符变量名称，除非它们很容易按照惯例理解。例如， `x`和`y`对于笛卡尔坐标有意义， `i`和`j`作为`for`循环中的整数变量。花一点精力来选择好名字是值得的，因为你的代码需要被（你和其他人）阅读的次数比编写的次数多，而且现代的代码编辑器可以轻松地自动完成命名。

## Use whitespace and punctuation to help the reader

使用一致的缩进。 `leap`例子在这方面做得很糟糕； `dayOfYear`例子要好得多。`dayOfYear`很好地将所有数字排列成列，使读者可以轻松比较和检查它们。这是对空白的一个很好的利用。

在代码行中添加空格以使其易于阅读。在二元运算符周围使用空格，例如`===`和`||`来衬托它们并使它们更加明显，并使用多行对齐来使相似点和差异向读者突出：

``` 
	if(tmp[2]==='1'||tmp[2]==='3'||tmp[2]==='5'||tmp[2]==='7'||tmp[2]==='9') {
```

```
    if (   tmp[2] === '1'
	    || tmp[2] === '3'
        || tmp[2] === '5'
        || tmp[2] === '7'
        || tmp[2] === '9' ) {
```

切勿使用制表符进行缩进，仅使用空格字符。请注意，我们说的是*字符*，而不是键。并不是说您永远不应该按 Tab 键；而是说你的编辑器永远不应该在源文件中放入制表符来响应 Tab 键。因为不同的工具对待制表符的方式不同。只需使用空格即可。始终将编辑器设置为在按 Tab 键时插入空格字符。

当使用 TypeScript（或 JavaScript、C、C++ 或 Java）等类 C 语言编写时，请在每个语句末尾添加分号。尽管 TypeScript/JavaScript 具有“自动分号插入”功能，可以自动插入缺失的分号，但它并不总是能达到预期。

始终在`if` 、`while`或`for`语句主体周围使用大括号。尽管类 C 语言允许您在主体仅包含一个语句时省略大括号，但这种快捷方式可能会导致错误。保持一致，并在所有代码块周围放置花括号，无论有多少条语句。

## Smelly example #3

```ts
let LONG_WORD_LENGTH = 5;
let longestWord;

function countLongWords(text: string): void {
    let words: Array<string> = text.split(' ');
    if (words.length === 0) {
        console.log("0");
        return;
    }
    let n = 0;
    longestWord = "";
    for (let word of words) {
        if (word.length > LONG_WORD_LENGTH) ++n;
        if (word.length > longestWord.length) longestWord = word;
    }
    console.log(n);
}
```

## Don’t use global variables

避免全局变量。全局变量(==_global variable_==)是：

- _变量_：其值可以更改的名称
- _全局的_：可以从程序中的任何位置访问和更改。

全局变量的危险：[Global Variables Are Bad](http://c2.com/cgi/wiki?GlobalVariablesAreBad)

在 TypeScript 中，使用`let`或`var`在任何函数定义之外声明全局变量。

但是，如果用`const`来声明，并且变量的类型是不可变的，则名称将变为全局常数(==_global constant_==)。全局常量可以在任何地方读取，但永远不会重新分配或改变，因此风险就消失了。全局常量是常见且有用的。

一般来说，将全局变量转换为参数和返回值，或者将它们放入要调用函数的对象内。

### Kinds of variables in snapshot diagrams

当我们绘制快照图时，区分不同类型的变量很重要：

- a ==_local variable_== inside a function
- an ==_instance variable_== inside an instance of an object
    - an instance variable may also be called a ==_property_== (particularly in TypeScript/JavaScript), a ==_member variable_== (C++), an ==_attribute_== (Python), or a ==_field_== (Java).
- a ==_global variable_== outside of any function or object

局部变量在函数调用时产生，在函数返回时消失。如果正在进行对同一函数的多次调用（例如由于递归），则每次调用都将具有其自己的独立局部变量。

当使用`new`创建对象时，实例变量就会存在，然后当对象不再可访问时，实例变量就会消失。每个对象实例都有自己的实例变量。

全局变量在其声明被执行时就存在，并在程序的剩余生命周期中存在。

一段使用所有三种变量的简单代码：

```ts
let taxRate = 0.05;

class Payment {
    public value: number;
}

function main(): void {
    let payment: Payment = new Payment();
    payment.value = 100;
    taxRate = 0.05;
    console.log(payment.value * (1 + taxRate));
}
```

## Functions should return results, not print them

`countLongWords`尚未准备好进行更改。它使用`console.log()`将一些结果发送到控制台。

一般来说，只有程序的最高级别部分才应该与用户或控制台交互。较低级别的部分应将其输入作为参数，并将其输出作为结果返回。这里唯一的例外是调试输出，当然可以将其打印到控制台。但这种输出不应该是您设计的一部分，而只是调试设计方式的一部分。

## Avoid special-case code

程序员常常想编写特殊的代码来处理看似特殊的情况，例如参数为 0、空数组或空字符串。`countLongWords`示例通过专门处理空`words`数组而陷入此陷阱。

起始`if`语句是不必要的。如果它被省略，并且`words`数组为空，那么`for`循环将什么也不做，并且无论如何都会打印`0` 。事实上，单独处理这种特殊情况可能会导致一个错误——与恰好没有长单词的非空数组相比，空数组的处理方式有所不同。

**积极抵制单独处理特殊情况的诱惑。** 努力地思考一般情况代码，要么确认它实际上已经可以处理特殊情况（这通常是正确的！），或者多花点精力让它处理特殊情况。

编写更广泛的通用代码是有回报的。它会导致函数更短，更容易理解并且隐藏错误的地方更少。它可能更安全，不会出现错误，因为它对正在使用的值做出了更少的假设。而且它更适合更改，因为当对函数的行为进行更改时，需要更新的地方更少。如果还没有编写一般情况的代码而尝试先处理简单的情况，那么你的顺序就错误了。请首先处理一般情况。

> Write a clean, simple, general-case algorithm first, and optimize it later, _only_ if it would actually help.

## Summary

Code review is a widely-used technique for improving software quality by human inspection. Code review can detect many kinds of problems in code, but as a starter, this reading talked about these general practices for good code:

- Don’t Repeat Yourself (DRY)
- Comment where needed
- Fail fast
- Avoid magic numbers
- One purpose for each variable
- Use good names
- Use whitespace and punctuation to help the reader
- Don’t use global variables
- Functions should return results, not print them
- Avoid special-case code

The topics of today’s reading connect to our three key properties of good software as follows:

- **Safe from bugs.** In general, code review uses human reviewers to find bugs. DRY code lets you fix a bug in only one place, without fear that it has propagated elsewhere. Documenting your assumptions with clear comments makes it less likely that another programmer will introduce a bug. The Fail Fast principle detects bugs as early as possible. Avoiding global variables makes it easier to localize bugs related to variable values, since non-global variables can be changed in only limited places in the code.
    
- **Easy to understand.** Code review is really the only way to find obscure or confusing code, because other people are reading it and trying to understand it. Using judicious comments, avoiding magic numbers, keeping one purpose for each variable, using good names, and using whitespace well can all improve the understandability of code.
    
- **Ready for change.** Code review helps here when it’s done by experienced software developers who can anticipate what might change and suggest ways to guard against it. DRY code is more ready for change, because a change only needs to be made in one place. Returning results instead of printing them makes it easier to adapt the code to a new purpose.   

Code review is not the only software development practice with strong supporting evidence. Another is: [sleep](https://increment.com/teams/the-epistemology-of-software-quality/).
