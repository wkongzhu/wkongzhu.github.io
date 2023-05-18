---
layout: post
tagline: "Supporting tagline"
author: Zhu,YH
category: verification
tags: [verification]
title: "systemverilog语法浏览"
---

# Systemverilog中的类和对象

来源：http://systemverilog.co/wiki/Objects_and_Classes（可惜此来源已经不复存在)

## 类是Systemverilog中许多关键概念的核心。 

1. 和其他面向对象的语言相似，类是面向对象的关键
2. 类对象通过引用传递，而其他Systemverilog类型通过值传递。 
3. 类是systemverilog随机化的关键。 
4. 约束(constraint)必须被申明为类成员。

## Systemverilog中的对象

​    在很多方面，Systemverilog中的类和对象的概念像Java胜过像C++. 

​    如同Java一样, Systemverilog实现了一个垃圾回收器，用户负责创建对象，只要此对象的所有引用都没有了，此对象就被标记销毁。  这个特点很方便，用户不需要关心内存泄漏。尽管如此，用户仍然需要保证对对象的引用尽量减少，特别要关心容器类对象；用户需要保证没有用的数据常常被清理掉。 

​     Systemverilog仅仅将类的实例视为对象。对象总是映射到堆中的一段内存（使用new()构造函数），而语言内建数据类型(以及struct/union/enum等)不会被映射到堆上。不允许对内建数据类型使用new操作，而是根据其申明的方式来决定到底映射到栈中还是静态内存区域。

​    当申明一个对象（类的实例）的时候，仅仅创建了一个类的引用【reference】，此引用被初始化为null(空引用)。  引用和C/C++中的指针相似。Systemveilog中的关键字null和C/C++中的空指针NULL相似。在Systemverilog中，对象通过引用和变量进行绑定(在语言参考手册中也被称为句柄【handle】). 变量不能够通过值来绑定对象。 

   除了null值之外，类实例的变量可以赋值为其他相同类型对象的句柄，或者赋值为一个从构造函数new()返回的全新句柄。 

   在Systemverilog中，当对象作为函数的参数，总是以引用的方式进行传递；另一方面，内建类型总是以值进行参数传递的。同样的，函数总是以引用来返回对象，但是总是通过值来返回内建数据类型。 

   当一个对象赋值给其他变量，或者作为函数参数传递，或者被放入一个容器，则对象引用计数加1。当对象的变量离开了作用域，则引用计数减1。如果没有任何变量绑定对象的句柄，那么此对象就被垃圾收集器标记。 

​     对象通过引用传递这个特点在两个方面特别重要：首先，类被用来创建复杂的数据结构，将会比原始内建数据类型消耗更多的内存资源，因此，通过引用传递对象在运行时将更有效率。更重要的是，通过引用传递对象是面向对象设计的范式.一个类对象常常会在其申明的作用域之外还保持生命，保持自生的状态；因此，即使通过函数传递，也需要保持一致性。因为对象是通过引用传递的(并不是通过复制方式)，所以他们在不同的函数作用域中都保持了一致性(补充：指向同一块memory)。 

## 类的创建

Systemverilog类是一个自包含的软件模块。使用类来创建高级(抽象的，具体的)数据类型。用systemverilog的关键字class来申明一个类. 

实例：类和对象创建

