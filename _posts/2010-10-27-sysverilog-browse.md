---
layout: post
tagline: "Supporting tagline"
category: Digital
tags: [ICDesign]
title: "systemverilog语法浏览"
---

# Systemverilog中的类和对象

来源：http://systemverilog.co/wiki/Objects_and_Classes（可惜此来源已经不复存在)

# 类是Systemverilog中许多关键概念的核心。 

1. 和其他面向对象的语言相似，类是面向对象的关键 

2. 类对象通过引用传递，而其他Systemverilog类型通过值传递。 

3. 类是systemverilog随机化的关键。 

4. 约束(constraint)必须被申明为类成员。

# Systemverilog中的对象

​    在很多方面，Systemverilog中的类和对象的概念像Java胜过像C++. 

​    如同Java一样, Systemverilog实现了一个垃圾回收器，用户负责创建对象，只要此对象的所有引用都没有了，此对象就被标记销毁。  这个特点很方便，用户不需要关心内存泄漏。尽管如此，用户仍然需要保证对对象的引用尽量减少，特别要关心容器类对象；用户需要保证没有用的数据常常被清理掉。 

​     Systemverilog仅仅将类的实例视为对象。对象总是映射到堆中的一段内存（使用new()构造函数），而语言内建数据类型(以及struct/union/enum等)不会被映射到堆上。不允许对内建数据类型使用new操作，而是根据其申明的方式来决定到底映射到栈中还是静态内存区域。

​    当申明一个对象（类的实例）的时候，仅仅创建了一个类的引用【reference】，此引用被初始化为null(空引用)。  引用和C/C++中的指针相似。Systemveilog中的关键字null和C/C++中的空指针NULL相似。在Systemverilog中，对象通过引用和变量进行绑定(在语言参考手册中也被称为句柄【handle】). 变量不能够通过值来绑定对象。 

   除了null值之外，类实例的变量可以赋值为其他相同类型对象的句柄，或者赋值为一个从构造函数new()返回的全新句柄。 

   在Systemverilog中，当对象作为函数的参数，总是以引用的方式进行传递；另一方面，内建类型总是以值进行参数传递的。同样的，函数总是以引用来返回对象，但是总是通过值来返回内建数据类型。 

   当一个对象赋值给其他变量，或者作为函数参数传递，或者被放入一个容器，则对象引用计数加1。当对象的变量离开了作用域，则引用计数减1。如果没有任何变量绑定对象的句柄，那么此对象就被垃圾收集器标记。 

​     对象通过引用传递这个特点在两个方面特别重要：首先，类被用来创建复杂的数据结构，将会比原始内建数据类型消耗更多的内存资源，因此，通过引用传递对象在运行时将更有效率。更重要的是，通过引用传递对象是面向对象设计的范式.一个类对象常常会在其申明的作用域之外还保持生命，保持自生的状态；因此，即使通过函数传递，也需要保持一致性。因为对象是通过引用传递的(并不是通过复制方式)，所以他们在不同的函数作用域中都保持了一致性(补充：指向同一块memory)。 

# 类的创建

Systemverilog类是一个自包含的软件模块。使用类来创建高级(抽象的，具体的)数据类型。用systemverilog的关键字class来申明一个类. 

实例：类和对象创建

```verilog
class Complex;
  local int real_;
  local int imag_;

  // constructor; 
  function new(int re_, int im_);
    real_ = re_;
    imag_ = im_;
  endfunction: new

  // methods
  function Complex add(Complex x, Complex y)
    Complex z;
    z = new(x.real_ + y.real_, x.imag_ + y.imag_);
    return z;
  endfunction: add
    
endclass: Complex

Complex foo = new(1.1, 2.5); // new allocates an object and foo is assigned the 
                             // handle corresponding to the new object.
                             // 新生成一个对象并将此新对象句柄赋值给变量foo

Complex bar, frop; // bar, and frop are null at this point  => bar, frop此时都是null

bar = foo;  // bar copies the handle foo also holds   => 变量bar复制了foo绑定的对象句柄

frop = bar.add(foo); // frop is assigned a new handle which got created by invocation of new
                     // and got returned by the add method => frop被赋值为一个新的句柄，
                     // 此句柄是从add方法中返回的，通过调用new得到的.

```

   以上代码描述了一个典型的systemverilog类，几个观点叙述如下：

