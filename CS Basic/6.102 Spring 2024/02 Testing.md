# Reading 2: Testing

#### Software in 6.102

| Safe from bugs                                   | Easy to understand                                                   | Ready for change                                  |
| ------------------------------------------------ | -------------------------------------------------------------------- | ------------------------------------------------- |
| Correct today and correct in the unknown future. | Communicating clearly with future programmers, including future you. | Designed to accommodate change without rewriting. |

#### Objectives

- understand the value of testing, and know the process of test-first programming;
- be able to judge a test suite for correctness, thoroughness, and size;
- be able to design a test suite for a function by partitioning its input space and choosing good test cases;
- be able to judge a test suite by measuring its code coverage; and
- understand and know when to use black box vs. glass box testing, unit tests vs. integration tests, and automated regression testing.

## Validation

测试是更一般的过程——确认(==*validation*==)的一个例子。验证的目的是发现程序中的问题，包括:

- 程序的形式推理(**formal reasoning**)，通常称为验证(==*verification*==)。验证形成了程序是正确的形式证明。
- 代码审查(**Code review**)。让其他人仔细阅读您的代码，并非正式地对其进行推理，可能是发现 bug 的一个好方法。
- 测试(**Testing**)。在仔细选择的输入上运行程序并检查结果。

## Why software testing is hard

这里有一些方法在软件世界中并不能很好地工作。

- 穷举的测试(**Exhaustive testing**)是不可行的。可能的测试用例的空间通常太大，无法全面覆盖。
- 随意的测试(**Haphazard testing**)(“just try it and see if it works”)发现错误的可能性很小，除非程序错误太多，以至于任意选择的输入很可能失败而不是成功。
- 随机或统计测试(**Random or statistical testing**)不能像在其他类型的工程构件中发现缺陷那样在软件中发现缺陷。

相反，应该仔细和系统地选择测试用例，即系统性测试(**systematic testing**)。

## Test-first programming

定义一些术语:

- 模块(==*module*==)是软件系统的一部分，它可以独立于系统的其余部分进行设计、实现、测试和推理。
- 规范(==*specifacation*== or spec)描述模块的行为。对于函数，规范给出了参数的类型以及对它们的任何附加约束(例如 `sqrt` 的参数必须是非负的)。它还提供了返回值的类型以及返回值与输入的关系。在 TypeScript 代码中，规范由函数签名和上面描述其功能的注释组成。
- 模块具有提供其行为的实现(==*implementation*==)和使用该模块的客户端(==*clients*==)。对于函数，实现是函数的主体，客户端是调用函数的其他代码。模块的规范对客户端和实现都有约束。
- 测试用例(==*test case*==)是输入的特定选择，以及规范所要求的预期输出行为。
- 测试套件(==*test suite*==)是模块的一组测试用例。

在测试优先编程(==*test-first programming*==)中，你甚至在编写任何代码之前就编写了规范和测试。单个功能的开发按以下顺序进行：

- **Spec**: 编写函数的规范。
- **Test**: 编写执行规范的测试。
- **Implement**: 编写实现。

事实证明，这是从头开始设计程序时遵循的一个很好的模式。测试优先编程的最大好处是避免错误。不要等到开发结束时才进行测试，到时你将有大量未经验证的代码。

## Systematic testing

系统测试(==*Systematic testing*==)意味着我们以有原则的方式选择测试用例，目标是设计一个具有三个理想属性的测试套件：

- **Correct**. 正确的测试套件是规范的合法客户端，它接受规范的所有合法实现。这使我们可以自由地更改模块的内部实现方式，而不必更改测试套件。
- **Thorough**. 彻底的测试套件可以发现实现中的漏洞。
- **Small**. 只有很少的测试用例的小型测试套件，首先编写起来会更快，并且如果规范发生变化，也更容易更新。小型测试套件的运行速度也更快。

设计一个既彻底又小规模的测试套件需要有正确的态度。通常，当你编码时，你的目标是使程序正常运行。但作为测试套件设计者，你希望*让它失败*。这是一个微妙但重要的区别。一个好的测试人员会有意地检查程序可能存在漏洞的所有地方，以便消除这些漏洞。

采取测试态度是测试优先编程的另一个观点。人们很容易将已经编写的代码视为珍贵的东西，并轻率地测试它只是为了看看它是否有效。然而，为了进行*彻底的*测试，你必须残酷无情。测试优先编程允许你在编写任何代码之前戴上“测试帽子”，并采取残酷的观点。

## Choosing test cases by partitioning

我们希望选择一组足够小的测试用例，以便易于编写和维护并快速运行，但又足够彻底，可以发现程序中的错误。

为此，我们将程序的输入空间划分为子域(==*subdomain*==)，子域形成输入空间的一个划分(==*partition*==)。然后，我们从每个子域中选择一个测试用例，这就是我们的测试套件。