```verilog
class Complex;
  local int real_;  // 类成员变量
  local int imag_;

  // constructor; 
  function new(int re_, int im_);
    real_ = re_;
    imag_ = im_;
  endfunction: new

  // methods
  function Complex add(Complex x, Complex y)
    Complex z;   // 函数内局部变量，外部不可见
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

  

​           使用类来进行行为级建模以及编写验证组件。类结构不能被合成，因此不能被用来进行寄存器传输级设计。 

​           在验证中，类被用来对所有东西建模，从简单的交易【transaction】，到复杂的产生器【generator】，驱动器【driver】，观察器【monitor】，记分牌【soreboard】。即使通讯通道【channel】也使用类来建模。 

​        很多时候设计类，是通过其公共方法和客户进行交互。类的数据成员代表对象的状态，用户不能通过直接操作修改对象的状态。通常在面向对象设计中，数据成员被申明为私有(使用local关键字)或者保护(protected关键字)。此原则在systemverilog中并不是总是适用，在Systemverilog中，通常将可以随机化的数据成员以及其对应的约束申明为公共(public)。

# Systemverilog枚举类型 

来源：http://systemverilog.co/wiki/Enumerations


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

## 枚举类型的方法

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

# SystemVerilog——字符串类型 

英文来源：http://systemverilog.co/wiki/String_Type

传统的Veriog仅仅支持文字表述上的字符串， 而SystemVerilog将字符串作为了内建的数据类型。类似C++的std::string类型，SystemVerilog字符串类型支持很多操作和函数。

SytemVerilog字符串类型支持的操作和方法：

## 一些有用的系统任务

| SytemVerilog字符串类型支持的操作和方法                       |                                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **操作**                                                     | **描述**                                                     |
| strA**==**strB                                               | 相等——操作数可以是字符串类型或者字符串文字，如果两个字符串由相同的字符序列组成，则返回1. |
| strA**!=**strB                                               | 不等于—— 对相等操作取反                                      |
| strA**<**strB strA**<=**strB strA**>**strB strA**>=**strB    | 比较——如果相应的按字典序的比较条件满足时，则返回1            |
| **{**strA**,**strB**,**..**,**strN**}**                      | 连接——扩展指定的字符串。操作符可以是字符串类型，也可以是字符串文字。当所有的操作符都是字符串文字时，此操作完成整体的连接，结果也是一个字符串文字。 |
| str**.len()**                                                | 长度——返回代表字符串的长度的整数                             |
| str**.putc(**i, c**)**                                       | 字符输入——将字符串str中的第i个字符替换成字符c。i必须是一个整数，c必须是一个字节类型的字符 |
| str**.getc(**i**)**                                          | 获取字符——返回字符串str的第i个字符。i必须是整数，返回的字符表示为一个字节。 |
| str**.toupper()**                                            | 转成大写——返回str中所有字符变成大写的字符串                  |
| str**.tolower()**                                            | 转成小写——返回str中所有字符变成小写的字符串                  |
| strA**.compare(**strB**)**                                   | 比较——比较strA和strB.从第一个字符开始比较。如果相等，则继续后续字符，知道两者有不同或者到达某个字符串的结尾。如果相等，返回’0‘；如果strA在strB之后，则返回整数；如果strA在strB之前，则返回负数. |
| strA**.icompare(**strB**)**                                  | 比较——和compare方法类似，但是不关心大小写。                  |
| str**.substr(**i, j**)**                                     | 子串——i和j都是整数，返回一个新的字符串，由str的第i和和第j个中间的所有字符组成 |
| str**.atoi()                 **str**.atohex()                 **str**.atooct()                 **str**.atobin()** | 字符串转整数——返回一个32bit整数（不考虑此字符串表示的数字的长度），对于atoi, 字符串将被看作十进制；对于atoh，十六进制；对于atooct,八进制；对于atob，二进制。对于比较大的数字，使用系统任务$sscanf更加合适[1]. |
| str**.atoreal()**                                            | 字符串转实数——此方法返回字符串str表示的实数                  |
| str**.itoa(**i**)                 **str**.hextoa(**i**)                 **str**.octtoa(**i**)                 **str**.bintoa(**i**)** | 整数转字符串——atoi,atohex,atooct,atobin的反运算。此系列方法输入一个整数i，将其对应的表示方式存在字符串str中。 |
| str**.realtoa(**r**)**                                       | 实数转字符串——atoreal的反运算。此方法输入一个实数r，将其对应的表示方式存在字符串str中。 |

  

字符串在构建日志信息的时候非常有用。有许多很方便有用的SystemVerilog系统函数，以下列举其中的几个：

## 有用的SystemVerilog系统任务

| 系统任务                        | 任务说明                                                     |
| ------------------------------- | ------------------------------------------------------------ |
| \$**sscanf**(str,format,args);  | \$sscanf 将字符串按照某个模板格式进行扫描，其字符串格式和C语言中的printf()函数类似 |
| \$**sformat**(str,format,args); | \$sformat是\$sscanf的反函数。将字符串按照给定的格式填入相应的参数args中 |
| \$**display**(format,args);     | \$display就是Verilog的printf语句，在stdout上显示格式化的字符串 |
| \$**sformatf**(format,args);    | $sformatf任务和\$sformat相似，除了其返回字符串结果。字符串作为\$sformatf的返回值，而不是像\$sformat一样放在第一个参数上。 |

### **注意**

- 一个流行的\$sformatf的替代方式是\$psprintf. 它实际上是由Vera遗留下来的。$sformtf  在后来成为了SystemVerilog的语言标准。然而，大部分流行的SystemVerilog编译器都支持\$psprintf, 尽管它没有成为标准。如果想符合标准，建议使用\$sformatf. 

## 参考资料

1. [↑](http://systemverilog.co/wiki/String_Type#cite_ref-0) Dave Rich, *Verification Guild*, 2007, "[String manipulation through atohex();](http://verificationguild.com/modules.php?name=Forums&file=viewtopic&t=1798)", accessed on August 22 2010 
2. [↑](http://systemverilog.co/wiki/String_Type#cite_ref-1) Ben Cohen, *Verification Guild*, 2006, "[$psprintf // In VCS not in P1800](http://verificationguild.com/modules.php?name=Forums&file=viewtopic&p=6370)", accessed on August 22 2010 

# SystemVerilog——容器类型(1):动态数组

来源：http://systemverilog.co/wiki/Container_Types

 Verilog仅仅提供了数组作为数据集合，并且此标准数据类型有诸多限制：

- 数组的长度是一个常数，尽管固定长度数组对硬件建模很好，但是对行为级建模验证来说灵活性不够。     
- 由于数组的长度是固定的，插入操作非常困难 
- 很难对比较大的稀疏矩阵建模 

为了克服这些限制，SystemVerilog引入了一些全新的数组类型，包括：

- 动态数组 
- 列表 
- 队列 
- 关联矩阵 

## 动态数组

动态数组类似C++中的new[]操作。可以在运行时指定数组大小。也可以删除以及重新分配动态数组。一旦动态数组被创建，将具有未打包的数组功能。动态数组必须在初始化之前先声明。

```verilog
// declaration
data_type array_name [];
 
// allocation
array_name = new[size_expression];
 
// allocation along with declaration
data_type array_name = new[size_expression];
 
// declaration and initialization
array_name = new[size_expression](initialization_array);
 
