---
title: HTML、CSS 与 JavaScript
created: 2026-05-09
tags:
  - HTML
  - Markdown
  - Programming_language
  - Cascading_Style_Sheets
  - Java_Script
---
# 一、Web 前端三大核心语言

| 技术         | 作用          |
| ---------- | ----------- |
| HTML       | 负责网页结构与内容   |
| CSS        | 负责网页样式与布局   |
| JavaScript | 负责网页交互与动态行为 |

一个网页通常由三部分组成：

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>网页标题</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <h1>Hello Web</h1>
  <p>这是一个网页。</p>

  <script src="script.js"></script>
</body>
</html>
```

---

# 二、HTML 语法规范

HTML，全称 HyperText Markup Language，超文本标记语言，用于描述网页结构。

## 1. 基本结构

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>网页标题</title>
</head>
<body>
  页面内容
</body>
</html>
```

## 2. 标签语法

HTML 由标签组成：

```html
<标签名>内容</标签名>
```

例如：

```html
<p>这是一段文字</p>
```

## 3. 单标签

有些标签不需要结束标签：

```html
<br>
<hr>
<img src="image.jpg" alt="图片">
<input type="text">
```

## 4. 属性语法

标签可以拥有属性：

```html
<a href="https://example.com" target="_blank">访问网站</a>
```

常见属性：

| 属性 | 作用 |
|--- |--- |
| id | 设置唯一标识 |
| class | 设置类名，常用于 CSS |
| style | 设置行内样式 |
| title | 鼠标悬停提示 |
| href | 链接地址 |
| src | 资源路径 |
| alt | 图片替代文本 |

## 5. 注释

```html
<!-- 这是 HTML 注释 -->
```

## 6. 标签嵌套规则

正确写法：

```html
<div>
  <p>这是一段文字</p>
</div>
```

错误写法：

```html
<div>
  <p>这是一段文字
</div>
</p>
```

---

# 三、HTML5 新增内容

HTML5 是现代网页开发的基础，增加了语义化标签、多媒体标签、表单增强、绘图、本地存储等功能。

## 1. HTML5 文档声明

```html
<!DOCTYPE html>
```

相比旧版本更简洁。

---

## 2. HTML5 语义化标签

语义化标签让网页结构更清晰，也有利于 SEO 和可访问性。

| 标签 | 用法 |
|--- |--- |
| `<header>` | 页面或区域的头部 |
| `<nav>` | 导航区域 |
| `<main>` | 页面主体内容 |
| `<section>` | 页面中的一个内容区块 |
| `<article>` | 独立文章内容 |
| `<aside>` | 侧边栏或附属内容 |
| `<footer>` | 页面或区域底部 |
| `<figure>` | 图片、图表等独立内容 |
| `<figcaption>` | `<figure>` 的标题说明 |
| `<mark>` | 高亮文本 |
| `<time>` | 时间或日期 |
| `<details>` | 可展开的详情区域 |
| `<summary>` | `<details>` 的标题 |

示例：

```html
<header>
  <h1>我的网站</h1>
</header>

<nav>
  <a href="#">首页</a>
  <a href="#">文章</a>
</nav>

<main>
  <article>
    <h2>文章标题</h2>
    <p>文章内容。</p>
  </article>
</main>

<footer>
  <p>版权所有</p>
</footer>
```

---

## 3. HTML5 多媒体标签

| 标签 | 用法 |
|--- |--- |
| `<audio>` | 插入音频 |
| `<video>` | 插入视频 |
| `<source>` | 为音频或视频指定多个资源 |
| `<track>` | 为视频添加字幕 |

示例：

```html
<audio controls>
  <source src="music.mp3" type="audio/mpeg">
</audio>

<video controls width="400">
  <source src="movie.mp4" type="video/mp4">
</video>
```

---

## 4. HTML5 图形绘制

| 标签 | 用法 |
|--- |--- |
| `<canvas>` | 使用 JavaScript 绘制图形 |
| `<svg>` | 绘制矢量图形 |

