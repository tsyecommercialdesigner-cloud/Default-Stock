---
title: Markdown语法规范
created: 2026-05-08
tags:
  - Markdown
  - Programming_language
  - Textual_format
source: https://m.ituring.com.cn/article/775
---
# 准确性检查与历史局限性分析

---
### (这部分是由AI进行内容审查而自动生成的)

您提供的文档中关于 Markdown 语法的描述是**非常准确的**。它忠实地翻译并反映了 Markdown 创始人 John Gruber 在 2004 年发布的**原生 Markdown 规范**（Original Markdown）。

**历史局限性：**
由于原生 Markdown 诞生较早，它的设计初衷主要是为了替代简单的 HTML 邮件和博客排版，因此存在明显的局限性：
- **代码块书写繁琐**：只能通过缩进 4 个空格或 1 个 TAB 来实现，不支持语法高亮。
- **缺乏复杂排版元素**：原生语法不支持表格（Tables）、脚注（Footnotes）、待办事项（Task lists）以及数学公式（Math equations）等。
- **缺乏现代笔记需求**：不支持高亮、删除线、以及 Obsidian 等现代工具中常用的呼出块（Callouts）。

如今我们常用的 Markdown（如 GitHub Flavored Markdown (GFM)、CommonMark 以及 Obsidian 风格的 Markdown）已经对原生语法进行了大量扩展。


# 现代 Markdown 扩展语法（新增与改进）

---

由于原生 Markdown 诞生较早（2004年），其基础语法在应对现代排版、编程和笔记需求时存在一定局限性。如今主流的 Markdown 解析器（如 GitHub Flavored Markdown、Obsidian 等）都引入了丰富的扩展语法。
## 以下是现代 Markdown 中最常用的新增和改进内容：

### 1. 围栏代码块 (Fenced Code Blocks)
原生 Markdown 需要缩进四个空格来表示代码块，这在复制粘贴时非常不便。现代 Markdown 支持使用三个反引号（\`\`\`）来包裹代码，并支持指定编程语言以实现语法高亮：

```markdown
```python
def hello_world():
    print("Hello, World!")
```

### 2. 表格 (Tables)
原生 Markdown 不支持表格，现代语法通过竖线 `|` 和连字符 `-` 轻松创建表格：

| 语法        | 描述    |
| --------- | ----- |
| Header    | Title |
| Paragraph | Text  |

### 3. 任务列表 (Task Lists)
现代 Markdown 引入了待办事项列表，使用带有空格或 `x` 的方括号来表示任务状态：

- [ ] 这是一个未完成的任务
- [x] 这是一个已完成的任务

### 4. 删除线与高亮 (Strikethrough & Highlights)
- 删除线：使用两个波浪号包裹文本，如 `~~被删除的文字~~`。
- 高亮：在 Obsidian 等现代笔记软件中，使用两个等号包裹文本可以实现高亮，如==重要的高亮文字==。

### 5. 数学公式 (Math Equations)
现代笔记软件（如 Obsidian）原生集成了 MathJx，支持 LaTeX 数学公式，使用 `$` 符号包裹：
- 行内公式：$E=mc^2$
- 块级公式：
$$
\int_{a}^{b} x^2 dx
$$

### 6. 脚注 (Footnotes)
支持在文中插入脚注引用，并在文档末尾添加注释内容，这对于学术写作和长文非常有用：
这是一个带有脚注的文本[^1]。

[^1]: 这是脚注的详细解释内容。

### 7. 标注/呼出块 (Callouts / Admonitions)
在 Obsidian 等现代工具中，可以通过在引用块中添加特定标记来生成带有图标和背景色的提示块，极大地丰富了文档的表现力：

> [!info] 提示
> 这是一个信息标注块，用于补充说明。

> [!warning] 注意
> 这是一个警告标注块，用于提示重要风险。


# 原生Markdown语法规范(2004年版)

---

## 概述

### 哲学

Markdown 的目标是易读易写。

Markdown 强调可读性高于一切。一份 Markdown 格式的文档应该能直接以纯文本方式发布，而不致一眼看过去满眼都是标签和格式化指令。Markdown 的语法确实受了几种现有的 text 转 HTML 过滤器影响－－包括 Setext, atx, Textile, reStructuredText, Grutatext, 和 EtText -- 其中对 Markdown 语法影响最大的单一来源是纯文本的 Email 格式。

