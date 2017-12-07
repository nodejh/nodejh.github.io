+++
title = "使用 Flexbox 使浮动元素垂直居中"
description = "How To Vertically Middle Align Floated Elements With Flexbox"
date = 2017-07-28T20:53:14+08:00
tags = ["CSS"]
categories = ["CSS"]
draft = false
+++

垂直居中一直是一个很麻烦的问题，但基于 FlexBox 的垂直居中就非常简单了。

 <!--more--> 

考虑下面的场景：

- 你正在使用网格布局的框架，比如 [Bootstrap](http://getbootstrap.com/)、[Foundation](http://foundation.zurb.com/grid.html)、[ Skeleton](http://getskeleton.com/)、[Susy](http://oddbird.net/susy/) 等。
- 你有两个包含动态内容的列（每列都是一个盒模型），你并不知道每列的具体尺寸，也不知道哪个更高。
- 你需要这两列能够垂直居中。


我们希望得到的布局就像下面这样：

![http://oh1ywjyqf.bkt.clouddn.com/How-to-Vertically-Middle-Align-Floated-Elements-with-Flexbox-01.png](http://oh1ywjyqf.bkt.clouddn.com/How-to-Vertically-Middle-Align-Floated-Elements-with-Flexbox-01.png)

但默认情况下，这两列将会顶部对齐：

![http://oh1ywjyqf.bkt.clouddn.com/How-to-Vertically-Middle-Align-Floated-Elements-with-Flexbox-02.png](http://oh1ywjyqf.bkt.clouddn.com/How-to-Vertically-Middle-Align-Floated-Elements-with-Flexbox-02.png)

所以问题来了，在不改变浮动布局的前提下，我们应该如何是元素垂直居中对齐？到目前为止，这个简单的问题都非常难以解决。

## 是否可以使用 `vertical-align:middle`

很不幸，不能，因为一些不同的原因。

首先，在 [MDN->CSS](https://developer.mozilla.org/en-US/docs/Web/CSS/vertical-align) 中有关于 `vertical-align` 的描述 `The vertical-align CSS property specifies the vertical alignment of an inline or table-cell box.`。我们的元素不是 `inline`、`inline-block` 或 `table-cells`，所以在不改变 `display` 的情况下 `vertical-align:middle` 并不会生效。

其次，网格布局的框架使用了 `float:right` 来对我们的两列元素进行定位，在 [W3C->CSS->9.5.1 Positioning the float: the 'float' property](https://www.w3.org/TR/CSS22/visuren.html#float-position) 第八条有描述 `A floating box must be placed as high as possible.`。这意味着浮动元素总会被固定在顶部。

第一个问题我们可以通过将 `display` 改为 `inline-block` 或者 `table-cell` 解决，但没有 CSS 技术可以解决第二个问题。我们需要移除浮动规则，但这会破坏基于网格布局的框架的基础。

## 使用 Flexbox 解决问题

像往常一样，Flexbox 对我们的问题有一个简单的解决方案。只需要简单的两步就行了：

1. 为元素添加 `display: flex` 规则。
2. 为对元素添加 `‘align-items: center` 规则。

这样就可以了！下面是一个简单的 HTML 和 CSS 例子：

```html
<div class="container">
 <div class="column-1">[Dynamic content]</div>
 <div class="column-2">[Dynamic content]</div>
</div>
```

```css
.container {
  display: flex;
  align-items: center;
}

.column-1,
.column-2 {
  float: left;
  width: 50%;
}
```

<p data-height="265" data-theme-id="light" data-slug-hash="RZWYZX" data-default-tab="html,result" data-user="nodejh" data-embed-version="2" data-pen-title="RZWYZX" class="codepen">See the Pen <a href="https://codepen.io/nodejh/pen/RZWYZX/">RZWYZX</a> by Hang Jiang (<a href="https://codepen.io/nodejh">@nodejh</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>


你可以从下面的动画中看到，两列元素将会根据内容的变化而始终保持垂直居中对齐：

![http://oh1ywjyqf.bkt.clouddn.com/How-to-Vertically-Middle-Align-Floated-Elements-with-Flexbox-03.gif](http://oh1ywjyqf.bkt.clouddn.com/How-to-Vertically-Middle-Align-Floated-Elements-with-Flexbox-03.gif)

这种解决方法最好的一点是通过添加两个规则，在没有对两列元素的原本样式做任何修改的前提下就实现了垂直居中对齐。现在大部分浏览器都支持 flex，老的浏览器会忽略该规则，元素将保持顶部对齐。



关于 flexbox 的浏览器兼容性可以在 [Can I Use Flexbox](http://caniuse.com/#search=flex) 查看得到：

![http://oh1ywjyqf.bkt.clouddn.com/How-to-Vertically-Middle-Align-Floated-Elements-with-Flexbox-04.png](http://oh1ywjyqf.bkt.clouddn.com/How-to-Vertically-Middle-Align-Floated-Elements-with-Flexbox-04.png)

---

参考

- [How to Vertically Middle-Align Floated Elements with Flexbox](https://spin.atomicobject.com/2016/06/18/vertically-center-floated-elements-flexbox/)