示例：

```html
<canvas id="myCanvas" width="300" height="150"></canvas>

<svg width="100" height="100">
  <circle cx="50" cy="50" r="40" fill="red" />
</svg>
```

---

## 5. HTML5 表单增强

### 新增 input 类型

| 类型 | 用法 |
|--- |--- |
| `email` | 邮箱输入 |
| `url` | URL 输入 |
| `number` | 数字输入 |
| `range` | 滑块输入 |
| `date` | 日期选择 |
| `time` | 时间选择 |
| `datetime-local` | 本地日期时间 |
| `month` | 月份选择 |
| `week` | 周选择 |
| `color` | 颜色选择 |
| `search` | 搜索框 |
| `tel` | 电话号码输入 |

示例：

```html
<input type="email" placeholder="请输入邮箱">
<input type="date">
<input type="color">
<input type="range" min="0" max="100">
```

### 新增表单属性

| 属性 | 用法 |
|--- |--- |
| placeholder | 输入提示 |
| required | 必填 |
| autofocus | 自动聚焦 |
| autocomplete | 自动完成 |
| min | 最小值 |
| max | 最大值 |
| step | 步长 |
| pattern | 正则校验 |
| multiple | 允许多个值 |

示例：

```html
<input type="text" placeholder="用户名" required>
<input type="number" min="1" max="10">
```

---

## 6. HTML5 本地存储

HTML5 提供浏览器端存储能力。

| API | 用法 |
|--- |--- |
| localStorage | 长期存储数据 |
| sessionStorage | 会话期间存储数据 |

```javascript
localStorage.setItem("name", "Tom");
let name = localStorage.getItem("name");

sessionStorage.setItem("token", "123456");
```

---

# 四、HTML5 以前的内容

HTML5 以前的内容主要包括基础标签、文本标签、列表、表格、表单、链接、图片、框架等。

---

## 1. 文档基础标签

| 标签 | 用法 |
|--- |--- |
| `<html>` | HTML 文档根元素 |
| `<head>` | 文档头部，放置元信息 |
| `<title>` | 网页标题 |
| `<meta>` | 元数据，如编码、关键词 |
| `<body>` | 页面主体内容 |
| `<link>` | 引入外部资源，如 CSS |
| `<style>` | 编写内部 CSS |
| `<script>` | 编写或引入 JavaScript |

示例：

```html
<head>
  <meta charset="UTF-8">
  <title>页面标题</title>
  <link rel="stylesheet" href="style.css">
</head>
```

---

## 2. 文本标签

| 标签 | 用法 |
|--- |--- |
| `<h1>` ~ `<h6>` | 标题，`<h1>` 最大，`<h6>` 最小 |
| `<p>` | 段落 |
| `<br>` | 换行 |
| `<hr>` | 水平线 |
| `<strong>` | 重要文本，通常加粗 |
| `<b>` | 加粗文本 |
| `<em>` | 强调文本，通常斜体 |
| `<i>` | 斜体文本 |
| `<u>` | 下划线 |
| `<s>` | 删除线 |
| `<sup>` | 上标 |
| `<sub>` | 下标 |
| `<small>` | 小号文字 |
| `<pre>` | 保留空格和换行 |
| `<code>` | 代码文本 |
| `<blockquote>` | 块级引用 |
| `<q>` | 短引用 |
| `<abbr>` | 缩写说明 |
| `<address>` | 地址信息 |

示例：

```html
<h1>一级标题</h1>
<p>这是一个段落。</p>
<strong>重要内容</strong>
<em>强调内容</em>
<code>console.log("Hello");</code>
```

---

## 3. 链接标签

| 标签 | 用法 |
|--- |--- |
| `<a>` | 创建超链接 |

常见属性：

| 属性 | 用法 |
|--- |--- |
| href | 链接地址 |
| target | 打开方式 |
| title | 提示文本 |

示例：

