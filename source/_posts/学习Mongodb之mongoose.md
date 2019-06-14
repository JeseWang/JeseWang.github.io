---
title: 学习Mongodb之mongoose
date: 2017-05-25 21:30:00
categories: 极客之道
tags: [mongodb]
---
一直以来对数据库的概念总是很模糊，于是花了一周的时间集中学习`mongodb`数据库，了解了数据库的基本部署，配置和操作，终于可以顺利的把`mongodb`运行在`Mac，Window和Linux`中了，同时对`Linux`也更深入了一个层次，而再次使用`mongoose`作为中间件来操作`mongodb`时，也有了很多新的认识……
<!--more-->

#### 基本操作
关于mongoose的基本操作其实网上很多教程，按照mongoose官网的[quick start guide](http://mongoosejs.com/docs/index.html)，很快就能有个大概的了解。
```
$ npm install monoose --save

var mongoose = require('mongoose')
mongoose.connect('mongodb://ip:port/database')
```
这里注意，很多教程都没有详细说明对于有权限的数据库的连接方式，也让我在刚接触时郁闷了很久，数据库加密是防止数据库发生损失的第一道锁，如下(在数据库中操作)
```
$ use database
$ db.createUser({
    user: user,
    pwd: pwd,
    roles:[
        {
            role: "readWrite ...", // 具体的权限
            db: "" //datebase
        }
    ]
})
$ db.auth("userName","password")
```
注意，在服务器上，如果要直接操作数据库，可以先连接，再通过`db.auth()`来验证权限，可是如果在`node`中该怎么做呢？

example：我的数据库账号是admin,密码为password,需要连接的database为test，数据库地址是111.222.333.444:1024那么再连接的时候就可以直接指定连接的地址为
```
mongoose.connect('mongodb://admin:password@111.222.333.444:1024/test')
```
这样就不会出现由于权限的问题而无法连接了，但是要注意，在mongodb的admin下建的账户，在子数据库中是不能直接验证的，同时也为了安全，应该在每个数据库下都建立不同权限的账户

```
$ use admin //切换到admin
$ db.system.users.find().pretty()   //查看系统下所有的账户
$ use test  //切换到test
$ show users  //查询test下的账户
```

#### 连接数据库
在成功连接了数据库并通过正好验证以后，就该对数据库进行增删改查了，很多关于`mongoose`的入门的教程都是寥寥几句，尤其对于最核心的`Schema`和`Model`，并没有过多的深入，而官方文档对于刚接触不深的我来说，也是比较吃力，下面是个人在学习过程中的一些小小的总结

`Schema`是mongoose的一种模式，这是比较抽象的一个东西，它对应的是数据库中的`collection`,比如，`db.collection.find({})`的数据是这样的
```
{
    name: 'Jack',
    age: '25',
    date: '2017-05-25'
}
```
那么对应的这个Schema就是这样的：
```
{
    name: {type: String},
    age: {type: Number},
    date: {type: Date}
}
```
Schema定义了一个集合中数据的基本格式，但控制数据的，其实是model,关于model稍后再细谈。

好了，现在就可以进行下一步了：
```
var Schema = mongoose.Schema;
var newSchema = new Schema(
    {
        name: {type: String},
        age: {type: Number},
        date: {type: Date}
    },
    {
        collection: "" // 这个是什么？？？
    }
)

module.exports = mongoose.model("User", newSchema)
```
在这一步里，我们成功的创造了一个`Schema`的实例，也就是一个新的模式，并通过`mongoose.model()`生成了一个model，而具体操作数据库的，就是这个`model`,比如这里的`User`，下面就插入一条数据：
```
// 先require()到刚才的文件
var user = new User({
    name: 'jesse',
    age: 25,
    date: new Date()
})
user.save(function(err, res){...}
```
我们可以查看一下这个数据
```
User.find(function(err, res){
    console.log(res)
})

```
可以成功打印出这个数据库集合，里面已经有了这条数据

看一下服务器：
```
$ show collections

users

$ db.users.find()
......
......  //具体的数据

```
也没有问题，可是，这个`users`是怎么来的呢？还有刚刚在定义Schema时，那个collection是干什么的？我们怎么从已有的数据库中取数据呢？
带着这三个问题，我们继续探索！

#### 继续探索
刚才，我们定义了数据的模式，生成了model，并成功的插入了一条数据，可是我们并没有定义这个`collection`的名字呀，怎么多出来了一个`users`，经过网上的搜索，应该是mongoose在生成数据时，自动在`User`等Schema的后面加上了`s`，并转换为小写，作为`collection`的默认名字，但如果想要自定义这个名字该怎么做呢?

可以这样
```
mongoose.model("User", newSchema,"userseseseses...")
```
也就是model()的第三个参数，也可以这样
```
var newSchema = new Schema({
    ...
},{
    collection: "collectionName"
})
```
即Schema({},{})的第二个参数，既然是这样，我就猜想，从指定的数据库中取数据应该可以用同样的方法
```
mongoose.model('User',new Schema(),'collectionName')

...

User.find(function(err, res){
    console.log(res)
})

......

```
果然，将刚才定义的第三个参数集合下所有的数据打印了出来！

`mongoose储存数据`的方式有两种，一是通过创建Model的实例，用save()方法，前面已经写过：
```
var model = new Model({
    name: ...,
    age:...,
    ....
})
model.save(callback)
```
还有就是可以直接用Model进行创建，需要用到Model的Create()方法
```
var params = {
    name: ...,
    age: ...,
    ...
}
Model.create(params,callback)
```
需要注意的是，通过Model直接创建的数据，传入的params需要使用`Json`格式。

#### 小结
关于mongoose的操作还有很多，比如update,remove,findOne等等等等，还有索引什么的，其实，只要对`mongodb`数据库的知识有一个全面的了解和基本的掌握，理解`mongoose`还是很简单的，具体的操作和方法，多看看官方的文档，很快就能掌握，加油，共勉！