为实现这一目标，Markdown 的语法几乎全部由标点符号构成，这些标点符号都是精心挑选而来，尽量做到能望文生义。如星号括着一个单词（Markdown中表示强调）看上去就像 *强调*。Markdown的列表看上去就像列表；Markdown的引文就象引文，和你使用 email 时的感觉一样。

### 内嵌HTML

Markdown 的语法为“方便地在网上写作”这一目标而生。

Markdown 不是 HTML 替代品，也不是为了终结 HTML。它的语法非常简单，只相当于 HTML 标签的一个非常非常小的子集。它并非是为了更容易输入 HTML 标签而创造一种新语法。在我看来，HTML 标签已经够容易书写的了。Markdown 的目标是让（在网上）让读文章、写文章、修改文章更容易。HTML 是一种适合发表的格式；而 Markdown 是一种书写格式。正因如此，Markdown 的格式化语法仅需解决用纯文本表达的问题。

对 Markdown 语法无法支持的格式，你可以直接用 HTML。你不需要事先声明或者使用什么定界符来告诉 Markdown 要写 HTML 了，你直接写就是了。

唯一的限制是那些块级 HTML 元素 -- 如 `<div>`, `<table>`, `<pre>`, `<p>` 等等 -- 必须使用空行与相邻内容分开，并且块元素的开始和结束标签之前不要留有空格或 TAB。Markdown 足够聪明不会添加额外的(也是不必要的) `<p>` 标签包住这些块元素标签。

下面这个例子，在一篇 Markdown 文章中添加了一个 HTML 表格：

```html
这是一个普通的段落。

<table>
<tr>
<td>
Foo
</td>
</tr>
</table>

这是另一个普通的段落。
```

注意一点，不要在块级 HTML 元素内使用 Markdown 格式化命令，Markdown 不会处理它们。比如你不要在一个 HTML 块中使用 `*emphasis*` 这样的 Markdown 格式化命令。

行内 HTML 标签 -- 如 `<span>`, `<cite>`, 或 `<del>` -- 在一个 Markdown 段落里、列表中、或者标题中－－随便用。如果需要，你甚至可以用 HTML 标签代替 Markdown 格式化命令。比方你可以直接用 HTML 标签 `<a>` 或 `<img>` 而不使用 Markdown 的链接和图片语法，随你的便。

不同于这些块级 HTML 元素，在 HTML 行内元素内的 Markdown 语法标记会被正确处理。

### 自动转换特殊字符

在 HTML 中，有两个字符需要特殊对待：`<` 和 `&`。`<` 用于标签开始，`&` 用于标识 HTML 实体。如果打算把它们当成普通字符，你必须使用反引号转义它们，如 `&lt;` 和 `&amp;`。

对一些互联网作家来说，`&` 符号特别使人烦恼。如果你打算写 'AT&T'，你就得写 `AT&amp;T`。甚至在 URL 中也得想着转义 `&` 符号。比方你打算写：
`http://images.google.com/images?num=30&q=larry+bird`
你就得在 A 标签中把 href 属性中的 URL 编码成：
`http://images.google.com/images?num=30&amp;q=larry+bird`
不用说，这很容易忘。这往往是那些良构 HTML 站点中最容易出错的地方。

在 Markdown 中，你尽管自然的使用这些字符，只需要关心那些必要的转义。如果使用在 HTML 实体中使用 `&` 符号，它会保持不变；而在其它场合，它会转换成 `&amp;`。

所以，如果你打算在文章中书写版权符号，你可以这样写：
`&copy;`
Markdown 不会碰它。然而如果你书写：
`AT&T`
Markdown 就会把它翻译成：
`AT&amp;T`

类似的，既然 Markdown 支持内嵌 HTML，如果你使用 `<` 作为 HTML 标签定界符，Markdown 就会把它们当成 HTML 标签定界符。可是如果你书写：
`4 < 5`
Markdown 就会把它翻译成：
`4 &lt; 5`

然而，在 Mardown 代码行内标记和块级标记之中，`<` 和 `&` 始终会被自动编码。这使得在 Markdown 文件中书写 HTML 代码更容易。(相对于纯 HTML。如果想在纯 HTML 里贴一段 HTML 代码，那才是糟糕透顶，必须对代码中的每一个 `<` 和 `&` 都转义才成。)

---

## 块级元素

### 段落和换行

一个段落由一行或多个相关文本行构成。段落之间用一个或多个空行分隔。（一个空行就是一个看上去什么也没有的行－－如果一行什么也没有或者只有空格和 TAB 都会被视为空行）正常的段落不要以空白或 TAB 字符开始。