```html
<a href="https://example.com" target="_blank">访问网站</a>
<a href="#top">返回顶部</a>
<a href="mailto:test@example.com">发送邮件</a>
```

---

## 4. 图片标签

| 标签 | 用法 |
|--- |--- |
| `<img>` | 插入图片 |

常见属性：

| 属性 | 用法 |
|--- |--- |
| src | 图片路径 |
| alt | 图片无法显示时的替代文本 |
| width | 图片宽度 |
| height | 图片高度 |

示例：

```html
<img src="logo.png" alt="网站 Logo" width="200">
```

---

## 5. 列表标签

| 标签 | 用法 |
|--- |--- |
| `<ul>` | 无序列表 |
| `<ol>` | 有序列表 |
| `<li>` | 列表项 |
| `<dl>` | 定义列表 |
| `<dt>` | 定义项名称 |
| `<dd>` | 定义项解释 |

示例：

```html
<ul>
  <li>苹果</li>
  <li>香蕉</li>
</ul>

<ol>
  <li>第一步</li>
  <li>第二步</li>
</ol>

<dl>
  <dt>HTML</dt>
  <dd>网页结构语言</dd>
</dl>
```

---

## 6. 表格标签

| 标签 | 用法 |
|--- |--- |
| `<table>` | 表格 |
| `<caption>` | 表格标题 |
| `<thead>` | 表头区域 |
| `<tbody>` | 表格主体 |
| `<tfoot>` | 表格底部 |
| `<tr>` | 表格行 |
| `<th>` | 表头单元格 |
| `<td>` | 普通单元格 |
| `<colgroup>` | 表格列组 |
| `<col>` | 表格列 |

示例：

```html
<table border="1">
  <caption>学生表</caption>
  <tr>
    <th>姓名</th>
    <th>年龄</th>
  </tr>
  <tr>
    <td>张三</td>
    <td>18</td>
  </tr>
</table>
```

---

## 7. 表单标签

| 标签 | 用法 |
|--- |--- |
| `<form>` | 表单 |
| `<input>` | 输入控件 |
| `<textarea>` | 多行文本框 |
| `<button>` | 按钮 |
| `<select>` | 下拉列表 |
| `<option>` | 下拉选项 |
| `<optgroup>` | 选项分组 |
| `<label>` | 表单说明文字 |
| `<fieldset>` | 表单分组 |
| `<legend>` | 分组标题 |

示例：

```html
<form action="/login" method="post">
  <label>用户名：</label>
  <input type="text" name="username">

  <label>密码：</label>
  <input type="password" name="password">

  <button type="submit">登录</button>
</form>
```

### 常见 input 类型

| 类型 | 用法 |
|--- |--- |
| text | 单行文本 |
| password | 密码 |
| radio | 单选框 |
| checkbox | 复选框 |
| submit | 提交按钮 |
| reset | 重置按钮 |
| button | 普通按钮 |
| file | 文件上传 |
| hidden | 隐藏字段 |

示例：

```html
<input type="radio" name="gender" value="male"> 男
<input type="checkbox" name="hobby" value="music"> 音乐
<input type="file">
```

---

## 8. 分区与容器标签

| 标签 | 用法 |
|--- |--- |
| `<div>` | 块级容器 |
| `<span>` | 行内容器 |

示例：

```html
<div class="box">
  <span>文字内容</span>
</div>
```

---

## 9. 嵌入内容标签

| 标签 | 用法 |
|--- |--- |
| `<iframe>` | 嵌入另一个网页 |
| `<object>` | 嵌入外部对象 |
| `<embed>` | 嵌入外部资源 |
| `<param>` | 设置 object 参数 |

示例：

```html
<iframe src="https://example.com" width="600" height="400"></iframe>
```

---

## 10. 已废弃或不推荐标签

以下标签在现代 HTML 中不推荐使用，应使用 CSS 替代。