子域背后的想法是将输入空间划分为相似输入的集合，程序在这些输入上具有相似的行为，这样我们只需要测试每个集合的一个代表。

请注意，出于划分的目的，程序的输入空间仅包括*合法*输入，程序没有定义行为的输入不是合法输入空间的一部分，不应包含在划分中。

#### Example: `abs()`

JavaScript 库中的 `Math.abs()` 函数：

```ts
/**
 * ...
 * @param a  the argument whose absolute value is to be determined
 * @returns the absolute value of the argument.
 */
function abs(a: number): number
```

**abs : number → number**

该函数具有一维输入空间，由*a*的所有可能值组成。考虑绝对值函数的行为方式，我们可以首先将输入空间划分为这两个子域： { *a* | *a* ≥ 0 } 和 { *a* | *a* < 0 }。在第一个子域上， `abs` 应返回未更改的 *a* 。在第二个子域上， `abs` 返回 *a* 的相反数。

```ts
// partition: a >= 0; a < 0
```

为了为我们的测试套件选择测试用例，我们从分区的每个子域中选择一个任意值 *a* ，例如：

- *a* = 17 覆盖子域 *a* ≥ 0
- *a* = -3 覆盖子域 *a* < 0

#### Example: `max()`

JavaScript 库中的 `Math.max()` 函数，只关注两个参数的形式：

```ts
/**
 * ...
 * @param a  an argument
 * @param b  another argument
 * @returns the larger of a and b.
 */
function max(a: number, b: number): number
```

**max : number × number → number**

划分如下所示：

```ts
// partition: a < b; a > b; a = b
```

测试套件可能是：

- (_a_,_b_) = (1, 2) 覆盖 *a* < *b*
- (_a_,_b_) = (10, -8) 覆盖 *a* > *b*
- (_a_,_b_) = (9, 9) 覆盖 *a* = *b*

总而言之，要形成划分，子域应具有三个理想的属性：

- **disjoint** 不相交的
- **complete** 完整的
- **nonempty** 非空的

### Include boundaries in the partition

错误经常发生在边界(==*boundaries*==)子域之间。

我们将边界合并为分区中的*单元素子域*，以便测试套件必须包含边界值作为测试用例。对于`abs` ，我们将为相关边界添加一个子域，然后，我们原来的两个子域将缩小以排除边界值。

```ts
// partition: a < 0; a = 0; a > 0
```

测试套件可能是：

- *a* = 0
- *a* = 17 覆盖子域 *a* > 0
- *a* = -3 覆盖子域 *a* < 0

#### Example: `BigInt` multiplication

`BigInt` 是 TypeScript 中内置的类型，可以表示任意大小的整数。
 
我们可以将`*`视为一个（重载）函数，它接受两个输入（每个输入均为`BigInt`类型），并生成一个`BigInt`类型的输出：

**multiply : BigInt × BigInt → BigInt**

我们可以独立地选择 *a* 和 *b* ：

- 0
- 1
- 小正整数（≤ `Number.MAX_SAFE_INTEGER`且 > 1）
- 小负整数（≥ `Number.MIN_SAFE_INTEGER`且 < 0）
- 大正整数（> `Number.MAX_SAFE_INTEGER` ）
- 大负整数（< `Number.MIN_SAFE_INTEGER` ）

因此，这将产生 6 × 6 = 36 个子域，用于划分整数对的空间。为了从此分区生成测试套件，我们将从网格的每个正方形中选择任意对 ( _a_ , _b_ )。

### Using multiple partitions

对于具有多个参数的函数，每个参数可能有有​​趣的行为变化和几个边界值，因此根据每个参数的行为的==*笛卡尔乘积*==形成输入空间的单个分区会导致结果测试套件大小的组合爆炸。

另一种方法是将每个输入_a_和_b_的特征视为输入空间的两个单独的分区。一个分区仅考虑*a*的值，另一个分区只考虑*b*的值。

```ts
// partition on a:
//   a = 0
//   a = 1
//   a is small positive integer > 1
//   a is small negative integer
//   a is large positive integer
//   a is large negative integer
//      (where "small" fits in a TypeScript number, and "large" doesn't)
// partition on b:
//   b = 0
//   b = 1
//   b is small positive integer > 1
//   b is small negative integer
//   b is large positive integer
//   b is large negative integer
```

独立分区*a*和*b*会增加不再测试它们之间交互的风险。例如，乘法中的符号处理可能是错误来源，并且结果的符号取决于*a*和*b*的符号。但我们可以添加一个额外的分区来捕获此交互：

```ts
// partition on signs of a and b:
//   a and b are both positive
//   a and b are both negative
//   a positive and b negative
//   a negative and b positive
//   one or both are 0
```

