---
title: 关于python之闭包
date: 2016-10-22 22:32:27
categories: 极客之道
tags: [python,JavaScirpt]
---
早就听说了python的强大，但直到亲自学习起来才发现这门语言的魅力，最近一直在学习**python**语言，准确的说，应该是python3，今天翻到了廖雪峰老师的**python3**教程，看到**返回函数**这一节，**python**也和**javascript**一样，可以将函数作为值进行传递，尤其是**闭包**的概念，正好今天借这个问题，来比较一下python和javascript的异同……
<!--more-->
在教程里有这么一个函数：
``` python
def count():
    fs = []
    for i in range(1, 4):
        def f():
             return i*i
        fs.append(f)
    return fs
    
f1, f2, f3 = count()
```
这段代码打印出来的结果是：
```
>>> f1()
9
>>> f2()
9
>>> f3()
9
```
因为函数***f( )***没有立即执行，当函数返回时，循环已经结束，引用的变量i已经为3，为了解决这个办法，可以再创建一个函数，用该函数的参数绑定循环变量当前的值，代码如下：
``` python
def count():
    def f(j):
        def g():
            return j*j
        return g
    fs = []
    for i in range(1, 4):
        fs.append(f(i))
    return fs
f1, f2, f3 = count()
>>> f1()
1
>>> f2()
4
>>> f3()
9
```
当然，为了简化代码，可以用python3的lambda函数来解决：
``` python
def count():
    fs = []
    for i in range(1,4):
        def f(x):
            return lambda: x*x
        fs.append(f(i))
    return fs 
    
f1,f2,f3 = count()
f1() => 1
f2() => 4
f3() => 9
```
看到这里，发现这和JS里关于变量作用域的问题简直如出一辙，在JS里，可以通过es6的 let 替换 var ，或者立即执行函数来解决这个问题，但好像都不能应用在python里，没关系，还可以新建一个变量来保存原来变量的指向，应用在python里就是这样的：
``` python
def count():
    fs = []
    for i in range(1,4):
            def f(j=i):
                    return j * j
            fs.append(f)
    return fs

f1,f2,f3 = count()
f1() => 1
f2() => 4
f3() => 9
```
啊，世界如此美好，哈哈，看来我大JS还是可以一统江湖的

给自己定一个小目标，先杀少林，再灭武当，啊不对，玩儿转**python**，搞定**Node**，早日练就前端界的**降龙十八掌**，加油！