| 标签 | 原作用 | 替代方式 |
|--- |--- |--- |
| `<font>` | 设置字体 | CSS `font-family`、`color` |
| `<center>` | 居中 | CSS `text-align: center` |
| `<big>` | 大号文字 | CSS `font-size` |
| `<strike>` | 删除线 | `<s>` 或 CSS |
| `<frameset>` | 框架集 | `<iframe>` 或现代布局 |
| `<frame>` | 框架页面 | `<iframe>` |

---

# 五、CSS3 的内容

CSS，全称 Cascading Style Sheets，层叠样式表，用于控制网页外观。

---

## 1. CSS 引入方式

### 行内样式

```html
<p style="color: red;">红色文字</p>
```

### 内部样式

```html
<style>
  p {
    color: blue;
  }
</style>
```

### 外部样式

```html
<link rel="stylesheet" href="style.css">
```

---

## 2. CSS 基本语法

```css
选择器 {
  属性: 值;
}
```

示例：

```css
p {
  color: red;
  font-size: 16px;
}
```

---

## 3. 常用选择器

| 选择器 | 示例 | 用法 |
|--- |--- |--- |
| 元素选择器 | `p` | 选择所有 p 元素 |
| 类选择器 | `.box` | 选择 class 为 box 的元素 |
| ID 选择器 | `#title` | 选择 id 为 title 的元素 |
| 后代选择器 | `div p` | 选择 div 内的 p |
| 子代选择器 | `div > p` | 选择 div 的直接子元素 p |
| 并集选择器 | `h1, p` | 同时选择多个元素 |
| 伪类选择器 | `a:hover` | 鼠标悬停状态 |
| 伪元素选择器 | `p::first-line` | 选择首行 |

---

## 4. 常用 CSS 属性

### 文本与字体

| 属性 | 用法 |
|--- |--- |
| color | 文字颜色 |
| font-size | 字体大小 |
| font-family | 字体 |
| font-weight | 字体粗细 |
| font-style | 字体样式 |
| text-align | 文本对齐 |
| text-decoration | 文本装饰 |
| line-height | 行高 |
| letter-spacing | 字符间距 |

```css
p {
  color: #333;
  font-size: 16px;
  line-height: 1.8;
}
```

### 背景

| 属性 | 用法 |
|--- |--- |
| background-color | 背景颜色 |
| background-image | 背景图片 |
| background-size | 背景尺寸 |
| background-position | 背景位置 |
| background-repeat | 背景重复方式 |

```css
body {
  background-color: #f5f5f5;
}
```

### 盒模型

| 属性 | 用法 |
|--- |--- |
| width | 宽度 |
| height | 高度 |
| margin | 外边距 |
| padding | 内边距 |
| border | 边框 |
| box-sizing | 盒模型计算方式 |

```css
.box {
  width: 200px;
  padding: 20px;
  border: 1px solid #ccc;
  margin: 10px;
  box-sizing: border-box;
}
```

### 显示与定位

| 属性 | 用法 |
|--- |--- |
| display | 显示方式 |
| position | 定位方式 |
| top/right/bottom/left | 定位偏移 |
| z-index | 层级 |
| overflow | 溢出处理 |
| visibility | 可见性 |

```css
.box {
  position: relative;
  top: 10px;
}
```

---

## 5. CSS3 新增常用特性

### 圆角

```css
.box {
  border-radius: 10px;
}
```

### 阴影

```css
.box {
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.2);
}

.text {
  text-shadow: 1px 1px 2px #999;
}
```

### 渐变

```css
.box {
  background: linear-gradient(to right, red, blue);
}
```

### 透明度

```css
.box {
  opacity: 0.8;
}
```

### 变形

```css
.box {
  transform: rotate(10deg) scale(1.2);
}
```

### 过渡

```css
.box {
  transition: all 0.3s ease;
}

.box:hover {
  background-color: red;
}
```

### 动画

```css
@keyframes move {
  from {
    left: 0;
  }
  to {
    left: 100px;
  }
}

.box {
  position: relative;
  animation: move 2s infinite;
}
```

---