// assignment
array_name = another_array;
```

支持如下操作：

**new[]()**

new[]函数将会为数组分配内存。此函数需要一个整数参数作为数组的大小，以及一个可选参数表示数组的类型。此可选参数可以是一个固定长度的数组，或者是另外一个动态数组，抑或是其他的容器类型（队列/列表），甚至是文字数组。如果此设置了参数，此参数用来对动态数组的元素进行初始化。

```verilog
int size = 4;
logic [7:0] data[]; // array declaration
data = new[size]({32, `bx, 0, `hBD});
data = new[2*size];
```

注意：用来指定数组大小参数的表达式不需要在编译时间是常数。可以对某个动态数组多次分配内存，之前分配的内存将会被垃圾回收器回收。

**delete()**

SystemVerilog为动态数组定义了delete()方法，此方法显式的收回动态数组的内存，类似C++中的delete[].

 

**size()**

size()方法返回值为数组长度的整数。对还没有被分配内存的动态数组以及被删除的数组，此函数返回'0'.

```verilog
module top;
  initial begin
    bit [3:0] foo [], bar[];
    int count = 16;
    $display(foo.size());   // displays 0;
    foo = new[count];
    $display(foo.size());   // displays 16;
    bar = new[20](foo);
    $display(bar.size());   // displays 20;
    foo.delete();
    $display(foo.size());   // displays 0;
   end
endmodule: top
```

#  SystemVerilog——容器类型(2):List

英文来源：http://systemverilog.co/wiki/Lists

##  Lists

请不要使用List.vh -- 不建议使用 (在IEEE 1800-2009标准中). 请看本文章最后.

### 标准List.vh包

IEEE 1800-2005标准定义了一个链表包. 此链表没有被编译器内建，所以必须将List.vh包含进来才能使用.

此文件实现了一个双向链表。此数据结构适用于，需要多次对其中间部分进行插入对象。但是索引操作代价较大，不是固定时间消耗。当有对象插入的时候，链表容器的大小可以动态调整。

当需要很多索引操作的时候，请不要使用表结构。当表越来越大时，索引操作越来越耗时间。因为，对于每次索引操作，必须从开头遍历到需要访问的成员。最佳使用表的方式是使用迭代的情况。

为了使用表，必须包含此文件。一旦参数化了一个表的实现， 创建一个类型为T的链表如下： 

```verilog
`include <List.vh>

List#(T) dl; // dl is a List of type 'T' objects
List_Iterator#(T) dl_itor;
 
// Alternately use typedef to create new type
typedef List#(T) ListT;
typedef List_Iterator#(T) ListT_Iterator;
 
ListT dlt; // dlt is a List of type 'T" objects
ListT_Iterator dlt_itor;
```

### **IEEE 1800-2009中不建议使用**

最新的SystemVerilog标准不建议使用List.vh. 为什么?

