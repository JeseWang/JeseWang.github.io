---
title: 重回前端——关于Class
date: 2019.06.03 23:34
categories: 大前端
tags: [前端]
---
在学习了C#之后，回头再看JavaScript时，第一时间就想到关于Class和伪类，当初在看《JavaScript高程》和《JavaScript语言精粹》时，在关于原型和继承的相关章节，不止一次的提到了伪类，并阐述了使用的伪类的优点和缺点。“伪类”形式可以给习惯了使用基于类的语言编程的后端程序员提供方便书写JavaScript的便利，但是隐藏了语言的本质，而作为一个从前端入门的程序员，对于类的概念始终模糊不清，即使当ES6推出了Class语法机制，也因为项目的关系和薄弱的基础不敢贸然使用，只是浅尝辄止于了解基本语法。
#### 关于伪类
伪类是仿照基于类的语言，通过构造器函数产生对象。在C#中，类是程序的基本单元，通过定义类，类的构造函数，类的方法等，通过父类和子类的继承关系，实现很多复杂的应用。而在JavaScript中，如果定义一个“类”呢？
```
// 定义一个函数
function MathHandle(x, y) {
  this.x = x
  this.y = y
}
// 通过函数的prototype添加该函数的原型方法add
MathHandle.prototype.add = function () {
  return this.x + this.y
}

var m = new MathHandle(1,2)
m.add() // 3
```
显而易见，通过定义一个“伪类”MathHandle，添加prototype的原型方法，通过new关键字构造一个实例。而在C#中如何表现呢？
```
// 定义类
public class Mathhandle
{
  private int x;
  private int y;
  // 定义类的构造函数
  public MathHandle(int x, int y)
  {
    this.x = x;
    this.y = y;
  }
  // 定义类的方法
  public int add()
  {
    return x + y;
  }
}

MathHandle m = new MathHandle(1, 2);
int sum = m.add() // 3
```
看了C#的语法再看JavaScript，会发现基于类的语言结构清晰，也更加严谨，所幸在ES6中推出了Class语法机制。

#### JavaScript中的Class
```
// 通过class关键字定义MathHandle类
class MathHandle {
  // 类中的构造函数，也叫构造器
  constructor(x, y) {
    this.x = x
    this.y = y
  }
  // 类中的构造方法
  add() {
    return this.x + this.y
  }
}

const m = new MathHandle(1, 2) //创建Class的实例，并调用默认构造函数
m.add() // 3
```
看明白了C#的语法再看es6中的class就会很轻松，可是看了很多文章，对于在JavaScript中使用class很多人持谨慎的态度，class只是JavaScript的语法糖而已，并没有改变语言的实质。
```
function MathHandle(x, y) {
  // ...
}
typeof MathHandle // 'function'
MathHandle.prototype.consturctor === MathHandle // true
m.__proto__ === MathHandle.prototype // true
```
在第一种方法中，MathHandle的typeof返回的是‘function'，这没有问题，查看他的原型链可以发现，他的隐式原型也等于函数本身，再来看看在class方法中是什么样子
```
class MathHandle {
  // ...
}
typeof MathHandle // 'function'
MathHandle.prototype.consturctor === MathHandle // true
m.__proto__ === MathHandle.prototype // true
```
看来，在es6的class中，class定义的所谓的“类”的类型，依然是function，而实例的方法，也是通过原型传递的。弄懂了es6中class的本质，会发现类和继承并没有什么神秘的地方。