# 渲染树构建、布局及绘制

在前面介绍构建对象模型的章节中，我们根据 HTML 和 CSS 输入构建了 DOM 树和 CSSOM 树。 不过，它们都是独立的对象，分别网罗文档不同方面的信息：一个描述内容，另一个则是描述需要对文档应用的样式规则。我们该如何将两者合并，让浏览器在屏幕上渲染像素呢？

- DOM 树与 CSSOM 树合并后形成渲染树。
- 渲染树只包含渲染网页所需的节点。
- 布局计算每个对象的精确位置和大小。
- 最后一步是绘制，使用最终渲染树将像素渲染到屏幕上。

第一步是让浏览器将 DOM 和 CSSOM 合并成一个“渲染树”，网罗网页上所有可见的 DOM 内容，以及每个节点的所有 CSSOM 样式信息。

![renderTree](/imgs/render-tree-construction.png)

为构建渲染树，浏览器大体上完成了下列工作：

1. 从 DOM 树的根节点开始遍历每个可见节点。

   - 某些节点不可见（例如脚本标记、元标记等），因为它们不会体现在渲染输出中，所以会被忽略。
   - 某些节点通过 CSS 隐藏，因此在渲染树中也会被忽略，例如，上例中的 span 节点---不会出现在渲染树中，---因为有一个显式规则在该节点上设置了“display: none”属性。

2. 对于每个可见节点，为其找到适配的 CSSOM 规则并应用它们。

3. 发射可见节点，连同其内容和计算的样式。

最终输出的渲染同时包含了屏幕上的所有可见内容及其样式信息。**有了渲染树，我们就可以进入“布局”阶段。**

到目前为止，我们计算了哪些节点应该是可见的以及它们的计算样式，但我们尚未计算它们在设备视口内的确切位置和大小---这就是“`布局`”阶段，也称为`自动重排`。

为弄清每个对象在网页上的确切大小和位置，浏览器从渲染树的根节点开始进行遍历。

下面简要概述了浏览器完成的步骤：

1. 处理 HTML 标记并构建 DOM 树。
2. 处理 CSS 标记并构建 CSSOM 树。
3. 将 DOM 与 CSSOM 合并成一个渲染树。
4. 根据渲染树来布局，以计算每个节点的几何信息。
5. 将各个节点绘制到屏幕上。

> 我们的演示网页看起来可能很简单，实际上却需要完成相当多的工作。如果 DOM 或 CSSOM 被修改，您只能再执行一遍以上所有步骤，以确定哪些像素需要在屏幕上进行重新渲染。

**优化关键渲染路径就是指最大限度缩短执行上述第 1 步至第 5 步耗费的总时间。** 这样一来，就能尽快将内容渲染到屏幕上，此外还能缩短首次渲染后屏幕刷新的时间，即为交互式内容实现更高的刷新率。

## 重绘

当元素的外观或外观可见性（visibility）发生变化时会触发重绘

## 回流

render 树中的部分或全部因为元素的规模尺寸、布局、隐藏等改变，需要重新计算 render 树。

**注意：回流必将引起重绘，而重绘不一定会引起回流。**

当页面布局和几何属性改变时就需要回流。下述情况会发生浏览器回流：

1. 添加或者删除可见的 DOM 元素；
2. 元素位置改变；
3. 元素尺寸改变——边距、填充、 边框、 宽度和高度
4. 内容改变——比如文本改变或者图片大小改变而引起的计算值宽度和高度改变；
5. 页面渲染初始化；
6. 浏览器窗口尺寸改变——resize 事件发生时；

## 优化（减少回流、重绘）

**浏览器本身的优化策略：**浏览器会维护 1 个队列，把所有会引起回流、重绘的操作放入这个队列，等队列中的操作到了一定的数量或者到了一定的时间间隔，浏览器就会 flush 队列，进行一个批处理。这样就会让多次的回流、重绘变成一次回流重绘。但有时候我们写的一些代码可能会强制浏览器提前 flush 队列，这样浏览器的优化可能就起不到作用了。当你请求向浏览器请求一些 style 信息的时候，就会让浏览器 flush 队列。

减少对 render tree 的操作（合并多次多 DOM 和样式的修改），并减少对一些 style 信息的请求，尽量利用好浏览器的优化策略

方法:

1.  将多次改变样式属性的操作合并成一次操作。

2.  将需要多次重排的元素，position 属性设为 absolute 或 fixed，这样此元素就脱离了文档流，它的变化不会影响到其他元素。例如有动画效果的元素就最好设置为绝对定位。

3)  在内存中多次操作节点，完成后再添加到文档中去。例如要异步获取表格数据，渲染到页面。可以先取得数据后在内存中构建整个表格的 html 片段，再一次性添加到文档中去，而不是循环添加每一行。

4)  由于 display 属性为 none 的元素不在渲染树中，对隐藏的元素操作不会引发其他元素的重排。如果要对一个元素进行复杂的操作时，可以先隐藏它，操作完成后再显示。这样只在隐藏和显示时触发 2 次重排。

5.  在需要经常取那些引起浏览器重排的属性值时，要缓存到变量。