mantis缺陷追踪系统中这部分记录的说明[[1\]](http://systemverilog.co/wiki/Lists#cite_note-0), List.vh的实现有些问题，用到了参数化类，但是类本身不是包的一部分. 使用`include方式用户可能会在多个名字空间中包含此文件，导致对链表结构有重复的多重定义，并且可能互不兼容。 

后来，有提到可能会实现一个更加完整的STL类似的通用数据结构，使之成为SystemVerilog标准的一部分。但是短期内很难实现。

## 参考资料

1. [↑](http://systemverilog.co/wiki/Lists#cite_ref-0) *SystemVerilog Mantis*, 2006, "[Issues with List.vh](http://www.eda.org/mantis/bug_view_page.php?bug_id=1256)", accessed on August 24 2010 

#  SystemVerilog——容器类型(3):队列 

英文来源：http://systemverilog.co/wiki/Queues

##  队列 

​         SystemVerilog语言手册提供了队列容器， 基本上是一个双端口队列[[1\]](http://systemverilog.co/wiki/Queues#cite_note-0). 队列就好象一个一维数组，可以自动在两端增长和减小。队列数据结构有如下特征： 

1. 当新增或者删除成员的时候，队列的大小自动调整. 
2. 任意成员的访问时间固定. 
3. 队列首尾插入新成员的时间固定. 
4. 支持通用的SystemVerilog数组操作，比如索引，分段，赋值，相等，连接. 

​        SystemVerilog中的队列很像C++ STL库中的std::deque. 为了更好的理解此数据结构的功能以及使用限制，最好对其实现方式有一定了解。 尽管语言手册上并没有其详细的实现细节，右图演示了其内部实现。虽然也有其他的实现方式，但是这种方式简单，详实，足够我们理解。[[2\]](http://systemverilog.co/wiki/Queues#cite_note-1).

​     当队列被声明后，它将被分配一块连续的内存区域，其头尾迭代指针被初始化用来跟踪队列两端的状态。如果一直往队列的某一段增加成员，这块内存最终将被耗尽，此时，内存管理器会从系统中再分配一块内存用来存放新的数据。 当需要访问某一个特定的成员时，内存管理器简单的查询该成员存放的相应内存块。一旦找到其位置，访问该成员仅仅需要一个类似的数组访问操作。

   当成员从队列的某一端被删除时，有可能让内存块得到释放，内存管理器将此内存释放给系统.

   现在我们来看看在队列中间增加和删除一个成员的操作方式。因为内存块本身是连续且固定大小的，这些操作可能导致这些数据成员的物理存储位置移动以提供位置给新插入的成员（删除的时候，是填补空位）. 根据列表的长度和操作成员的位置，插入和删除操作可能会移动很多成员，因此队列的插入和删除操作不是固定时间。

## 队列的声明和使用

​     SystemVerilog中的队列声明是通过在标识符加后缀[$]. 例如，声明一个unsigned int类型的队列，使用如下方式：

```verilog
int unsigned intQ[$];
```

​         缺省时，队列没有任何成员。正如数组一样，队列可以在声明的时候被初始化为一些成员. 这些用来初始化队列的成员可以是文字型数组，也可以来自另外一个已经存在的数组:

```verilog
// Initializing a Queue  
int unsigned intQ[$] = {1, 2, 3, 5, 8,  13}; 

// An already existing array may be used  to  
// initialize a queue  
int unsigned intDA[] = {1, 2, 3, 5, 8,  13};  
int unsigned intQalt[$] = intDA;
```

​    队列声明可以限制长度。限制长度的队列成员将不会超过最大长度. 如果你只需要一个小队列，并且确认不会超过某个范围，那么最好声明成限制长度的队列。特别是队列本身是类的一部分。如果没有声明为限制长度，那么队列至少会消耗一整个内存块(如上描述的架构). 一个内存块一般占用1K甚至更多(由编译器实现决定). 为了让队列消耗更少的内存，队列声明为限制长度的：

```verilog
// A queue that can grow only up to a  size of 32  
int unsigned intBQ[$:32];
```

​         一旦队列被声明，其成员可以像普通的数组一样被访问。实际上，SystemVerilog手册认为队列是通用未打包的数组的一种[[3\]](http://systemverilog.co/wiki/Queues#cite_note-2)[[4\]](http://systemverilog.co/wiki/Queues#cite_note-3). 类似其他数组，队列通过值传递。如果需要传递队列的引用，需要将参数声明为ref.

队列可以和其他队列以及其他类型的数组直接进行复制[[5\]](http://systemverilog.co/wiki/Queues#cite_note-4).

 

## 队列的方法和操作

​     除了所有的数组都支持的索引和片选操作，SystemVerilog队列还支持许多其他操作：

| **队列的方法和操作** |                                                              |
| -------------------- | ------------------------------------------------------------ |
| **方法/操作**        | **功能描述**                                                 |
| q[a]                 | 索引操作——返回对应索引的成员。如果超出范围，**则返回此成员类型的缺省值。** |
| q[a:b]               | 片选操作——返回指定范围的队列成员。如果a为负数，则从0开始，如果b大于最后成员的索引，则截至到最后一个成员。 |
| q.size()             | 返回队列中成员的个数                                         |
| q.insert(index,item) | 指定索引处插入item. 第1个参数index必须是整数，第2个参数item必须和队列成员类型一致。如果队列已经到达设定的大小(队列满)，则最后一个成员被隐式的删除 |
| q.delete(index)      | 从队列中删除对应索引的成员                                   |
| q.delete()           | delete方法被重载。当没有索引参数的时候，整个队列被清除，等价于: q = {}. |
| q.push_back(item)    | 将item增加到队列尾部，如果队列满了，操作将失败               |
| q.pop_back()         | 删除并返回队列的最后一个成员                                 |
| q.push_front(item)   | 将item加到队列前面。如果队列已经满了，**则最后一个成员将被隐式的丢掉** |
| q.pop_front()        | 删除并返回队列的第一个成员                                   |

 

##  参考资料

1. [↑](http://systemverilog.co/wiki/Queues#cite_ref-0) *Wikipedia*, "[Double-ended queue](http://en.wikipedia.org/wiki/Double-ended_queue)", accessed on August     25 2010 
2. [↑](http://systemverilog.co/wiki/Queues#cite_ref-1) Nicolai M. Josuttis, *Addison Wesley*,     1999, "[Deque, Section 6.3 of The C++ Standard Library](http://books.google.com/books?id=n9VEG2Gp5pkC)",     accessed on September 02 2010 
3. [↑](http://systemverilog.co/wiki/Queues#cite_ref-2) *IEEE 1800-09*, Section 7.4 
4. [↑](http://systemverilog.co/wiki/Queues#cite_ref-3) *SVUG*,     "[SV language query related to dynamic array](http://svug.org/Forum/tabid/57/view/topic/postid/997/Default.aspx)",     accessed on September 02 2010 
5. [↑](http://systemverilog.co/wiki/Queues#cite_ref-4)     Puneet, *Coverification Blog*, 2009, "[Copying arrays and queues in SystemVerilog](http://coverification.com/2009/07/24/copying-arrays-and-queues-in-systemverilog/)",     accessed on September 02 2010 

 

# SystemVerilog-typedef类型定义

typedef的定义在语言中使用很广泛，特别是UVM/OVM/VMM等高级验证库，如果想了解其根本原理，必须掌握typedef的使用。

英文来源：http://systemverilog.co/wiki/Typedefs

------

​    SystemVerilog中关键字“**typedef”**有多重意义. 可以用来前向声明某种类型（还没有详细定义）；可以用来创建别名；还可以用来方便快捷的定义结构体(使用struct/union)和枚举类型.

## 目录

1. 前向 typedef声明
2. 结构化typedef定义
3. 传统typedef用法
4. 参考资料                         

## 前向typedef声明

前向申明的代码片段：

```verilog
typedef enum type_identifier;
typedef struct type_identifier;
typedef union type_identifier;
typedef class type_identifier;
typedef type_identifier;
```

SystemVerilog语言手册[[1\]](http://systemverilog.co/wiki/Typedefs#cite_note-0) 对前向声明做了如下限制：

​         尽管前向声明在对类定义很实用（8.25章）, 它不能用在组合的结构体中，因为结构体是静态声明的，并没有支持指向结构体的句柄.

​         实际上, 在真实的类型定义之前只允许对类进行前向声明. 主要原因是只有类才有对象句柄(通过引用传递). SystemVerilog中的句柄类似C++中的指针。请参考直接编程接口章节。

​         对结构体、联合体、枚举类型进行前向声明仍然有用处 -- 尽管对他们的前向声明只能和类类型一起使用. 例如, 请看下面的代码片段:

```verilog
//////////////////////////////////////////////////////////////////////
//    对类进行前向typedef声明的合法使用
//////////////////////////////////////////////////////////////////////
typedef class Class2;      // 对Class3进行前向typedef声明
 
class Class1;
  Class2 c;                // 合法 -- c是一个类对象句柄
endclass
 
class Class2;
  Class1 c;
endclass
 
//////////////////////////////////////////////////////////////////////
//   对类进行前向声明的不合法使用
//////////////////////////////////////////////////////////////////////
typedef class Class3;     // 对Class3进行前向typedef声明
 
Class3 a;                 // 合法 -- a仅仅是一个空的句柄
a = new();                // 不合法 -- 目前还不知道new函数
a.pdisplay();             // 不合法 -- 类的细节还不知道
 
//////////////////////////////////////////////////////////////////////
// 对枚举类型的前向声明的合法以及不合法使用
//////////////////////////////////////////////////////////////////////
typedef enum foo;         // 对枚举类型foo的前向typedef声明
 
foo bar;                  // 这个不合法，因为bar不是一个句柄
 
// frop是一个参数化的类型
typedef frop #(foo) fred; // 将foo作为frop类型的类型参数
 
fred xactn;               // 这个是合法的，因为前向声明的枚举类型是被间接使用的
```

## 结构化typedef定义

SystemVerilog中第二种用法  **typedef**是和结构体/联合体/枚举类型一起使用。尽管定义这些类型可以不使用**typedef**, 但是这样会造成一些限制，所以实际中几乎没有这样，举例如下：

```verilog
enum {RED, GREEN, BLUE} color1 color2;
```

定义了枚举类型变量 *color1*和 *color2*. 但是这样的原始定义并不能知道此类型的名字，并且无法在其他地方再声明和color1和color2一样的枚举类型变量。如果想要在别的地方使用同样的枚举类型变量——必须重新定义相同内容的类型（属于不同的type)。此限制对结构体和联合体同样有效。

下面通过 **typedef**来弥补这个限制，使用**typedef**可以定义一个方便使用的类型。实际使用中，在某个包中定义一个结构/联合/枚举类型，然后在使用此类型的地方将此包导入即可。

```verilog
package graphics;
  typedef enum {RED, GREEN, BLUE} color_e;
  // other common stuff
endpackage: graphics
```

​                          

## typedef用作别名

**typedef**的第3种用法是创建别名，类似C/C++中的typedef的使用方式，使得定义复杂的类型变得简单。例如，下面的代码显示了VMM库中如何创建以某种transaction类型为参数的vmm_channel类的别名 ...

`````verilog
`define vmm_channel(T) typedef vmm_channel_typed#(T) T``_channel;
`````

## 参考资料

1. [↑](http://systemverilog.co/wiki/Typedefs#cite_ref-0) IEEE, *IEEE standard 1800-2009*, 2009, "[Available for purchase from IEEE website](http://ieeexplore.ieee.org/servlet/opac?punumber=5354133)", accessed on 2010-Aug-01

# SystemVerilog——常量(const) 

之前看过《Thinking In C++》这本书，再看此文章，语言和语言之间，互有借鉴，但是却互不相同。

来源：http://systemverilog.co/wiki/Constants

**Contents**            

              1. [const限制符]()
                 2. [为什么使用常量](http://systemverilog.co/wiki/Constants#Why_use_Constants)
                 3. [注意Gotchas](http://systemverilog.co/wiki/Constants#Gotchas)
                 4. [参考资料](http://systemverilog.co/wiki/Constants#References)                     

##  const限制符

SystemVerilog允许声明常量（或称作不变量）描述符。通过在变量定义前增加const关键字来声明此变量为常量。当定义为常量的时候，必须给其提供一个值，在之后的代码里面，SystemVerilog编译器将禁止任何对此变量进行修改的动作。

除了对通常的变量进行常量声明之外，const限制符还可以用来声明函数/任务的参数为常量型。函数的参数可以是值传递，或者是引用传递。当是值传递的时候，const限制符和平常意义一样，本地参数绑定的值不能被改变。

当使用引用传递的时候，SystemVerilog编译器保证其引用不会被函数修改。普通数组（包括动态数组）是通过值传递。由于数组有可能很大，SystemVerilog允许通过关键字“ref”声明数组参数为引用方式。但是这种方式会造成一定风险，用户可能会破坏性的修改数组原来的值。通过const关键字来限制数组参数，可以避免此危险。编译器将保证在函数内部，用户不能修改此数组的值。

## 为什么要使用常量

Scott Meyers在他的经典作品"*Effective C++*[[1\]](http://systemverilog.co/wiki/Constants#cite_note-0)"中展示了在C++中使用常量的多种情况。但是，现在我们看来，大多数情况并不适用于Systemverilog中的const限制符——主要原因是当前SystemVerilog的区域(scope)限制符功能太有限。

const限制符的基本使用是用来声明常量. 可以保证值不被错误的代码改变。

```verilog
// Barker Code for FCS
const bit [63:0] fcs_barker='hB6AB31E0;

// do not change bus width dynamically
const int bus_width = 8;

// Nobody should replace Janick
const string name = "Janick Bergeron";

// no need to modify data while calculating fcs
function int fcs (const ref bytes[]);
```

------

更有意义的是使用**const**进行数组引用传递.  一个典型的使用方式是声明类的接口(通过虚函数——不要和SystemVerilog中的interface结构相混淆). 将类接口想象为一个契约，**const**限制符能帮助我们强化这个契约.

看下面xactn基类, 虚函数定义了类的接口，任何由此类扩展而来的类必须实现这两个方法。

方法*pack*要求要求返回一个字节序列，其值等于所引用的数组. 另一方面, *unpack*方法从给定的字节序列重建交易(transaction). 从方法的功能上可以看到，不能修改给定的字节序列。为了满足这个要求被满足，用户在继承xactn基类的时候，参数*bytes*需要被声明为**const**.在虚函数*unpack*中使用**const**来限制*bytes*参数, 可以确保任何*unpack*的实现都不能够修改给定的字节序列参数.

```verilog
// virtual base class
class xactn;
	// user functions
    virtual function void pack(ref bytes[]);
    virtual function void unpack(const ref bytes[]); #确保bytes不被函数体修改

	// other function declarations omitted
endclass
```

## Gotchas

- SystemVerilog中常量不是编译时的常量. 因此不允许使用常量(const)作为数组的范围参数。这一点不像C++,  其常量是编译时常量。但是，有两外一种选择，SystemVerilog用户允许使用参数(parameter)作为数组的范围（类似参数化的函数和类）. 

```verilog
// const标识符并不是编译时常量，所以不能用来作为数组的范围定义
// const int unsigned bus_width = 32;
     
// 使用参数作为数组的范围定义是合法的。
parameter bus_width = 32;
     
// 下面的声明如果bus_width是const常量，则不合法，如果是参数，就合法
logic [bus_width-1:0] data[];  
```

- 尽管可以将方法的对象类型参数声明为const, 但是const仅仅限制对象句柄有效，对象的值并没有约束效果[[3\]](http://systemverilog.co/wiki/Constants#cite_note-2). 在SystemVerilog中没有任何方式可以禁止用户修改对象本身. 但是，另一方面，在C++中，用户可以定义指针和值都是常量. （译者补充：C++这方面功能比systemverilog要强大）
- 由于上述限制，SystemVerilog不允许用户定义常量函数。比如将虚函数pack方法定义为常量函数或许很有用——可以保证其实现不会修改交易本身. (译者补充：可惜SystemVerilog并不支持). 

## 参考资料

1. [↑](http://systemverilog.co/wiki/Constants#cite_ref-0) Scott Meyers, *Addison Wesley*, July 2006, "[Effective C++, third edition](http://books.google.com/books?id=sqSvQgAACAAJ)", accessed on August 19 2010 
2. [↑](http://systemverilog.co/wiki/Constants#cite_ref-1) Bertrand Meyer, *Prentice Hall*, 1997, "[Object Oriented Software Construction](http://books.google.co.in/books?id=w8_KRAAACAAJ)", accessed on August 19 2010 
3. [↑](http://systemverilog.co/wiki/Constants#cite_ref-2) Janick Bergeron, *Verification Guild*, 2005, "[Const usage in SV](http://verificationguild.com/modules.php?name=Forums&file=viewtopic&p=6505)", accessed on August 19 2010 

# SystemVerilog——任务和函数(Tasks and Functions)

SystemVerilog从Verilog继承了任务和函数功能。任务和函数是两种用来定义子程序的方式。如果子程序需要消耗仿真时间，使用任务，否者子程序消耗仿真时间为0，则使用函数。另外，函数可以有返回值，而任务没有。

SystemVerilog给任务和函数增加了新的语义特性. 这些新的特性对高级抽象建模非常重要：

- 静态和自动作用域 
- 参数传递 
- 线程 
- 参数化函数 

------

## 静态和自动作用域   

### **Verilog中变量的作用域**

Verilog中，任务和函数中局部定义的变量是静态作用域。因此，如果多次调用函数/任务，则此局部变量将在多个函数执行线程中共享。在递归函数以及任务中通过fork-join执行多线程的情况下，常常会造成困扰。因此，Verilog的函数/任务仅仅只可能进行本地(native)递归。由于Verilog主要用来做RTL级的设计，递归函数不是必须的。

【译者查资料，猜想本地递归是指函数的调用中没有使用局部变量，仅仅只有函数自己和参数的运算，举例如下：】

```c
fibonacci(n)
if (n <2) return n 
else return fibonacci(n-1) + fibonacci(n-2)
```

### **静态和自动作用域**

静态作用域(static)变量被绑定到应用程序的数据存储区域(在这里是指仿真器的数据存储区). 此存储区域将被所有的线程所共享. 而另一方面对于自动(automatic)变量，将映射到栈区存储区， 对同一个函数进行多次调用，自动变量将映射到栈中互不相同的区域。

​        对于行为级建模，递归和多线程基本上都需要。为此，SystemVerilog允许变量以自动作用域绑定。为了在函数/任务中使用自动绑定功能，函数/任务需要声明为***automatic***.  另外，所有在**program**块中声明的函数/任务缺省都是自动作用域的。如果在自动作用域的子程序中，仍然需要使用静态变量，那么必须使用**static**关键字显式声明此变量为静态的。注意在C/C++以及其他语言中，缺省就是这样的（类似program块中的样子: 缺省情况下都是自动变量，只有显式声明static才是静态变量)。

```verilog
//Factorial using automatic binding
function automatic int unsigned factorial(int unsigned n);
      if (n = 1) return 1;
      else       return n * factorial(n-1);
endfunction: factorial
```

------

## 参数传递    

下面讲述使用函数的通用情况，大部分也是用于SystemVerilog的任务.

### **值传递**

SystemVerilog中函数参数缺省是通过值传递，适用于所有的数据类型，包括容器类型。唯一一个例外是类对象，对象本身并没有绑定到变量描述符，描述符所绑定的是对象的句柄(类似C/C++中的指针).  当一个类实例（实际上是它的句柄）被传递的时候，句柄本身是值传递，但是，因为句柄仅仅是指向真实数据，所以，被参数指向的真实的对象就和函数（任务）内部看到的对象是一样的——等效于类对象看起来是通过引用来传递的.

SystemVerilog提供了一个**ref**关键字作为函数参数的前缀。当使用**ref**时，表明参数是使用引用传递，**ref**语法类似C++中的引用.` 

有两种情况下使用**ref**做参数比较有意义。第一种情况，由于函数只能有一个返回值（不考虑传统Verilog上的input/output参数端口声明），任务没有返回值。当函数需要返回多个值或者任务需要返回一个以上值的时候，通过引用传递就用得上。

第二种情况是运行效率的考虑。当大量的数据需要作为参数传递的时候，值传递效率很低。所有的数据需要在每次函数调用的时候被复制。如果参数使用**ref**前缀，可以不需要进行数据复制。但是这样会使得参数的数据容易被函数/任务中的代码修改。此危险可以通过声明**ref**参数为常量**const**来解决.

```verilog
// virtual base class
class xactn;
      // user functions
      virtual function void pack(ref bytes[]);
      virtual function void unpack(const ref bytes[]);
      // other function declarations omitted
endclass
```



## 函数参数中的缺省值

SystemVerilog允许给函数设置缺省值。如果没有给参数指定值，并且参数有缺省值，那么函数将使用参数的缺省值。此指定缺省值的语法和C++类似.

```verilog
task void foo (int unsigned x, int unsigned y = 10);
      // the tasks implementation is not provided
endtask: foo
```

SystemVerilog提供了额外的特征，不像C++,  SystemVerilog允许用户将使用指定值的参数放在使用缺省值的参数的后面。使用缺省的参数很简单的在函数调用的参数列表中不出现，后面指定值的参数放在一个逗号后面，此逗号在一个空格之后，表示没有相应的参数(也就是不指定值)。         

```verilog
function void foo(int unsigned first = 1,
                  int unsigned second = 2,
                  int unsigned third = 3,
                  int unsigned fourth = 4);

  // function's code omitted
endfunction: foo

// foo函数的调用例子：
// 第3个和第4个参数使用缺省值
foo (10, 20);
// 第1个和第3个参数使用缺省值
foo ( ,20, ,40);
```

# SystemVerilog——类的继承

SystemVerilog支持单继承(类似Java, 而不像C++). 有一个让SystemVerilog支持多重继承的提案[1], 但是短期内不会看到曙光。



## 目录

- [1 什么是继承?](http://systemverilog.co/wiki/Class_Inheritance#What_is_Inheritance_about.3F) 
- [2 有什么好处](http://systemverilog.co/wiki/Class_Inheritance#What_have_we_gained) 
- [3 开-关定律](http://systemverilog.co/wiki/Class_Inheritance#The_Open-Closed_Principle) 
- [4](http://systemverilog.co/wiki/Class_Inheritance#References)[参考资料  ](http://systemverilog.co/wiki/Class_Inheritance#References)

------

## 什么是继承?

继承是面向对象编程范式的关键概念。类用来创建用户自定义类型.  继承使得用户可以用非常安全、非侵入的方式对类的行为进行增加或者修改。使用继承可以定义子类型，在子类型中增加新的方法和数据。被继承的类一般称为基类(SystemVerilog中的超类)，得到的新类一般称为引申类（或子类）。

为什么继承如此重要? 因为它使得复用得以实现。让我们通过实例来说明. 假设我们对一个图像模块进行建模. 对其中一部分，我们写了一个代表颜色的类:

```verilog
class Color;
  byte unsigned red;
  byte unsigned green;
  byte unsigned blue;

  function new(byte unsigned red_ = 255,
               byte unsigned green_ = 255,
               byte unsigned blue_ = 255);
    red = red_;
    green = green_;
    blue = blue_;
  endfunction: new

  function mix(Color other); endfunction
  function brighter(float percent); endfunction
  task draw_pixel(int x, int y); endtask

endclass
```

现在, 它的下一个版本希望能够处理部分透明的图像。为此，我们给Color类增加了一个alpha成员,。alpha代表图像的透明度。alpha越大，图像的像素越结实(不透明)。'0'代表完全透明，使得图片的背景全部可见。因此，我们修改color类如下： 

```verilog
class Color;
  byte unsigned red;
  byte unsigned green;
  byte unsigned blue;
  byte unsigned alpha;
  
  function new(byte unsigned red_ = 255,
               byte unsigned green_ = 255,
               byte unsigned blue_ = 255,
               byte unsigned alpha_ = 255);
    red = red_;
    green = green_;
    blue = blue_;
    alpha = alpha_;
  endfunction: new
  
    function mix(Color other); endfunction // new implementation -- would depend on 
                                           // alpha values for both the colors
  function brighter(float percent); endfunction// original implementation good enough
  task draw_pixel(int x, int y);  endtask  // new implementation
        
  // Other functions ...
endclass: Color
```

注意，即使许多代码是由之前版本的Color类复制而来，我们还是需要单独维护两个版本的代码。这时继承就可以发挥作用，使用继承，我们可以简单的从原始的Color类继承出新类，来添加alpha成员。   

```verilog
class ColorWithAlpha extends Color;
  byte unsigned alpha;
  
  function new(byte unsigned red_ = 255,
               byte unsigned green_ = 255,
               byte unsigned blue_ = 255,
               byte unsigned alpha_ = 255);
    super.new(red_, green_, blue_);
    alpha = alpha_;
  endfunction: new
  
  function mix(Color other); endfunction// new implementation -- would depend on 
                                        // alpha values for both the colors
  task draw_pixel(int x, int y);  endtask  // new implementation
      
  // Other functions ...
endclass: ColorWithAlpha
```

这里我们使用关键字 "**extend**" 来创建一个新类ColorWithAlpha. 注意到我们仅仅需要声明新增加的alpha数据成员。其他成员作为超类对象的一部分用来表示原始的Color类。

有C++背景的用户会注意到如何访问原始Color类成员的方式，SystemVerilog和C++很不一样。在C++中，原始类（基类）的数据成员的就像本来也就属于继承类的成员一样；在SystemVerilog中，需要额外一层的间接访问，**super**指定的超类对象和继承类被看作不同的对象。但是，如果需要在继承类中去访问基类成员，SystemVerilog编译器会隐式的帮助完成这部分间接访问。（译者补充：不需要用super去限定，编译器帮忙做这个事情)

```verilog
ColorWithAlpha color1;

color1 = new(127, 127, 127, 255); // solid gray50 

color1.brighter(50);              // make 50% brighter

if (color1.red == 127) begin
  // some code
  // ..
end
```

顺便说一声,  将**super**作为一个不同的对象，也就要求SystemVerilog将super放在不同的分开的存储块（和扩展类的存储块不连续）因此，在编译器层来看，实现C++风格的多重继承就变得更加困难。这也就是为什么提案中的SystemVerilog多重继承实现方式采用了java语言的方法。



## 有什么好处？

正如上面例子所示，继承时我们容易扩展类，使得基类复用。复用致力于能方便写库，库本身就是一系列的复用组件。

在过程式语言编程中，编写库很简单，仅仅需要写一系列的函数，用户只需要直接调用这些函数即可。 如果用户需要某些方面的定制，一般是通过定制变量以及钩子(回调函数)来完成。  面向对象的库不一样，一个面向对象库一般需要提供一系列的类（抽象的或者具体的类)。许多部分可以直接拿来使用，但是更多的情况是，用户需要根据自己需要扩展这些库中的类。继承就提供了一个很重要的扩展方式。

下面给出面向对象程序设计中最重要的原则。

> **根据译者的理解进行补充：**
>
> 回调函数的概念是指预先定义一些需要调用的函数原型，缺省不执行任何动作，用户可以覆盖此函数，通过将调用此函数的函数指针修改，使得在之前的代码不变的情况下，调用函数的内容发生变化，达到定制要求，这个在面向对象中也有用到，比如Callback函数

### **开-关定律**

Dr Bertrand Meyer 由于发明了“开-关”定律而享有盛誉。 此定律被认为是面向对象编程中最重要的原则。简而言之，此定律告诉我们写模块的时候应该使得模块在不做修改的情况下很容易扩展。此定律更正式的表述如下[2]：

> 软件单元（类，模块，函数，等等）应该对扩展开放，但是对修改关闭。(允许扩展，但是拒绝修改)

## 参考资料

1. [↑](http://systemverilog.co/wiki/Class_Inheritance#cite_ref-0) Dave Rich, *DVCon*, 2010, "The Problems with Lack of Multiple Inheritance in SystemVerilog and a Solution", accessed on Friday, August 13 2010 
2. [↑](http://systemverilog.co/wiki/Class_Inheritance#cite_ref-1) Meyer, Bertrand, *Prentice Hall*, 1988, "Object-Oriented Software Construction", accessed on August 16 2010 