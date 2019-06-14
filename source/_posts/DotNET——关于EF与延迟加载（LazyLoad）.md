---
title: DotNET——关于EF与延迟加载（LazyLoad）
date: 2019-04-26 11:09
categories: 极客之道
tags: [C#,EF]
---

- 前一段独立负责了一个web项目，由于自己一个人开发，技术选型也更加自由一些，因为项目相对没那么复杂，抛弃了公司一直沿用的```IBatis.Net```，用了微软自己的ORM框架——EF（Entity Framework），在使用的过程中遇到了些许问题，发现了EF的延迟加载特性，在使用的过程中，深入研究了一下，正好结合一些基础知识，做了些深入的学习。
- 关键字：```虚方法``` ```多态```
####基本操作
由于项目使用的EF的```Code First```开发方式，并且使用Fluent API配置实体类与表的映射，在OnModelCreating方法里通过```modelBuilder.Configruations.AddFromAssembly(Assembly.GetExecutingAssembly())```加载所有继承自```EntityTypeConfiguration<>```的配置类为模型类进行配置。
```
public class EFDbContext : DbContext
{
     public EFDbContext() : base("name = connstr"){}
     protected override void OnModelCreating(DbModelBuilder modelBuilder)
     {
        base.OnModelCreating(modelBuilder);
        modelBuilder.COnfigurations.AddFromAssembly(Assembly.GetExecutingAssembly());    
     }
}
```
```
public class Class
{
    public long Id { get; set; }
    public string Name { get; set; }
}
```
```
using System.Data.Entity.ModelConfiguration;
class ClassConfig: EntityTypeConfiguration<Class>
{
    public ClassConfig()
    {
        ToTable("T_Class");
    }
}
```
- 当多表之间需要配置主外键关系时，需要在多端进行配置
```
public class Student
{
    public long Id { get; set; }
    public long ClassId { get; set; }
    public virtual Class Class { get; set; }
    public string Name { get; set; }
}
```
```
class StudentConfig : EntityTypeConfiguration<Student>
{
    public StudentConfig()
    {
        this.ToTable("T_Student");
        this.HasRequired(e => e.Class).WithMany().HasForeignKey(e => e.ClassId);
    }
}
```
调用情况如下
```
using (EFContext ctx = new EFContext())
{
    Student s = ctx.Students.First();
    Console.WriteLine(s.SName);
    Console.WriteLine(s.Class.ClassName);
}
```
打印一下sql调用过程和结果如下
```
EF执行SQL:Opened connection at 2019/4/26 9:58:03 +08:00

EF执行SQL:SELECT TOP (1)
    [c].[Id] AS [Id],
    [c].[ClassId] AS [ClassId],
    [c].[SName] AS [SName]
    FROM [dbo].[T_Student] AS [c]
EF执行SQL:

EF执行SQL:-- Executing at 2019/4/26 9:58:03 +08:00

EF执行SQL:-- Completed in 0 ms with result: SqlDataReader

EF执行SQL:

EF执行SQL:Closed connection at 2019/4/26 9:58:03 +08:00

小王
EF执行SQL:Opened connection at 2019/4/26 9:58:03 +08:00

EF执行SQL:SELECT
    [Extent1].[Id] AS [Id],
    [Extent1].[ClassName] AS [ClassName]
    FROM [dbo].[T_Class] AS [Extent1]
    WHERE [Extent1].[Id] = @EntityKeyValue1
EF执行SQL:

EF执行SQL:-- EntityKeyValue1: '1' (Type = Int64, IsNullable = false)

EF执行SQL:-- Executing at 2019/4/26 9:58:03 +08:00

EF执行SQL:-- Completed in 1 ms with result: SqlDataReader

EF执行SQL:

EF执行SQL:Closed connection at 2019/4/26 9:58:03 +08:00

三年二班
```
通过以上操作可以发现，当分别打印Student和Class的字段时，会两次调用SQL语句，这就是EF的延迟加载。
####深入延迟加载
延迟加载是EF的默认特性，在平时调用时，只有通过循环或者类似ToList()操作时才会真正操作SQL语句。而在Student实体类中，当需要添加Class作为外键时，需要添加```public virtual Class class{get;set;}```，如果没有添加```virtual```会怎么样呢？
```
EF执行SQL:Opened connection at 2019/4/26 10:20:48 +08:00

EF执行SQL:SELECT TOP (1)
    [c].[Id] AS [Id],
    [c].[ClassId] AS [ClassId],
    [c].[SName] AS [SName]
    FROM [dbo].[T_Student] AS [c]
EF执行SQL:

EF执行SQL:-- Executing at 2019/4/26 10:20:48 +08:00

EF执行SQL:-- Completed in 4 ms with result: SqlDataReader

EF执行SQL:

EF执行SQL:Closed connection at 2019/4/26 10:20:48 +08:00

小王

未经处理的异常:  System.NullReferenceException: 未将对象引用设置到对象的实例。
   在 ConsoleApp.Program.Main(String[] args) 位置 D:\Work\Code\ConsoleApp\ConsoleApp\Program.cs:行号 37
```
可以发现，当SQL执行完Student的访问后，去访问Student里面的Class，然后就报错了```未将对象引用设置到对象的实例。```

简单分析一下，就很容易发现，在Student类中只是定义了一下Class类型的Class字段，可是用没有赋值，没有初始化对象，直接调用，肯定会报错，可是为什么加上```virtual```就可以了呢？
- 继续深入
```
Student s = ctx.Students.First();
Console.WriteLine(s.Name);
Console.WriteLine(s.Class.Name);
Console.WriteLine(s.GetType());
Console.WriteLine(s.Class.GetType());

控制台：
System.Data.Entity.DynamicProxies.Student_E7E50A6894411395155465B3AA4894D4DAE31B725BE1AD63AA27469E722AB9D3
ConsoleApp.Class
```
通过分别打印s和s.Class的GetType()可以发现，**拿到的对象其实是Student子类的对象，因此EF其实是动态生成了实体类对象的子类，然后override了这些virtual的属性**，在内部进行操作，将Student的Class属性进行赋值，这样就解释的通了。
####virtual和override
EF的延迟加载其实用到了C#的多态性，通过方法重写和虚方法的调用，实现了根据不同的需要，分别加载数据，之前阅读过一本书《net 4.0面向对象编程漫谈》，其中有一个章节就非常形象的介绍了C#的这一特性。
```
class Parent
{
    public void HideF()
    {
        Console.WriteLine("Parent.HideF()");
    }
}

class Parent : Parent
{
    public void HideF()
    {
        Console.WriteLine("Child.HideF()");
    }
}
当子类和父类都拥有了一个完全相同的方法HideF，问题发生了：
Child c = new Child();
c.HideF();      //输出：Child.HideF()
修改一下代码：
Parent p = new Parent();
p.HideF();     //输出：Parent.HideF()

由此得出结论：当分别位于父类和子类两个方法完全一样时，调用哪个方法由对象变量的类型决定。

继续测试：
Parent p = new Child();
p.HideF();     //输出：Parent.HideF()

继续得出结论：当分别位于父类和子类的两个方法完全一样时，调用哪个方法由对象变量的编译时类型决定。
如果确实希望调用的是子类的方法，应先进行强制类型转换：
((Child)p).HideF();    //输出：Child.HideF()

所以，如果父类和子类方法重名，应该子类同名方法前加上new关键字public new void HideF(){}，
如果要调用父类的同名方法，可以使用base.HideF();
```
*由于子类隐藏了父类的通过名方法，如果不进行强制转换，就无法通过父类变量直接调用子类的同名方法，哪怕父类变量引用的是子类对象。*

> **为了达到这个目的，需要在父类同名方法前加关键字virtual，表明这是一个虚方法，子类可以重写此方法。与此同时，需要在子类同名方法钱加关键字override，表明对父类同名方法进行重写。**
- 修改之后的结果
```
Child c = new child();
Parent p = c;
p.HideF();    //输出：Child.HideF()

这一示例表明：将父类方法定义为虚方法、子类重写同名方法后，通过父类变量调用此方法，到底调用父类还是子类的方法，由父类变量引用的真实对象类型决定，而与父类变量无关。
```
可见，根据C#的虚方法调用特性，可以在同样的语句下，通过引用不同的类型子类对象，就可以实现不同的功能。


