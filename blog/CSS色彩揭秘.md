

每每提及色彩，我总会想起苏轼的`一年好景君须记，正是橙黄橘绿时`，白居易的`日出江花红胜火，春来江水绿如蓝`，朱熹的`等闲识得东风面，万紫千红总是春`。


也许世界本无色彩。

古人曾用诗词赋予了这个世界色彩，而如今是物理的光学让我们认识到了色彩。那么本文将带你揭秘CSS中的色彩。

**`先赞在看，养成习惯！`**

## CSS中的color
> CSS1只支持16个基本颜色关键字
>
> CSS2在CSS1的基础上添加了橙色`orange`
>
> CSS3增加了147个关键字
>
> CSS4只增加了一个关键字 `rebeccapurple`

链接：[CSS3中新增147个颜色关键字](https://www.w3school.com.cn/cssref/css_colornames.asp)

我知道你会说这个CSS4的新增的一个关键字rebeccapurple是什么鬼？还嫌我英文单词记的不够多吗？搞个这么难记的东西出来？

但实际上，CSS的作者Eric Meyer 的女儿丽贝卡死于脑癌，享年六岁。这是她最喜欢的颜色。

`丽贝卡紫`

链接：[为了纪念 Eric Meyer 的女儿，色值 #663399 在 CSS4 中被定义为 「rebeccapurple」](https://lists.w3.org/Archives/Public/www-style/2014Jun/0312.html)

**冰冷严谨的代码中充满了如此温情之举，你值得记忆。**


## RGB
三原色理论告诉我们，`红、绿、蓝`三种颜色的光可以构成所有的颜色。

### 为什么是这三种颜色呢？
因为人类的视神经对这三种颜色比较敏感。

现代计算机中用0-255的数字来表示每一种颜色。在RGB颜色表示方法中，三色数值最大的就是白色，三色数值为0则表示黑色。理解起来也比较符合人类的认知，红绿蓝三种颜色的光混合起来就是白光，没有光就是黑色。

除此之外，我们需要注意两点：

> 1.rgb数值格式只能是整数，不能是小数。
>
>2.还可以用百分比来进行表示，百分比范围0%-100%

### RGB表示法

```css
#div {background-color:rgb(255,0,0);} /* 红 */
#div {background-color:rgb(0,255,0);} /* 绿 */
#div {background-color:rgb(0,0,255);} /* 蓝 */
```

### 16进制表示法 HEX
十六进制颜色实际上和rgb颜色是近亲，都归属于rgb颜色，只是进制有差异。
```css
#div {background-color:#FF0000;} /* 红 */
#div {background-color:#00FF00;} /* 绿 */
#div {background-color:#0000FF;} /* 蓝 */
```

## RGBA
`RGBA`是在`RGB`的扩展，增加了一个`Alpha`的色彩通道，规定了透明度(取值范围0～1)，0表示全透明。

### RGBA代码
```css
div {background:rgba(255,0,0,0.5);}
```

### RGBA与opacity的区别？
`opacity `是属性，`rgba()`是函数，计算之后是个属性值。

`rgba`一般修改的是背景色或者文本的颜色，内容不会继承透明度。

`opacity`作用于元素和元素的内容，内容会继承透明度。

## HSL
人类对颜色的感知是远远大于红、绿、蓝的，因此HSL颜色模型被设计出来。

`HSL`分别代表色相，纯度以及明度，也有色调、饱和度、亮度的说法。
>`h表示hue`，取值0-360。大致按照红橙黄绿青蓝紫变化  0 (或 360) 为红色, 120 为绿色, 240 为蓝色
>
>`s表示saturation`，用0-100%表示。值越大，饱和度越高，颜色越亮 0%为灰色，100%为全色
>
>`l表示lightness`，用0-100%表示。值越高，颜色越亮，100%是白色，50%是正常亮度，0%就是黑色

## HSLA
如同`RGBA`模式是`RGB`的扩展模式，`HSLA`模式是`HSL`的扩展模式，在`HSL`的基础上增加一个透明通道`Alpha`来设置透明度。

## CMYK
回忆起儿时美术老师(我们区的美术协会主席)曾经教过，颜料的三原色是“红黄蓝”。

而颜料能够显示颜色的原理是它吸收了所有别的颜色的光，只返回一种色彩。这个世界就是这么魔幻，你看到的不一定是你看到的。

颜料三原色是红、绿、蓝的补色，也可以叫它们“品红、黄、青”。这种颜色表示法就叫做`CMYK`。

熟悉UI设计的同学们一定知道，在印刷业，就是采用颜料三原色来配置油墨。不过除了品红、黄、青外，因为黑色颜料常用并且价格低廉，所以被单独指定。

## transparent
`transparent`用来指定全透明色彩，我们可以利用这个属性画出三角形。
```css
.triangle {
    width: 0;
    height: 0;
    border-left: 50px solid transparent;
    border-right: 50px solid transparent;
    border-bottom: 50px solid #ff0000;
}
```
```html
<div class="triangle"></div>
```
`transparent`也比较魔幻，`background-color:transparent`包括IE6都支持，`boder-color:transparent`从IE7开始支持，但是`color:transparent`却从IE9浏览器开始支持。

## currentColor
`currentColor`的意思是使用当前color的计算值，也从IE9+才支持。

CSS中很多的属性值默认就是`currentColor`的表现。
```css
div {
  color: red;
  border: 1px solid;
}
```

一般情况下无须画蛇添足的添加currentColor。

```css
div {
  color: red;
  border: 1px solid currentColor;
}
```

## 兼容性查询
授人以鱼不如授人以渔。

大家可以去[这个网站](http://caniuse.com/)查找你所要用的属性在浏览器中的兼容性。



