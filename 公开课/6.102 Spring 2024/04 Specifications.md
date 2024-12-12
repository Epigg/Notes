# Reading 4: Specifications

*November 17, 2024*

#### Objectives

- Understand preconditions and postconditions in function specifications, and be able to write correct specifications
- Be able to write tests against a specification
- Understand how to use exceptions

## Introduction

规范是团队合作的关键。没有规范，就不可能委托他人负责实现功能的责任。规范就像一份合同：实现者有责任履行合同，而使用该功能的客户可以信赖这份合同。

## Behavioral equivalence

假设你正在编写一个包含该函数的程序，该函数用于查找数组中指定整数的索引：

```ts
function find(arr: Array<number>, val: number): number {
    for (let i = 0; i < arr.length; i++) {
        if (arr[i] === val) return i;
    }
    return -1;
}
```

`find` 函数在程序中有很多*客户端*（函数被调用的地方）。你意识到，当 `find` 被调用时，如果使用的是一个大数组，那么它找到的值很可能要么接近数组的起点，要么接近终点（这样查找速度很慢）。因此，你想到了一个巧妙的办法，即同时从数组的两端进行搜索，从而加快速度：

```ts
function find(arr: Array<number>, val: number): number {
    for (let i = 0, j = arr.length-1; i <= j; i++, j--) {
        if (arr[i] === val) return i;
        if (arr[j] === val) return j;
    }
    return -1;
}
```

为了确定行为等价性(==*behavioral equivalence*==)，就是要问能否在不影响正确性的情况下用一种实现替代另一种实现。

这些实现不仅具有不同的性能特征，而且在某些输入情况下具有不同的输出行为，例如 `val` 在数组中出现*不止一次*。

然而，当 `val` 只出现在数组的一个索引上时，两种实现方式的行为相同：都返回该索引。在这种情况之外，客户可能从来不会依赖这种行为。对于这样的客户端，这两个版本的 `find` 是相同的。

行为等价性的概念是在客户端的眼中。为了能够用一种实现替代另一种实现，并知道何时可以接受这种替代，我们需要一种规范，准确说明客户端依赖于什么。

在这种情况下，允许这两种实现在行为上等同的规范可以是：

```ts
find(arr: Array<number>, val: number): number
```

- *requires*: `val` 在 `arr` 中恰好出现一次
- *effects*: 返回满足 `arr[i]` = `val` 的索引 `i` 

*required* 子句指定了合法调用 `find` 时必须为真的条件，*effects* 子句则指定了客户端合法调用后的实现行为。

## Why specifications?

`find` 示例展示了规范如何帮助程序既能适应变化，又能避免错误。程序中许多最糟糕的错误都是由于对两段代码之间接口行为的误解造成的。代码中的精确规范可以让你分清责任（是代码片段的责任，而不是人的责任），也可以让你不必再苦苦思索错误的位置。

规范对有助于模块更容易理解，就像黑盒子外面的标签一样。有了规范，你无需阅读模块的代码就能了解模块的功能。

 规范对函数的实现者来说是件好事，因为它可以让实现者在不告知客户的情况下自由地改变实现。
 
 规范还可以使代码更快。规范可以排除函数可能被调用的某些状态。限制输入可以让实现者跳过不再需要的昂贵检查，使用更高效的实现。

合同就像客户端与实现者之间的防火墙。这道防火墙的结果是解耦(==*decoupling*==)，允许模块代码和客户端代码独立更改，只要更改符合规范即可——各自都遵守合约规定。

## Specification structure

抽象地说，一个函数的规范(==_specification_==)包括几个部分：

- signature: 函数签名，包括名称、参数类型和返回类型
- _requires_: 要求，说明对参数的其他限制
- _effects_: 效果，描述函数的返回值、异常和其他效果

这些部分共同构成了函数的前置条件(==_precondition_==)和后置条件(==_postcondition_==)。

前提条件是客户端（函数的调用者）的一项义务。它是函数被调用时的状态条件。前提条件的一个方面是函数签名中参数的数量和类型，TypeScript 可以静态检查这些参数。其他条件写在 *requires* 子句中，例如：

- 缩小参数类型（例如 `x is an integer >= 0` ）
- 参数之间的相互作用（例如 `val` 在 `arr` 中恰好出现一次 )

后置条件是函数实现者的义务。TypeScript 可以静态检查后置条件的某些部分，尤其是返回类型。其他条件写入 *effects* 子句，包括：

- 返回值与输入值的关系
- 何时抛出何种异常
- 对象是否和如何改变