## 6. Flex 弹性布局

Flex 用于一维布局，适合横向或纵向排列。

```css
.container {
  display: flex;
  justify-content: center;
  align-items: center;
}
```

常用属性：

| 属性 | 用法 |
|--- |--- |
| display: flex | 开启弹性布局 |
| flex-direction | 主轴方向 |
| justify-content | 主轴对齐 |
| align-items | 交叉轴对齐 |
| flex-wrap | 是否换行 |
| gap | 元素间距 |
| flex | 子元素弹性比例 |

---

## 7. Grid 网格布局

Grid 用于二维布局。

```css
.container {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 20px;
}
```

常用属性：

| 属性 | 用法 |
|--- |--- |
| display: grid | 开启网格布局 |
| grid-template-columns | 定义列 |
| grid-template-rows | 定义行 |
| gap | 网格间距 |
| grid-column | 设置元素占据列 |
| grid-row | 设置元素占据行 |

---

## 8. 媒体查询

用于响应式布局。

```css
@media screen and (max-width: 768px) {
  .container {
    flex-direction: column;
  }
}
```

---

# 六、JavaScript 的内容

JavaScript 是网页脚本语言，用于实现动态交互。

---

## 1. JavaScript 引入方式

### 内部脚本

```html
<script>
  alert("Hello JavaScript");
</script>
```

### 外部脚本

```html
<script src="script.js"></script>
```

---

## 2. 基本语法

```javascript
let name = "Tom";
const age = 18;

console.log(name);
```

语句通常以分号结束，虽然可省略，但建议保留。

---

## 3. 变量声明

| 关键字 | 用法 |
|--- |--- |
| var | 旧式变量声明，函数作用域 |
| let | 块级作用域变量 |
| const | 块级作用域常量 |

```javascript
let count = 10;
const PI = 3.14;
```

---

## 4. 数据类型

| 类型 | 示例 |
|--- |--- |
| Number | `123`、`3.14` |
| String | `"hello"` |
| Boolean | `true`、`false` |
| Undefined | 未赋值 |
| Null | 空值 |
| Object | 对象 |
| Array | 数组 |
| Function | 函数 |
| Symbol | 唯一值 |
| BigInt | 大整数 |

```javascript
let name = "Alice";
let age = 20;
let isStudent = true;
let list = [1, 2, 3];
let user = {
  name: "Alice",
  age: 20
};
```

---

## 5. 运算符

| 类型 | 示例 |
|--- |--- |
| 算术运算符 | `+`、`-`、`*`、`/`、`%` |
| 赋值运算符 | `=`、`+=`、`-=` |
| 比较运算符 | `>`、`<`、`>=`、`<=`、`==`、`===` |
| 逻辑运算符 | `&&`、`||`、`!` |
| 三元运算符 | `条件 ? A : B` |

```javascript
let result = age >= 18 ? "成年" : "未成年";
```

---

## 6. 条件语句

```javascript
if (age >= 18) {
  console.log("成年");
} else {
  console.log("未成年");
}
```

```javascript
switch (day) {
  case 1:
    console.log("星期一");
    break;
  default:
    console.log("其他");
}
```

---

## 7. 循环语句

```javascript
for (let i = 0; i < 5; i++) {
  console.log(i);
}
```

```javascript
let i = 0;
while (i < 5) {
  console.log(i);
  i++;
}
```

```javascript
for (let item of ["a", "b", "c"]) {
  console.log(item);
}
```

---

## 8. 函数

### 普通函数

```javascript
function add(a, b) {
  return a + b;
}
```

### 函数表达式

```javascript
const add = function(a, b) {
  return a + b;
};
```

### 箭头函数

```javascript
const add = (a, b) => a + b;
```

---

## 9. 数组

```javascript
let arr = [1, 2, 3];

arr.push(4);
arr.pop();

console.log(arr.length);
```

常用方法：

