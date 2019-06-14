---
title: 走向Vue2.0
date: 2017-04-3 20:59:22
categories: 大前端
tags: [Vue]
---
去年来到新公司，接触了我的第一个vue项目，从官方文档到各种demo，慢慢的也开始了我的vue之旅，在开发时的确各种得心应手，很快便融入了新项目的开发工作中，但随着2.0版本的逐渐占据主流，在工作之余也开始拥抱新的变化，由于在目前的项目用的都是很基础的一些特性，对于vue的理解还仅仅停留在可以用的基础上，很多更高级更深入的东西并没有过多的研究，所以正好趁着在学习新版本时更深入的学习一下vue，记录下我的vue踩坑日常吧！
<!--more-->
#### 关于生命周期
![image](https://cn.vuejs.org/images/lifecycle.png)
![image](http://img.blog.csdn.net/20170307143230692?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2V4eV9zcXVpcnJlbA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
说的再多还是图来的直接一些，关于和1.0的区别，这个图已经说的很清楚了，由于之前的1.0项目中，经常和后台的数据进行交互，所以请求和更新数据很多都是在ready这一步完成的，可是由于对Vue的整个生命周期没有什么概念，所以一直云里雾里，2.0的改变还是很直观的，可以对程序在哪一步发生了什么一目了然。

#### Vue的脚手架
在过去使用Vue1.0的工作中，由于使用Sea.js作为模块化，并没有引入Vue的概念，而且压缩打包工具用的也是fis3，所以将Vue作为全部的模块进行了引入，每个模块都是单独的`new Vue({})`，Vue的使用更多的是和Jquery一样，什么时候需要了什么时候引入，而自从发现了Vue-cli后，发现真的是太方便了，Vue真的是作为了最底层的框架去使用，可操作性大大提高。每个模块可以查分为更多的小模块，.Vue文件也大大增强了各模块的灵活性。

#### 组件通信
在Vue中，不同的组件之间通信还是很麻烦的，之前为了简化开发的难度，往往一些不太大的模块都没有进行拆分，直接导致一个文件的代码超过了几千行，在刚刚接手的时候的确有些力不从心啊。
对于父子组件，用props进行通信还是很方便的，例如：
```
...
<child :data="data"></chils/>
...
Vue.component('child',{
    template:'<div>{{data}}</sdiv>',
    props: {
        data: DATA.data
    },
    data(){
        return {
            data$: this.data
        }
    }
})

```
但是，props是单向数据流的，只能向子组件传递数据，而并不能逆向的回传，所以，如果子组件向父组件传递数据，可以利用自定义事件：
```
在vue中，可以使用
    $on(eventName) 监听事件
    $emit(eventName) 触发事件
按照官方的说法，父子组建通信的原则是'props down, events up'

下面援引一个官方的经典例子：
```
```
<div id="counter-event-example">
  <p>{{ total }}</p>
  <button-counter v-on:increment="incrementTotal"></button-counter>
  <button-counter v-on:increment="incrementTotal"></button-counter>
</div>
```
```
Vue.component('button-counter', {
  template: '<button v-on:click="increment">{{ counter }}</button>',
  data: function () {
    return {
      counter: 0
    }
  },
  methods: {
    increment: function () {
      this.counter += 1
      this.$emit('increment')
    }
  },
})
new Vue({
  el: '#counter-event-example',
  data: {
    total: 0
  },
  methods: {
    incrementTotal: function () {
      this.total += 1
    }
  }
})
```
现在，更好的方案其实是`Vuex状态管理器`，大量的组件间通信势必会造成代码的繁琐，尤其是两个子组件间想要进行数据传递时，而我也已经感受到了管理各种数据的痛处，虽然现在没有直接用Vuex，但是了解一下还是很有必要的。

#### Vuex

![image](https://vuex.vuejs.org/zh-cn/images/vuex.png)
> Vuex将组件的共享状态抽取出来，以一个全局单例模式进行管理，在这种模式下，组件树构成了一个巨大的视图，不管在树的哪个位置，任何组件都能获取状态或者触发行为。

Vuex还需要多理解官方的文档，多练习，相信完成几个demo后会有更深入的理解。