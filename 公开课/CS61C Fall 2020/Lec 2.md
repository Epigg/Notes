# Number Representation

### Bits can represent anything

> 数据输入：模拟（Analog） → 数字（Digital）

- 现实世界是模拟的！
- 要导入模拟信息：
    - 采样（Sample）
    - 量化（Quantize）

数字数据不一定源于模拟。

> 重要理念：位可以表示任何东西！！

## Binary, Decimal, Hex

- Binary 二进制
- Decimal 十进制
- Hexadecimal 十六进制

> 每个进制都是“十进制”...

三者之间的转换三角关系

## Number Representation

- 无符号整数（unsigned int）
	例如 00000~11111 表示 0~31
- 溢出（Overflow）
	数码位数不够表示数字，造成溢出

### Represent Negative Numbers

- 原码（Sign-Magnitude）
	最高位表示符号，其余位表示大小
- 反码（One's Complement）
	若符号位为负，其余位全部取反
- 补码（Two's Complement）

补码转化为十进制，最高位的系数增加一个负号，例如 1101 = -8 + 4 + 1 = -3 。

对于负数，补码取反码加一得到原码，原码取反码加一得到补码。

- $2^{N-1}$ 个非负数
- $2^{N-1}$ 个负数
- 一个零

绝大部分机器中存储整数类型是采用补码的。

### Bias Encoding

- 无符号数 + 偏置（bias） 
- $bias=-(2^{N-1}-1)$ 
- 一个零

例如，对于 N=5 ，用 00000 表示最小的数 -15，即偏置为 -15，则此时 00000~11111 表示 -15~16。
