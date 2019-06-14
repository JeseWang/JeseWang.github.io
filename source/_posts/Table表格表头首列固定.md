---
title: Table表格表头首列固定
date: 2017-3-2 22:46:09
categories: 大前端
tags: [javascript,table]
---
最近在做手机端的web时有一个需求，需要实现table表格的首行和首列固定，虽然这是个很简单也很经典的一个问题，但关键是要保证在手机端浏览拥有大量数据的表格，而且表头表尾和首列在固定的前提下还要跟随滑动，我第一个想法就是网上找一个处理类似问题的第三方插件，但想了想，仅仅为了这一个页面而引用插件甚至一个库，本来展示的数据量就比较大，再加上插件，有点儿得不偿失，算了，看能不能用最简单的方法把问题给解决了，于是就有了下面的内容……
<!--more-->
首先展示最终的效果：
![table](/something/table.gif)

1. 首先，为了表格的展现方便，引用了bootstrap的table.less样式，自己写也可以，不用用gulp压缩一下也就不到8kb，大小还能接受
2. 整个表格分为两部分，固定的首列和展示数据的右侧，而左右内容有分为三列，固定的表头表尾，和中间可以滑动的数据部分
3. 首行和首列的跟随滑动是简单的通过js监听各部分的scoll实现的

```
<div class="content">
	<div class="left">
		<div class="table-responsive t-header">
			<table class="table table-bordered">...</table>
		</div>
		<div class="table-responsive t-body" id="t_l_content">
			<table class="table table-bordered">...</table>
		</div>
		<div class="table-responsive t-footer">
			<table class="table table-bordered">...</table>
		</div>
	</div>
	<div class="right">
		<div class="table-responsive t-header" id="t_r_title">
			<table class="table table-bordered">...</table>
		</div>
		<div class="table-responsive t-body" id="t_r_content" onscroll="scrollCtrl()">
			<table class="table table-bordered">...</table>
		</div>
		<div class="table-responsive t-footer" id="t_r_footer">
			<table class="table table-bordered">...</table>
		</div>
	</div>
</div>
```
```
function scrollCtrl(){
    var top = document.getElementById("t_r_content").scrollTop; 
    var left = document.getElementById("t_r_content").scrollLeft; 
    document.getElementById("t_l_content").scrollTop=top; 
    document.getElementById("t_r_title").scrollLeft=left; 
    document.getElementById("t_r_footer").scrollLeft=left; 
}
```
