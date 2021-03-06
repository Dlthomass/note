## 《高性能JavaScript》读书笔记

### 第一章  加载和执行

#### 脚本位置

由于脚本阻塞的存在，建议将所有`<script>`标签尽可能放到 `<body>` 标签的底部，以减少对整个页面下载的影响。

#### 组织脚本

减少脚本的HTTP请求数量，你可以将脚本合并。文件合并的工作可以通过离线的打包工具或者类似Yahho!combo handler的实时在线服务来实现。

#### 无阻塞的脚本

在页面加载完成后再加载JavaScript代码，也就是在window对象的load事件触发后再下载脚本。

#### 延迟的脚本

将无需修改DOM的脚本加上async属性，用于异步加载脚本。

#### 动态脚本元素

你可以使用DOM对象来动态的加载一个脚本：

```js
var script = document.createElement("script");
script.src = "file.js";
```

使用动态脚本下载文件时，返回的代码通常会立即执行。如果你要动态加载多个脚本文件，一定要考虑清楚文件的加载顺序。并不是所有的浏览器都会保证以你添加脚本的顺序来执行代码，那取决于服务器的返回时间。

#### XMLHttpRequest脚本注入

也就是通过Ajax来加载脚本文件。这种方法的优点在于你可以下载JavaScript代码但不立即执行。

#### 推荐的无阻塞模式

向页面中添加大量JavaScript的推荐做法只需两步：

1. 添加动态加载所需代码。
2. 加载初始化页面所需代码。

```html
<script src="loader.js"></script>
<script>
    loadScript("the-rest.js",function(){
        Application.init();
    });
</script>
```



### 第二章  数据存取

#### 管理作用域

要理解速度和作用域的关系，首先要正确地理解作用域的工作原理。

#### 作用域链和标识符解析的性能

一个标识符所在的位置越深，它的读写速度就越慢。因此，函数中读写局部变量总是最快的，而读写全局变量总是最慢的。全局变量总是存在于执行作用域链的最末端。

对于有过优化的浏览器，跨作用域访问标识符时并没有类似的性能损失。

在没有优化的浏览器中，建议尽可能使用局部变量。

#### 改变作用域链

不要使用 `with` 来改变作用域链。

#### 动态作用域

动态作用域只存在于代码执行过程中，因此无法通过静态分析检查出来。

不推荐使用动态作用域。

#### 闭包，作用域和内存

闭包是对作用域的引用，因此在使用闭包时，可能会造成性能问题。在脚本编程中，最好小心的使用闭包，它同时关系到内存和执行速度。

#### 原型链与嵌套成员

属性或方法在原型链中的位置越深，访问它的速度就越慢。

尽量避免在同一个函数中多次查找同一个对象，除非它的值改变了。

### 第三章 DOM编程

用脚本进行DOM操作的代价很昂贵，它是富应用中常见的性能瓶颈。

#### DOM访问与修改

访问DOM的次数越多，代码的运行速度越慢。因此，应该减少访问DOM的次数，把运算尽量留在ECMAScript这一端进行。

#### HTML集合

HTML集合一直与文档保持着连接，每次你需要最新的信息时，都会重复执行查询的过程。这正是低效之源。

你可以将HTML集合存储在局部变量中，这样可以减少引擎对HTML的读取。

#### 遍历DOM

使用浏览器提供的API来过滤文本节点会更高效。

如果要处理大量组合查询，使用 `querySelectorAll()` 更有效率。

#### 重绘与重排

当元素的几何属性改变时，会引起浏览器的重排与重绘，重排和重绘都是代价昂贵的操作，它们会导致WEB应用程序的UI反应迟钝。所以，应该尽可能减少这类过程的发生。

重排何时发生：

- 添加或删除可见的DOM元素。
- 元素位置改变。
- 元素尺寸改变。
- 内容改变。
- 页面渲染器初始化
- 浏览器窗口尺寸改变

为了减少重绘及重排的发生次数，你可以：

- 合并多次对DOM和样式的修改
- 批量修改DOM

#### 事件委托

使用事件委托来减少事件处理器的数量，可以显著提高性能。

### 第四章 算法和流程控制

#### 循环

`forEach()` 等函数迭代方法尽管被支持也非常便利，但它仍然比基于循环的迭代要慢一些。如果你的程序有性能要求，基于函数的迭代不是很合适的选择。