一行或多个相关文本行意味着 Markdown 支持“硬折行”。这一点与其它 text 转 HTML 的程序完全不同（包括 Moveable Type 的“Convert Line Breaks”选项），它们会将段落中的每一个换行符转换成 `<br />` 标签。

如果你确实需要使用 Markdown 插入一个 `<br />` 换行符，只需要在每一行的末尾以两个或更多个空格符号结束，然后再打回车键。

没错，在 Markdown 里生成一个 `<br />` 稍稍有一点麻烦，但那种简单的“把每一个换行符都转换成 `<br />` 规则”并不适用于 Markdown。Markdown Email 风格的 blockquoting 和 multi-paragraph 更好用 -- 并且更美观 -- 在你用换行符对其格式化时。

### 标题

Markdown 支持两种风格的标题，Setext 和 atx。

Setext-风格的一级标题下面一行使用等号符号，二级标题下面使用连字符符号，例如：
```markdown
这是一个一级标题
=============

这是一个二级标题
-------------
```
至少有一个 `=` 和 `-` 就能正常工作。

Atx-风格的标题在每行的开头使用1－6个井号字符，分别对应标题级别1－6。例如：
```markdown
# 这是一级标题
## 这是二级标题
###### 这是六级标题
```

如果愿意, 你也可以 "结束" atx-风格的标题。这纯粹是美观考虑--如果你觉得这样会看上更舒服些的话。结束用的井号个数随便，不必与起始井号数量相同 (起始井号的数量决定标题级别)：
```markdown
# 这是一级标题 #
## 这是二级标题 ##
### 这是三级标题 ######
```

### 引用块

Markdown 使用 Email 风格的 `>` 字符引用块。如果你熟悉 Email 中的引用块，你就知道在 Markdown 中如何使用引用块。如果每一行你都使用硬换行并在行首放一个 `>` 符号，看上去会很美观：

```markdown
> This is a blockquote with
> two paragraphs. Lorem ipsum dolor sit amet,
> consectetuer adipiscing elit. Aliquam hendrerit mi posuere lectus.
> Vestibulum enim wisi, viverra nec, fringilla in, laoreet vitae, risus.
> 
> Donec sit amet nisl. Aliquam semper ipsum sit amet velit. Suspendisse
> id sem consectetuer libero luctus adipiscing.
```

（如果觉得每行写一个 `>` 太累，）Markdown 允许你偷懒，你只需在硬换行段落的第一行之前放一个 `>` 号：

```markdown
> This is a blockquote with two paragraphs. Lorem ipsum dolor sit amet,
consectetuer adipiscing elit. Aliquam hendrerit mi posuere lectus.
Vestibulum enim wisi, viverra nec, fringilla in, laoreet vitae, risus.

> Donec sit amet nisl. Aliquam semper ipsum sit amet velit. Suspendisse
id sem consectetuer libero luctus adipiscing.
```

只需要多加一个 `>`，就得到嵌套的引用块 (即引用块中的引用块)：
```markdown
> This is the first level of quoting.
>
> > This is nested blockquote.
>
> Back to the first level.
```

引用块中可包含其它 Markdown 元素，如标题、列表和代码块：
```markdown
> ## This is a header.
> 
> 1. This is the first list item.
> 2. This is the second list item.
> 
> Here's some example code:
> 
>     return shell_exec("echo $input | $markdown_script");
```

### 列表

Markdown 支持有序列表和无序列表。

无序列表可使用星号、加号和连字符（这几个符号是等价的，你喜欢哪个就用哪个）作为列表标记：
```markdown
* Red
* Green
* Blue
```
等同于：
```markdown
+ Red
+ Green
+ Blue
```
也等同于：
```markdown
- Red
- Green
- Blue
```

有序列表则使用数字加英文句点：
```markdown
1. Bird
2. McHale
3. Parish
```

有一点需要注意，你在列表中输入的标记数字并不会反映到 Markdown 输出的 HTML 之中。上面这个列表 Markdown 会输出为：
```html
<ol>
<li>Bird</li>
<li>McHale</li>
<li>Parish</li>
</ol>
```
即使你写成下面这样（数字无序），都会得到一模一样（但正确的）输出：
```markdown
3. Bird
1. McHale
8. Parish
```

列表标记通常从左边界开始，至多可以有三个空格的缩进。列表标记之后至少要跟一个空格或 TAB。

如果列表项之间用空行分隔，Markdown 就会在 HTML 输出中使用 `<p>` 标签包裹列表项。列表项有可能由多个段落组成，列表项的每个后续段落必须缩进至少4个空格或者一个 TAB。要在列表项中使用代码块，代码块需要缩进两次 -- 8个空格或者两个 TAB。

