---
title: Flex布局详解
date: 2016-04-15 08:59:22
categories: 大前端
tags: [Flex,HTML]
---
由于在工作中经常用到页面布局，例如多元素之间的居中，单行排列，多行排列，之前一直用的是`display,float,position`等传统的盒模型布局方案，虽然也可以解决大部分的页面布局要求，但总归还是不太方便，例如：单行多元素分散对齐等，需要很多行css去实现，而且一不小心还容易出现很多小问题，而直到接触了**Flex布局**，仿佛一下子看到了黎明的曙光，简单的几行代码就可以完成很多复杂的布局，而且随着flex布局也越来越多的应用也逐渐成为主流，在此总结一下Flex布局的用法及一些需要注意的地方……
<!--more-->
（图片及部分引用来自*阮一峰老师博客*）
#### 一、指定容器
> Flex是Flexible Box的缩写，意为"弹性布局"，用来为盒状模型提供最大的灵活性。
	任何一个容器都可以指定为Flex布局。
	
如何指定flex？
```
.box{
	display: -webkit-flex;
	display: flex;
}
```
好了，现在.box的元素已经被指定为flex模型的容器，它的所有子元素都将被指定为fle布局，同时，子元素的float、position和vertical-align属性将失效。
#### 二、布局概念
**父容器：flex-container**
**字元素：flex-item**
>容器默认存在两根轴：水平的主轴（main axis）和垂直的交叉轴（cross axis）。主轴的开始位置（与边框的交叉点）叫做main start，结束位置叫做main end；交叉轴的开始位置叫做cross start，结束位置叫做cross end。
项目默认沿主轴排列。单个项目占据的主轴空间叫做main size，占据的交叉轴空间叫做cross size。

**一张很好的图片对该容器的布局进行说明**
![](http://www.ruanyifeng.com/blogimg/asset/2015/bg2015071004.png)
#### 三、 容器的属性
容器共有6个属性，分别是：
- **flex-direction**
- **flex-wrap**
- **flex-flow**
- **justify-content**
- **align-items**
- **align-content**

`flex-direction`属性共有4个值，决定主轴的方向（即项目的排列方向）。
```
.box{
	flex-direction: row | row-reverse | column | column-reverse;
}

row（默认值）：主轴为水平方向，起点在左端。
row-reverse：主轴为水平方向，起点在右端。
column：主轴为垂直方向，起点在上沿。
column-reverse：主轴为垂直方向，起点在下沿。
```
![Alt text](http://www.ruanyifeng.com/blogimg/asset/2015/bg2015071005.png)

`flex-wrap`属性共有3个值，它决定了当项目在一条轴线上（横或者竖）过多时如何处理（换行？如何换行？）
```
.box{
	flex-wrap: 
		nowrap（默认）：不换行。
		wrap : 换行，第一行在上方。
		wrap-reverse : 换行，第一行在下方。
}
```
`flex-flow`属性是`flex-direction`属性和`flex-wrap`属性的简写形式，默认值为`row nowrap`。
```
.box {
	flex-flow: <flex-direction> || <flex-wrap>;
}
```
`justify-content`属性定义了项目在主轴上的对齐方式，对齐方向均为从start 到 end 方向
```
.box {
	justify-content: 
		flex-start（默认值）：左对齐
		flex-end：右对齐
		center： 居中
		space-between：两端对齐，项目之间的间隔都相等。
		space-around：每个项目两侧的间隔相等。所以，项目之间的间隔比项目与边框的间隔大一倍。
}
```
`align-items`属性定义项目在交叉轴上如何对齐。
```
.box {
	align-items: 
		flex-start：交叉轴的起点对齐。
		flex-end：交叉轴的终点对齐。
		center：交叉轴的中点对齐。
		baseline: 项目的第一行文字的基线对齐。
		stretch（默认值）：如果项目未设置高度或设为auto，将占满整个容器的高度。
}
```
**看图说话：**
![Alt text](http://www.ruanyifeng.com/blogimg/asset/2015/bg2015071011.png)
`align-content`属性定义了多根轴线的对齐方式。如果项目只有一根轴线，该属性不起作用。
```
.box {
	align-content: 
		flex-start：与交叉轴的起点对齐。
		flex-end：与交叉轴的终点对齐。
		center：与交叉轴的中点对齐。
		space-between：与交叉轴两端对齐，轴线之间的间隔平均分布。
		space-around：每根轴线两侧的间隔都相等。所以，轴线之间的间隔比轴线与边框的间隔大一倍。
		stretch（默认值）：轴线占满整个交叉轴。
}
```
**以上均为父容器的属性，定义在父容器的样式上**

#### 四、项目的属性
项目（子元素）也有6个属性，非别是
- **order**
- **flex-grow**
- **flex-shrink**
- **flex-basis**
- **flex**
- **align-self**

**`order`**属性定义项目的排列顺序。数值越小，排列越靠前，默认为0。

**`flex-grow`**属性定义项目的放大比例，默认为0，即如果存在剩余空间，也不放大。
![Alt text](http://www.ruanyifeng.com/blogimg/asset/2015/bg2015071014.png)
如果所有项目的flex-grow属性都为1，则它们将等分剩余空间（如果有的话）。如果一个项目的flex-grow属性为2，其他项目都为1，则前者占据的剩余空间将比其他项多一倍。

**`flex-shrink`属性定义了项目的缩小比例，默认为1，即如果空间不足，该项目将缩小。
![Alt text](http://www.ruanyifeng.com/blogimg/asset/2015/bg2015071015.jpg)
如果所有项目的flex-shrink属性都为1，当空间不足时，都将等比例缩小。如果一个项目的flex-shrink属性为0，其他项目都为1，则空间不足时，前者不缩小。负值对该属性无效。

**`flex-basis`**属性定义了在分配多余空间之前，项目占据的主轴空间（main size）。浏览器根据这个属性，计算主轴是否有多余空间。它的默认值为`auto`，即项目的本来大小。它可以设为跟width或height属性一样的值（比如350px），则项目将占据固定空间。

**`flex`**属性是`flex-grow`, `flex-shrink` 和 `flex-basis`的简写，默认值为`0 1 auto`。后两个属性可选。

**`align-self`**属性允许单个项目有与其他项目不一样的对齐方式，可覆盖`align-items`属性。默认值为`auto`，表示继承父元素的`align-items`属性，如果没有父元素，则等同于`stretch`。
```
.item {
  align-self: auto | flex-start | flex-end | center | baseline | stretch;
}
```