#### 条件语句

优化 `if-else`  ：

1. 最简单的方法是将最可能出现的条件放在首位，这样可以避免很多不必要的判断。
2. 还可以将多个判断分组，减轻条件判断的负担。

查找表：当大量离散值需要测试时，可以使用数组和普通对象来构建查找表，这样可以提高数据访问速度。

```js
var results = [result1,result2,result3,result4,result5,result6,······];
return results[value];
```

当单个键和单个值之间存在逻辑映射时，查找表的优势便能体现出来。

#### 递归

浏览器拥有固定数量的调用栈限制，当你使用了太多的递归超过最大调用栈容量时，浏览器将会报错。

#### 迭代

任何使用递归实现的算法同样可以使用迭代来实现。使用优化后的循环替代长时间运行的递归函数可以提升性能，因为运行一个循环比反复调用一个函数的开销要少得多。

### 第五章 字符串和正则表达式

提高正则表达式效率的方法：

1. 正则表达式以简单，必须的字元开始。起始标记最好为一个锚，特定字符串，字符类，和单词边界。
2. 使用量词模式，使它们后面的字元互斥。
3. 减少分支数量，减少分支范围。
4. 使用非捕获组。
5. 只捕获感兴趣的文本以减少后处理。
6. 暴露必需的字元。
7. 使用适合的量词。
8. 把正则表达式赋值给变量并重用它们。
9. 将复杂的正则表达式拆分

### 第六章 快速响应的UI界面

在你的web应用中限制高频率重复定时器的数量。

Web Workers是浏览器支持的特性,它允许你在UI线程外部执行JavaScript代码，从而避免锁定UI。

你可以用Web Worker来：

- 编/解码大字符串
- 复杂数学运算
- 大数组排序

### 第七章 AJAX

通过缓存数据来提高减少请求数量，选择合适的数据介质来提高你的数据解析速度。

你应该知道如何使用成熟的Ajax类库，以及何时编写自己的底层Ajax代码。

### 第八章 编程实践

#### 使用Obejct/Array直接量

直接量比使用对象更快：

```js
var array1 = [];
// 上面的代码比下面的运行得更快
var array2 = new Array();
```



#### 位操作

JavaScript的位运算的速度其实是非常快的，你可以尽管用它来做一些必要的运算。

#### 原生方法

无论你的JavaScript代码如何优化，都永远不会比JavaScript引擎提供的原生方法更快。当有原生方法可用时，尽量使用它们。特别是数学运算和DOM操作。

### 第九章 构建并部署高性能JavaScript应用

#### 合并多个JavaScript文件

将多个JavaScript文件合并为一个文件以减少HTTP请求数。

#### 预处理JavaScript文件

预处理你的JavaScript源文件并不会让应用变得更快，但它允许你做些其他事情，例如有条件地插入测试代码，来衡量你的应用程序的性能。

#### JavaScrip压缩

JavaScript压缩是指把JavaScript文件中所有与运行无关的部分进行剥离的过程。剥离的内容包括注释和不必要的空白字符。

#### JavaScript的HTTP压缩

当Web浏览器请求一个资源时，它通常会发送一个HTTP头来告诉服务器它支持哪种编码类型，这个信息主要用来压缩文件以获得更快的下载。

`Accept-Eecoding`通常包括`gzip`，`compress`,`deflate`,`identiry`。

#### 使用内容分发网络（CDN）

CDN通过向地理位置最近的用户传输内容，极大的减少网络延时。

你可以自建CDN或者使用第三方CDN服务。

### 第十章 工具

性能分析工具：[YUI Profiler](https://yuilibrary.com/)

网站性能检测工具：[Page Speed](https://www.pingdom.com/)

HTTP代理调试工具：[Fibbler](https://www.telerik.com/fiddler)

工具的作用：

- 使用网络分析工具找出加载脚本和页面中其他资源的瓶颈，这会帮助你决定哪些脚本需要延迟加载，或者需要进一步分析。

- 尽管传统的经验告诉我们要减少HTTP请求数，但把脚本尽可能延迟加载可以加快页面的渲染速度，给用户带来更好的体验。

- 使用性能分析工具找出脚本运行过程中速度慢的地方，检查每个函数所消耗的时间，以及函数被调用的次数，通过调用栈自身提供的一些线索来找出需要集中精力优化的地方。

  