| 方法 | 用法 |
|--- |--- |
| push | 末尾添加 |
| pop | 删除末尾 |
| shift | 删除开头 |
| unshift | 开头添加 |
| splice | 添加或删除元素 |
| slice | 截取数组 |
| forEach | 遍历数组 |
| map | 映射新数组 |
| filter | 筛选数组 |
| reduce | 累计计算 |
| find | 查找元素 |
| includes | 判断是否包含 |

---

## 10. 对象

```javascript
let user = {
  name: "Tom",
  age: 18,
  sayHello: function() {
    console.log("Hello");
  }
};

console.log(user.name);
user.sayHello();
```

---

## 11. DOM 操作

DOM 是文档对象模型，JavaScript 可以操作网页元素。

### 获取元素

```javascript
let title = document.getElementById("title");
let items = document.querySelectorAll(".item");
```

### 修改内容

```javascript
title.innerText = "新标题";
title.innerHTML = "<span>新内容</span>";
```

### 修改样式

```javascript
title.style.color = "red";
title.classList.add("active");
```

### 创建元素

```javascript
let p = document.createElement("p");
p.innerText = "新段落";
document.body.appendChild(p);
```

---

## 12. 事件处理

```html
<button id="btn">点击我</button>
```

```javascript
let btn = document.getElementById("btn");

btn.addEventListener("click", function() {
  alert("按钮被点击了");
});
```

常见事件：

| 事件 | 用法 |
|--- |--- |
| click | 点击 |
| dblclick | 双击 |
| mouseover | 鼠标移入 |
| mouseout | 鼠标移出 |
| keydown | 键盘按下 |
| keyup | 键盘松开 |
| submit | 表单提交 |
| change | 内容改变 |
| input | 输入变化 |
| load | 页面加载完成 |

---

## 13. 定时器

```javascript
setTimeout(function() {
  console.log("3 秒后执行");
}, 3000);

setInterval(function() {
  console.log("每秒执行一次");
}, 1000);
```

---

## 14. JSON

JSON 是常用的数据交换格式。

```javascript
let user = {
  name: "Tom",
  age: 18
};

let json = JSON.stringify(user);
let obj = JSON.parse(json);
```

---

## 15. 异步请求 Fetch

```javascript
fetch("https://api.example.com/users")
  .then(response => response.json())
  .then(data => {
    console.log(data);
  })
  .catch(error => {
    console.error(error);
  });
```

使用 async/await：

```javascript
async function getData() {
  try {
    let response = await fetch("https://api.example.com/users");
    let data = await response.json();
    console.log(data);
  } catch (error) {
    console.error(error);
  }
}
```

---

# 七、综合示例

## HTML

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <title>前端示例</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <header>
    <h1 id="title">我的网页</h1>
  </header>

  <main>
    <p>这是一个 HTML、CSS、JavaScript 综合示例。</p>
    <button id="btn">点击切换颜色</button>
  </main>

  <script src="script.js"></script>
</body>
</html>
```

## CSS

```css
body {
  font-family: Arial, sans-serif;
  text-align: center;
  background: linear-gradient(to right, #e0f7fa, #ffffff);
}

h1 {
  color: #333;
  transition: color 0.3s;
}

.active {
  color: red;
}

button {
  padding: 10px 20px;
  border: none;
  border-radius: 8px;
  background-color: #2196f3;
  color: white;
  cursor: pointer;
}

button:hover {
  background-color: #1976d2;
}
```

## JavaScript

```javascript
const btn = document.getElementById("btn");
const title = document.getElementById("title");

btn.addEventListener("click", function() {
  title.classList.toggle("active");
});
```

---

# 八、学习建议

- 先学 HTML：理解网页结构和常用标签。
- 再学 CSS：掌握样式、盒模型、Flex、Grid、响应式布局。
- 最后学 JavaScript：掌握变量、函数、DOM、事件和异步请求。
- 多写小案例，例如个人主页、登录表单、图片列表、待办事项应用。
- HTML 负责内容，CSS 负责外观，JavaScript 负责交互。