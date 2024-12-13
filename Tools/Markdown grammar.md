# Markdown grammar

涉及部分Obsidian中的语法

- [MARKDOWN 中文](https://markdown.cn/)
- [Markdown 教程](https://markdown.com.cn/)
- [Obsidian 中文帮助](https://publish.obsidian.md/help-zh/)

## 1. 标题

### 1.1. 另类标题

### 1.2. 段落

段落开头不要加空白

## 2. 换行

结尾空格 or `<br>`

## 3. 强调

### 3.1. Markdown 

- 一个星号 *斜体*
- 两个星号 **粗体**
- 三个星号 ***粗斜体***
- 两个波浪号 ~~删除样式~~
- 两个等号 ==高亮==
- 两个分号 `%%评论%%`

### 3.2. 非Markdown 

#### 3.2.1. <u>下底线</u>

`<u> </u>`

#### 3.2.2 上标与下标

`$ ^ or _ $` （斜体）
- $E=MC^2$
- $CO_2$

`<sup> </sup> or <sub> </sub>`
- E=MC<sup>2</sup>
- CO<sub>2</sub>

## 4. 引用

```
> Dorothy followed her through many of the beautiful rooms in her castle.
```

> Dorothy followed her through many of the beautiful rooms in her castle.

```
> Dorothy followed her through many of the beautiful rooms in her castle.
> 
>> The Witch bade her clean the pots and kettles and sweep the floor and keep the fire fed with wood.
```

> Dorothy followed her through many of the beautiful rooms in her castle.
> 
>> The Witch bade her clean the pots and kettles and sweep the floor and keep the fire fed with wood.

区块前后最好插入一行空行

### Obisidian Callouts提示语法

```
> [!type] Title
> 内容
```

Types: 
- note
- abstract, summary, tldr
- info, todo
- tip, hint, important
- success, check, done
- question, help, faq
- warning, caution, attention
- failure, fail, missing
- danger, error
- bug
- example
- quote, cite

>[!INFO]- Foldable callouts
>折叠的功能：使用 `[!type]-`

## 5. 列表

### 5.1. 有序列表

1. 以"数字、点、空白"开头
2. 数字不必按照顺序排列
3. 数字以列表的第一个项目的数字开始依次增加
4. 开头四个空格或"Tab"形成第二阶层列表
5. 自动重新编号：按"Tab"再按"Shift+Tab"

### 5.2. 无序列表

- 以"星号、空白"开头
- 可用减号、加号代替星号
- 要在保留列表连续性的同时在列表中添加另一种元素，请将该元素缩进四个空格或一个制表符

## 6. 代码

### 6.1. 行内代码 

要将单词或短语表示为代码，请将其包裹在反引号 (`` ` ``) 中。

如果你要表示为代码的单词或短语中包含一个或多个反引号，则可以通过将单词或短语包裹在双反引号(` `` `)中。

```
``Use `code` in your Markdown file.``
```

``Use `code` in your Markdown file.``

### 6.2. 代码块

```
<Tab>public class HelloWorld {
<Tab>	public static void main (String[] args) {
<Tab>		System.out.println("Hello world!");
<Tab>	}
<Tab>}
```

	public class HelloWorld {
		public static void main (String[] args) {
			System.out.println("Hello world!");
		}
	}

````
```java
public class HelloWorld {
	public static void main (String[] args) {
		System.out.println("Hello world!");
	}
}
```
````

```java
public class HelloWorld {
	public static void main (String[] args) {
		System.out.println("Hello world!");
	}
}
```


1. 要创建代码块，请将代码块的每一行缩进至少四个空格或一个制表符。
2. 用三个倒引号开头，结尾行用三个倒引号
3. 三个倒引号也可以用波浪号
4. 开头倒引号后可以指定代码使用语言

## 7. 分割线

三个以上的星号、减号或下划线。
分割线前后插入一个空行。

`---`

---

## 8. 网址链接

### 8.1 链接语法

#### 8.1.1 带Title的链接

`[超链接显示名](超链接地址 "超链接title")`

Obsidian中超链接title不显示。

```
这是一个链接 [Markdown语法](https://markdown.com.cn)
```

这是一个链接 [Markdown语法](https://markdown.com.cn)

#### 8.1.2 网址和Email地址

使用**尖括号**可以很方便地把URL或者email地址变成可点击的链接。

```
<https://markdown.com.cn>
<fake@example.com>
```

<https://markdown.com.cn>
<fake@example.com>

#### 8.1.3 带格式化的链接

**强调**链接, 在链接语法前后增加星号。 要将链接表示为代码，请在方括号中添加反引号。

```
I love supporting the **[EFF](https://eff.org)**.
This is the *[Markdown Guide](https://www.markdownguide.org)*.
See the section on [`code`](#code).
```

I love supporting the **[EFF](https://eff.org)**.
This is the *[Markdown Guide](https://www.markdownguide.org)*.
See the section on [`code`](#code).

#### 8.1.4 网址里有空格

1. 把空格改成「%20」
2. 网址改成<链接网址>  
3. Obsidian在遇到http://、https://开头的网址文字时，会自动变成网址超链接

### 8.2 Obsidian内部链接

```
[[本地笔记路径]]
[[本地笔记路径|特定的显示文本]]
[[本地笔记路径#标签]]
[[本地笔记路径^代码块]]
```

### 8.3 引用类型链接

#### 8.3.1 链接的第一部分格式

引用类型的链接的第一部分使用两组括号进行格式设置。第一组方括号包围应显示为链接的文本。第二组括号显示了一个标签，该标签用于指向您存储在文档其他位置的链接。

- `[hobbit-hole][1]`
- `[hobbit-hole] [1]`

#### 8.3.2 链接的第二部分格式

1. 放在括号中的标签，其后紧跟一个冒号和至少一个空格（例如`[label]:`）。
2. 链接的URL，可以选择将其括在尖括号中。
3. 链接的可选标题，可以将其括在双引号，单引号或括号中。

- `[1]: https://en.wikipedia.org/wiki/Hobbit#Lifestyle`
- `[1]: https://en.wikipedia.org/wiki/Hobbit#Lifestyle "Hobbit lifestyles"`
- `[1]: https://en.wikipedia.org/wiki/Hobbit#Lifestyle 'Hobbit lifestyles'`
- `[1]: https://en.wikipedia.org/wiki/Hobbit#Lifestyle (Hobbit lifestyles)`
- `[1]: <https://en.wikipedia.org/wiki/Hobbit#Lifestyle> "Hobbit lifestyles"`
- `[1]: <https://en.wikipedia.org/wiki/Hobbit#Lifestyle> 'Hobbit lifestyles'`
- `[1]: <https://en.wikipedia.org/wiki/Hobbit#Lifestyle> (Hobbit lifestyles)`

可以将链接的第二部分放在Markdown文档中的任何位置。

### 8.4 嵌入网页

`<iframe src="网址"></iframe>`

<iframe src="http://www.example.com"></iframe>



### 8.5 自动网址链接

```
http://www.example.com
```

http://www.example.com

**禁用自动URL链接**

```
`http://www.example.com`
```

`http://www.example.com`

## 9. 图片

*markdown语法*
```
![显示文字](图片网址链接 "提示文字")
![显示文字|显示宽度](图片网址链接 "提示文字")
![显示文字|显示宽度x高度](图片网址链接 "提示文字")
![显示文字][引用代码]
```

*obsidian-wiki链接* （相对仓库的路径）
```
![[本地图片路径]]  
![[本地图片路径|显示宽度]]  
![[本地图片路径|显示宽度x高度]]
```

 ***obsidian嵌入内部链接***
```
![[内部链接]]  
![[内部链接|特定显示文字]]  
![[内部链接#标题]]  
![[内部链接^区块代码]]
```

## 10. 转义字符语法

要显示原本用于格式化 Markdown 文档的字符，请在字符前面添加反斜杠字符 \\ 。

## 11. 表格

### 11.1. 简单表格

```
标题1 | 标题2 | 标题3
--|--|--
111 | 222 | 333
444 | 555 | 666
```

| 标题1 | 标题2 | 标题3 |
| --- | --- | --- |
| 111 | 222 | 333 |
| 444 | 555 | 666 |

### 11.2. 有对齐功能的表格

```
| Syntax      | Description | Test Text     |
| :---        |    :----:   |          ---: |
| Header      | Title       | Here's this   |
| Paragraph   | Text        | And more      |
```

| Syntax    | Description |   Test Text |
| :-------- | :---------: | ----------: |
| Header    |    Title    | Here's this |
| Paragraph |    Text     |    And more |

- 使用表格的HTML字符代码（`&#124;`）或者`\|`在表中显示竖线（`|`）字符。
- 使用 `<br>` 表示表格里的换行。

## 12. 任务

```
- [x] Write the press release
- [?] Update the website
- [ ] Contact the media
```

- [x] Write the press release
- [?] Update the website
- [ ] Contact the media

## 13. 脚注

创建脚注参考，请在方括号（`[^1]`）内添加插入符号和标识符。标识符可以是数字或单词，但不能包含空格或制表符。标识符仅将脚注参考与脚注本身相关联-在输出中，脚注按顺序编号。

```
Here's a simple footnote,[^1] and here's a longer one.[^bignote]

[^1]: This is the first footnote.

[^bignote]: Here's one with multiple paragraphs and code.

    Indent paragraphs to include them in the footnote.

    `{ my code }`

    Add as many paragraphs as you like.
```

Here's a simple footnote,[^1] and here's a longer one.[^bignote]

[^1]: This is the first footnote.

[^bignote]: Here's one with multiple paragraphs and code.

    Indent paragraphs to include them in the footnote.

    `{ my code }`

    Add as many paragraphs as you like.