我们可以继续以这种方式添加分区，因为我们更多地考虑规范并观察可能导致错误的其他行为变化。通过仔细选择测试用例，额外的分区应该需要很少（如果有）额外的测试用例。

有时我们可能想在多个分区上使用笛卡尔积方法，以生成更全面的测试套件。但即使在这些情况下，笛卡尔积也可能比我们预期的要小。当来自不同分区的子域结果是互斥的时，笛卡尔积将不会包含该特定子域组合的子域。

## Automated unit testing

对单个模块进行隔离的测试称为单元测试(**unit testing**)。

自动化测试(==**Automated testing**==)意味着自动运行测试并检查其结果。在模块上运行测试的代码是测试驱动程序(*test driver*)，测试驱动程序应该在固定的测试用例上调用模块本身，并自动检查结果是否正确。

大多数编程语言都至少有一个流行的库用于编写自动化单元测试，包括用于 Java 的 JUnit 和用于 Python 的 unittest ，用于 JavaScript 和 TypeScript 的 Mocha 。

Mocha 单元测试被编写为对函数`it()`调用，测试的名称作为其第一个参数，测试的主体（写为函数表达式）作为第二个参数。测试主体通常包含对正在测试的模块的一次或多次调用，然后使用断言函数（如 `assert` 、 `assert.strictEqual` 或 `assert.throws`）检查结果。

```ts
it("covers a < b", function() {
  assert.strictEqual(Math.max(1, 2), 2);
});
```

请注意， `assert.strictEqual`的参数顺序很重要。第一个参数应该是*实际*结果——代码实际执行的操作。第二个参数应该是测试想要看到的*预期*结果——通常是一个常数。断言还可以采用可选的消息字符串作为最后一个参数，您可以使用它来使测试失败更加清晰。

要将一组单元测试收集到测试套件中，请将它们放入对`describe()`调用中。

```ts
describe("Math.max", function() {
  it("covers a < b", function() {
    assert.strictEqual(Math.max(1, 2), 2);
  });

  it("covers a = b", function() {
    assert.strictEqual(Math.max(9, 9), 9);
  });

  it("covers a > b", function() {
    assert.strictEqual(Math.max(10, -9), 10);
  });
})
```

测试套件可以包含任意数量的`it()`函数，当您使用 Mocha 运行测试套件时，这些函数会*独立*运行。如果测试用例中的断言失败，则该测试用例立即返回，并且 Mocha 会记录该测试的失败。即使一项测试失败，套件中的其他测试仍将运行。

`strictEqual`使用的比较对于`string`和`number`等内置类型按预期工作，但它对数据结构不起作用。例如，以下所有断言都会*失败*：

- `strictEqual([], [])`
- `strictEqual([1], [1])`
- `strictEqual(new Set([1]), new Set([1]))`

这些可变数组和集合是否“相等”的问题是一个相当棘手的问题。因此，当我们测试返回这种结构的函数时，我们有两个选择：

- 使用 `assert.deepStrictEqual` ，它执行相等比较，将具有相同内容的数组、集合、映射和对象文字视为相等。一个危险的警告：仅使用它来比较内置数组、集合、映射和对象文字的结构，而不是其他任何东西。
- 对结构的属性使用简单的`assert`和`assert.strictEqual`断言。

例如，如果我们想断言`actual`值是一个包含单个元素`"hello"`的`Set` ，我们可以使用：

- `assert.deepStrictEqual(actual, new Set(["hello"]));`
- `assert(actual.has("hello"));` 
	`assert.strictEqual(actual.size, 1);`

## Documenting your testing strategy

最好写下用于创建测试套件的测试策略：分区、它们的子域以及选择每个测试用例覆盖哪些子域。

- 在`describe()`函数顶部的注释中记录分区和子域。
- 每个测试用例应按其涵盖的子域命名。
- 大多数测试套件将有多个分区，并且大多数测试用例将覆盖多个子域。

## Black box and glass box testing

黑箱测试(==**Black box testing**==)意味着仅从规范中选择测试用例，而不是从功能的实现中选择。

白箱测试(==**Glass box testing**==)意味着在了解功能实际实现方式的情况下选择测试用例。

在进行白箱测试时，必须注意测试用例*不需要*规范未特别要求的特定实现行为。

## Coverage

评价测试套件的一种方法是查询它对程序的执行程度。这个概念被称为覆盖范围(==*coverage*==)。以下是三种常见的覆盖范围：

- **Statement coverage**: 每条语句是否至少由一个测试用例运行？
- **Branch coverage**: 对于程序中的每个控制分支（例如`if`或`while`或`?:`三元表达式），分支的每一侧是否至少有一个测试用例？
- **Path coverage**: 是否每一种可能的分支组合（程序中的每一条路径）都被至少一个测试用例采用？

分支覆盖比语句覆盖更强（即需要更多的测试才能实现），路径覆盖比分支覆盖更强。