1. 正如C++类，Systemverilog类包含属性和方法。属性是数据成员，用来存储对象的状态。方法代表类可能执行的动作。 
2. 属性可以是任意用户定义的数据类型或者是内建数据类型 
3. 类方法可以是任务【task】和函数【function】。 
4. 属性和方法可以申明为静态类型 
5. 类成员可以申明为本地【local】和被保护【protected】。 
6. 本地成员仅仅在类的内部可见。被保护成员在类的内部可见，以及在继承此类的子类中可见。类成员缺省是公共的(完全可见). 
7. 类可以从其他类继承而来。Systemverilog通过关键字extends支持继承。 
8. 类可以申明其成员为静态(static)，静态成员将被所有的类对象共用. 

  

​     使用类来进行行为级建模以及编写验证组件。类结构不能被合成，因此不能被用来进行寄存器传输级设计。 

   在验证中，类被用来对所有东西建模，从简单的交易【transaction】，到复杂的产生器【generator】，驱动器【driver】，观察器【monitor】，记分牌【soreboard】。即使通讯通道【channel】也使用类来建模。 

​    很多时候设计类，是通过其公共方法和客户进行交互。类的数据成员代表对象的状态，用户不能通过直接操作修改对象的状态。通常在面向对象设计中，数据成员被申明为私有(使用local关键字)或者保护(protected关键字)。此原则在systemverilog中并不是总是适用，在Systemverilog中，通常将可以随机化的数据成员以及其对应的约束申明为公共(public)。

# Systemverilog枚举类型 

来源：http://systemverilog.co/wiki/Enumerations

####  

#### 将整型向下转型到枚举类型

 

枚举类型是强类型，因此，枚举变量不能够被赋值为另一个整数类型值，除非显示的转型（静态转型或者动态转型）。静态转型不会去检查赋值是否满足枚举取值范围的限制范围，而动态转型会检查，除非很明确知道取值范围满足要求，否则请使用动态转型。 

使用类型转换对枚举变量赋值 

```verilog
typedef enum {MON, TUE, WED, THU, FRI, SAT, SUN} Weekday;
Weekday day1 = WED; // no type conversion required
day1 = 2;      // illegal type conversion
day1 = Weekday’(2);  // OK
day1 = Weekday'(10);  // legal out of range assignment forced by static cast
if(! $cast(day1, 10)) // legal dynamic cast operation that would FAIL
    $display("Dynamic cast operation FAILED"); 
```

#### 将枚举类型向上转型到整形。

 枚举类型和标准整数类型之间的互操作是合法的，不需要特别的语法。当参与运算的时候，枚举类型变量会自动向上转型成其他整数类型。 

#### 枚举类型的方法

| 函数名字      | 说明                                           |
| ------------- | ---------------------------------------------- |
| first()       | 返回枚举类型的第一个值                         |
| last()        | 返回枚举类型的最后一个值                       |
| next(int N=1) | 枚举值加Ｎ，如果超过范围，将继续从第一个值开始 |
| prev(int N=1) | 枚举值减Ｎ，如果超过范围，继续回到最后一个     |
| num()         | 返回枚举类型的取值个数                         |
| name()        | 返回枚举值对应的字符串                         |


name()方法在打印调试信息时特别有用。

 

#### 参考文献： 

1. [↑](http://systemverilog.co/wiki/Enumerations#cite_ref-0) Srinivas Venkataraman, *VerificationOnWeb (VoW)*, 2009, "[SystemVerilog tip: watch out enum and randc](http://www.cvcblr.com/blog/?p=55)", 08-AUG-2010 