### 代码块

我们经常在写有关编程或标记语言源代码时用到预格式化的代码块。不像格式化普通段落，代码块中的行会按字面进行解释。

在 Markdown 中要生成一个代码块，只需要在代码块内容的每一行缩进至少四个空格或者一个 TAB。比如：
```markdown
This is a normal paragraph:

    This is a code block.
```

Markdown会生成：
```html
<p>This is a normal paragraph:</p>
<pre><code>This is a code block.
</code></pre>
```

代码块在遇到没有缩进的一行，或者文件末尾时自动结束。
在代码块中，`&` 符号和 `<`、`>` 会自动转换成 HTML 实体。

### 水平线

如果在一行里只放三个或更多个连字符，或星号或下划线，你就会得到一个水平线标记(`<hr />`)。下面每一行都会得到一个水平线：

```markdown
* * *
***
*****
- - -
---------------------------------------
```

---

## 行内元素

### 链接

Markdown 支持两种风格的链接：行内链接 和 引用链接。
两种风格的链接，链接文本都放在中括号之内 `[square brackets]`。

要生成一个行内链接，在链接文本之后紧跟用一对小括号。小括号里放链接地址和可选的链接 title。如果提供链接 title 的话，链接 title 要用引号包起来。例如：
```markdown
这是一个 an example [<sup>2</sup>](http://example.com/ "Title") 行内链接。这个链接 [<sup>3</sup>](http://example.net/) 没有title属性。
```

引用风格的链接，在链接文本之后紧跟又一对中括号。这对中括号里放的是该链接的标识符（可以理解为别名）：
```markdown
这是一个引用型链接 [示例][id]。
```

然后，我们可以在文档的任意位置，像下面这样定义链接标识与链接的对应关系（一行一个链接）：
```markdown
[id]: http://example.com/ "Optional Title Here"
```

隐式链接标识允许我们省略链接标识，这时链接文本本身就是链接标识。例如，使用"Google"文本链接到 google.com 站点：
```markdown
[Google][]
```
然后这样定义它的链接：
```markdown
[Google]: http://google.com/
```

### 强调

Markdown 使用星号(`*`)和下划线(`_`)作为表示强调。用一个 `*` 或 `_` 包裹的文本会使用 HTML `<em>` 标签包裹; 用两个 `*` 或 `_` 包裹的文本会使用 HTML `<strong>` 标签包裹。如：

```markdown
*single asterisks*
_single underscores_
**double asterisks**
__double underscores__
```

如果不想让 Markdown 解释这两个元字符，就转义它：
```markdown
\*this text is surrounded by literal asterisks\*
```

### 代码

要在行内表示部分代码，用反引号(`)包住它。与预格式代码块不同和，行内代码用于段落之内。例如：
```markdown
Use the `printf()` function.
```

要在一个行内代码中使用反引号(`)本身，用多个反引号作为定界符包住它：
```markdown
``There is a literal backtick (`) here.``
```

### 图片

Markdown 使用了类似链接语法来表示图片，同样有两种风格：行内图片 和 引用图片。

行内图片语法示例：
```markdown
!Alt text [<sup>4</sup>](/path/to/img.jpg)
!Alt text [<sup>5</sup>](/path/to/img.jpg "Optional title")
```

引用图片语法如下：
```markdown
![Alt text][id]
```
这里 "id" 是图片引用标识。图片引用定义的语法与链接定义完全相同：
```markdown
[id]: url/to/image "Optional title attribute"
```

在写这篇文章时，Markdown 还没有语法指定图片的大小，如果这一点对你特别重要，你可以直接使用 `<img>` 标签。

---

## 杂七杂八

### 自动链接

Markdown 提供了一种快捷方式"自动地"定义链接和 Email 地址：直接用一对尖括号把 URL 或 Email 地址包住。这表示链接文本就是 URL 本身，Email 文本就是 Email 本身。例如：
```markdown
<http://example.com/>
```
Markdown 会将它转换为：
```html
<a href="http://example.com/">http://example.com/</a>
```

### 反斜线转义

Markdown 允许你使用反斜线转义那些 Markdown 元字符，让它们失去原有的“魔力”。举个例子，如果你确实想用星号包住一个词组（而不是想得到 `<em>` 标签），就可以在星号之前使用反斜线将其转义。

Markdown中，以下字符支持使用反斜线转义：
```text
\   反斜线
`   反引号
*   星号
_   下划线
{}  大括号