一般来说，后置条件是函数*调用后*程序状态的条件，前提是前置条件为真。

整体结构是一个逻辑蕴涵：如果在调用函数时前提条件成立，那么在函数完成时后条件必须成立。如果调用函数时前提条件不成立，函数的实现就*不受*后置条件的约束。

## Specifications in TypeScript

TypeScript 的静态类型声明实际上是函数的前置条件和后置条件的一部分，由编译器自动检查和执行。合同的其余部分，即不能写成类型的部分，必须在函数前面的注释中描述，通常要靠人来检查和保证。

一个最初为 Java 设计、但现在也被 TypeScript 和 JavaScript 使用的常见约定是在函数前添加文档注释，其中包括：

- 参数由 `@param` 描述
- 结果由 `@returns` 描述

在可能的情况下，前置条件一般放在 `@param` 中，后置条件放在 `@returns` 中。

`find` 的规范在 TypeDoc 中可能这样呈现：

```ts
/**
 * Find a value in an array.
 * @param arr array to search, requires that val occurs exactly once
 *            in arr
 * @param val value to search for
 * @returns index i such that arr[i] = val
 */
function find(arr: Array<number>, val: number): number
```

### What a specification may talk about

函数的规范可以谈论函数的参数和返回值，但绝不能谈论函数的局部变量或函数类的私有字段。就客户端而言，函数的实现处于防火墙之后。

客户端不必担心函数实现的细节，只需关注规范即可。

## Avoid null

许多语言允许变量使用 `None` 或 `null` 这样的特殊值，这意味着变量不指向对象。TypeScript 也是如此，因此无论变量的类型如何，任何变量都可以是 `null` ：

```typescript
let word: string = null;  // legal by default!
```

这是静态类型系统的漏洞，因为 `null` 并不是真正合法的 `string` 值。你不能使用这个引用调用任何方法或使用任何属性：

```js
word.toLowerCase() // throws TypeError
word.length        // throws TypeError
```

注意， `null` 与空字符串 `""` 或空数组 `[ ]` 不同。在空字符串或空数组上，可以调用方法和访问字段。

还要注意的是，一个数组可能是非空的，但其值可能包含 `null` ：

```ts
let words: Array<string> = [ null ];
```

null 值既麻烦又不安全，因此好的程序设计都会尽量避免空值。一般来说，**参数和返回值中不允许出现 null 值**，除非规范另有明确规定。因此，每个函数的参数和返回值中的对象和数组类型，包括容器类型的元素都必须是非null值。

如果函数允许在参数或返回值中使用 `null` ，则需要在类型中明确声明 `null` （例如 `string|null` ）。但应尽量避免这样做。**Avoid `null`**.

## Include emptiness

确保理解 `null` 和空(emptiness)的区别。

在 Python 中， `None` 与空字符串 `""` 、空数组 `[ ]` 或空字典 `{ }` 不同。这些空对象是有效的对象，只是碰巧不包含任何元素，但可以使用该类型允许的所有常规操作。

在 TypeScript 中。 `null` 引用不是有效的字符串、数组、映射或其他对象。但空字符串 `""` 是一个有效的 `string` 值，空数组 `[ ]` 是一个有效的数组值。

除非规范明确禁止，否则**始终允许将 empty 值作为参数或返回值**。

## Testing and specifications