一种标准的测试方法是添加测试，直到测试套件达到足够的语句覆盖率。在实践中，语句覆盖率通常由代码覆盖率工具来测量，该工具会计算测试套件运行每个语句的次数。

## Unit and integration testing

单独测试模块可以使调试变得更加容易。当模块的单元测试失败时，您可以更加确信错误是在该模块中找到的，而不是在程序中的任何地方。

与单元测试(==**unit tests**==)相比，集成测试(==**integration test**==)测试模块的组合，甚至整个程序。如果你只有集成测试，那么当测试失败时，你必须寻找错误。但是集成测试仍然很重要，因为程序可能会在模块之间的连接处失败。

假设你正在构建一个文档搜索引擎。其中两个模块可能是 `load()` 和 `extract()` ，前者负责加载文件，后者负责将文档分割成单词：

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
```

这些函数可能会被另一个模块 `index()` 用来制作搜索引擎的索引：

```ts
/**
 * @returns an index mapping a word to the set of filenames
 *         containing that word, for all files in the input set
 */
function index(filenames: Set<string>): Map<string, Set<string>> {
    ...
    for (const file of files) {
        const doc = load(file);
        const words = extract(doc);
        ...
    }
    ...
}
```

程序员有时会犯的一个错误是，在为 `extract` 编写测试用例时，测试用例依赖于 `load` 才能正确。

最好单独考虑和测试 `extract` 。可以将文件内容存储为字面字符串，然后直接传给 `extract` 。不要在测试用例中实际调用 `load` ，因为 `load` 可能会出现错误！

需要注意的是， `index` 的单元测试不容易以这种方式隔离。当一个测试用例调用 `index` 时，它不仅要测试 `index` 内部代码的正确性，还要测试所有被 `index` 调用的函数的正确性。

## Automated regression testing

每次更改代码后运行所有测试称为回归测试(==**regression testing**==)。

每当您发现并修复错误时，请获取引发错误的输入并将其作为测试用例添加到自动化测试套件中。这种测试用例称为回归测试用例(*regression test*)。这有助于用好的测试用例填充您的测试套件。保存回归测试还可以防止重新引入错误的版本。

这个想法也导致了测试优先的调试(*test-first debugging*)。当错误出现时，立即为其编写一个引发该错误的测试用例，并立即将其添加到测试套件中，并使用一个名称将其标识为来自错误报告而不是分区，例如 `it("covers bug #1079")` 。

> **Automated regression testing** is a best-practice of modern software engineering.

## Iterative test-first programming

有效的软件工程并不遵循线性过程。实践上是迭代的(==**iterative**==)测试优先编程：

1. 编写函数的规范。
2. 编写测试来执行规范。当发现问题时，**迭代**规范和测试。
3. 编写实现。当发现问题时，**迭代**规范、测试和实现。

每个步骤都有助于验证前面的步骤。编写测试是理解规范的好方法。该规范可能不正确、不完整、不明确或缺少极端情况。尝试编写测试可以尽早发现这些问题，避免您浪费时间来实现有缺陷的规范。同样，编写实现可能会帮助您发现缺失或不正确的测试，或者提示您重新访问和修改规范。

**Plan for iteration:**

- 对于大型规范，首先只编写规范的一部分，然后继续测试和实现该部分，然后迭代更完整的规范。
- 对于复杂的测试套件，首先选择几个重要的分区，创建一个小型测试套件。继续进行通过这些测试的简单实现，然后使用更多分区迭代测试套件。
- 对于棘手的实现，首先编写一个简单的暴力实现来测试您的规范并验证测试套件。然后继续进行更困难的实现，并确信规范是好的并且测试是正确的。

迭代并不是试图从头到尾完美地解决一个问题，而是意味着尽快达成一个粗略的解决方案，然后稳步细化和改进它，这样你就有时间在必要时丢弃和返工。

## Summary

In this reading, we saw these ideas:

- Test-first programming. Write tests before you write code.
- Systematic testing with partitioning and boundary values, to design a test suite that is correct, thorough, and small.
- Glass box testing and statement coverage for filling out a test suite.
- Unit-testing each module, in isolation as much as possible.
- Automated regression testing to keep bugs from coming back.
- Iterative development. Plan to redo some work.

The topics of today’s reading connect to our three key properties of good software as follows:

- **Safe from bugs.** Testing is about finding bugs in your code, and test-first programming is about finding them as early as possible, right after you introduce them.
    
- **Easy to understand.** Systematic testing with a documented testing strategy makes it easier to understand how test cases were chosen and how thorough a test suite is.
    
- **Ready for change.** Correct test suites only depend on behavior in the spec, which allows the implementation to change within the confines of the spec. Besides, automated regression testing helps keep bugs from coming back when changes are made to code.