在测试中，*黑箱*测试只考虑规范，而*白箱*测试则考虑实际实现。但需要注意的是，即使是**白箱测试也必须遵循规范**([[02 Testing#Black box and glass box testing]])。你的实现可能会提供比规范的要求更强的保证，也可能会有规范未定义的特定行为。但测试用例不应依赖于这种行为。测试用例必须正确，就像其他客户端一样遵守契约。

例如，假设您要测试的 `find` 规范与我们迄今为止使用的规范略有不同：

```ts
find(arr: Array<number>, val: number): number
```

- *requires*: `val` 出现在 `arr` 中
- *effects*: 返回满足 `arr[i]` = `val` 的索引 `i` 

这个规范有一个很强的前提条件，即要求找到 `val` ；它有一个相当弱的后置条件，即如果 `val` 在数组中出现了不止一次，这个规范就不会要求返回 `val` 的哪个索引。即使你在实现 `find` 时总是返回最低索引，你的测试用例也不能假设这种特定行为：

```ts
let array: Array = [7, 7, 7];
let i = find(array, 7);
assert.strictEqual(i, 0);  // bad test case: assumes too much, more than the postcondition promises
assert.strictEqual(array[i], 7);  // correct
```

同样，即使你在实现 `find` 时，（合理地）使其在 `val` 未找到时抛出异常，而不是返回某个任意的误导性索引，你的测试用例也不能假设这种行为，因为它不能以违反先决条件的方式调用 `find()` 。

###  Testing units

利用这些函数回顾一下[[02 Testing#Unit and integration testing|Testing中的搜索引擎示例]]：

```ts
/** 
 * @returns the contents of the file
 */
function load(filename: string): string { ... }

/** 
 * @returns the words in string s, in the order they appear,
 *         where a word is a contiguous sequence of
 *         non-whitespace and non-punctuation characters
 */
function extract(s: string): Array<string> { ... }

/**
 * @returns an index mapping a word to the set of filenames
 *         containing that word, for all files in the input set
 */
function index(filenames: Set<string>): Map<string, Set<string>>  {
    ...
    for (let file of files) {
        let doc = load(file);
        let words = extract(doc);
        ...
    }
    ...
} 
```

单元测试(==*Unit testing*==)是对程序的每个模块单独编写测试。好的单元测试只关注一个规范。如果*另一个*函数未能满足的其规范，我们编写的一个函数的单元测试不应该失败。正如我们在示例中看到的，如果 `load()` 不满足后置条件，对 `extract()` 的测试也不应该失败。

好的集成测试(==*integration tests*==)，即使用模块组合的测试，将确保我们的不同函数具有兼容的规范：不同函数的调用者和实现者按照对方的期望传递和返回值。

集成测试不能取代系统设计的单元测试。如果我们只通过调用 `index` 来测试 `extract` ，那么我们只能在其输入空间中的一小部分进行测试：即只将 `load` 可能输出作为输入。 这样做，我们就为错误留下了藏身之处，当我们在程序的其他地方将 `extract` 用于不同目的时，或者当 `load` 开始返回以新格式编写的文档时，错误就会出现。

### All tests must follow the spec

我们无法测试违反先决条件时的行为。如果前置条件是实现者可以合理检查的，那么修改规范往往是有意义的：删除前置条件，而在后置条件中指定行为。

## Specifications for mutating functions

[[01 Static Checking#Mutating values vs. reassigning variables|可变对象与不可变对象]]

下面介绍规范如何在后置条件中描述副作用(==*side-effects*==)，即可变对象或输入/输出状态的变化。

这里有一个规范描述了一个改变对象的函数：

```ts
addAll(array1: Array<string>, array2: Array<string>): boolean
```

- *requires*: `array1` 和 `array2` 不是同一个对象
- *effects*: 通过将 `array2` 的元素添加到 `array1` 的末尾来修改 `array1` ，并且只有当 `array1` 在调用后发生变化时才返回 true

如果 `array1` 和 `array2` 是同一个数组，这个简单的算法将不会终止，当数组对象增长到如此之大，以至于消耗了所有可用内存时，它将抛出一个内存错误。无论是无限循环还是崩溃，都是规范允许的结果。

另一个变异函数的示例：

```ts
sort(array: Array<string>): void
```

- *requires*: nothing
- *effects*: 将 `array` 排序，即在所有 0 ≤ `i` < `j` < `array.length` 的情况下 `array[i]` ≤ `array[j]`

不改变参数的函数示例：

```ts
toLowerCase(array: Array<string>): Array<string>
```

- *requires*: nothing
- *effects*: 返回一个新数组 `t` ，长度与 `array` 相同，其中 `t[i]` = `array[i].toLowerCase()`

正如 `null` 除非另有说明，否则隐含地不允许出现一样，程序员也假定**除非另有说明，否则不允许变异**。

## Exceptions

由于异常(==*exception*==)是函数的一种可能输出，因此必须在函数的后置条件中加以说明。可以在文档注释中使用 `@throws` 子句来记录异常。

### Exceptions for signaling bugs

Python 编程中的一些异常：

- `IndexError`: 当数组索引 `array[i]` 超出数组 `array` 的有效范围时
- `KeyError`: 当字典查询 `dict[key]` 没有找到匹配的键时
- `TypeError`: 各种动态类型错误，例如试图调用 `None` 上的方法时

这类异常通常表示客户端或实现中存在错误(**bugs**)，异常抛出时显示的信息可以帮助你找到并修复错误。

提示错误的异常不属于函数的后置条件，因此不应出现在 `@throws` 中。

### Exceptions for anticipated failures

异常不只是用来提示错误的，异常可以用来提示预期的故障源。使用 `@throws` 记录提示预期故障的异常，说明故障发生的条件。例如：

```ts
/**
 * Compute the integer square root.
 * @param x integer value to take square root of
 * @returns square root of x
 * @throws NotPerfectSquareError if x is not a perfect square
 */
function integerSquareRoot(x: number): number
```

这是 `addAll` 规范的新版本：

```ts
/**
 * If the array1 !== array2, adds the elements of array2 to the end of array1.
 * @returns true if array1 changed as a result
 * @throws AliasingError if array1 === array2
 */
function addAll(array1: Array<string>, array2: Array<string>): boolean
```

在[[04 Specifications#Specifications for mutating functions|上述最初的]] `addAll` 中，规范将 `array1 !== array2` 作为一个前提条件：如果违反该要求，函数的行为将是未定义的。在这一版本的规范中，前提条件被删除，取而代之的是一个后置条件，它定义了数组是不同对象和数组是同一对象（ `@throws` 子句）时的行为。

## Special results

异常是传达调用者不可能处理的问题的最佳方式。

对于调用者*应处理*的不寻常或异常结果，不同的编程语言采取了不同的方法。例如，在 Java 语言中，实现者可以声明静态要求处理的异常：如果调用者没有得到预期，也没有将其作为抛出异常添加到自己的规范中，编译器就会给出一个静态错误。

TypeScript 不提供异常处理的静态检查，因此使用异常处理特殊情况的结果可能会导致错误。一个很好的替代方法是将函数的返回类型设为联合类型，这种类型的值集是由两个（或更多）其他类型定义的值集的联合：

```ts
/**
 * Compute the integer square root.
 * @param x integer value to take square root of
 * @returns square root of x if x is a perfect square, undefined otherwise
 */
function integerSquareRoot(x: number): number|undefined
```

`undefined` 与 `null` 非常相似：在 JavaScript 中，已声明但未初始化的变量的值是 `undefined` 而不是 `null` 。

`null` 表示 "没有值的值"，在任何时候都应避免使用。与此相反， `undefined` 表示 "这里根本没有值"，只要严格执行 null 检查，我们就可以在联合类型中使用它来表示特殊情况的结果。

使用更新后的 `integerSquareRoot` 而不考虑它可能返回 `undefined` 的可能性，在很多情况下都会导致静态错误：

```ts
let twice = integerSquareRoot(input) * 2; // static error
if (integerSquareRoot(input) > 4) { ... } // static error
console.log('Your lucky number is:', integerSquareRoot(5)); // no error
```

### Missing `Map` keys

在 Python 中，后置条件指定了一个 `KeyError` 。 在 TypeScript 中，对于值为 `V` 类型的 `Map` ，后置条件指定了一个返回类型 `V|undefined` 。

```ts
const zoo = new Map([['Tim', 'beaver']]);
zoo.get('Tim');     // => 'beaver'
zoo.get('Bao Bao'); // => undefined
```

### Illegal array indices

```ts
const zoo = [ 'Tim' ];
for (let i = 0; i <= 1; i++) {
  let name: string = zoo[i];
  console.log(`Hello, ${name}!`);
}
// => prints:
//    Hello, Tim!
//    Hello, undefined!
```

请注意，这个示例正在进行越界索引（超出数组的末尾）。数组索引返回 `undefined` 的另一个可能原因是数组是稀疏的(*sparse*)，应避免使用稀疏数组。

*sparse*: 可以在任意索引处放置新元素，而不仅仅是在从 0 开始的连续序列中。例如：

```typescript
const arr: Array<string> = [];
arr[2] = "x";
arr[0]    // returns undefined
arr[1]    // returns undefined
arr[2]    // returns "x"
arr[3]    // returns undefined
```

## Summary

A specification acts as a crucial firewall between the implementer of a module and its client. It makes separate development possible: the client is free to write code that uses the module without seeing its source code, and the implementer is free to write the code that implements the module without knowing how it will be used.

Let’s review how specifications help with the main goals of this course:

- **Safe from bugs**. A good specification clearly documents the mutual assumptions that a client and implementer rely on. Bugs often come from disagreements at the interfaces, and the presence of a specification helps avoid those disagreements. Using machine-checked language features in your spec, like static typing and exceptions rather than just a human-readable comment, can reduce bugs still more.
    
- **Easy to understand**. A short, simple spec is easier to understand than the implementation itself, and saves other people from having to read the code.
    
- **Ready for change**. Specs establish contracts between different parts of your code, allowing those parts to change independently as long as they continue to satisfy the requirements of the contract.