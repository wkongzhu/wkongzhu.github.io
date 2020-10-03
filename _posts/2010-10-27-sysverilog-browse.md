---
layout: post
tagline: "Supporting tagline"
category: Digital
tags: [ICDesign]
title: "systemverilog语法浏览"
---

# Systemverilog中的类和对象

来源：http://systemverilog.co/wiki/Objects_and_Classes（可惜此来源已经不复存在)

### 类是Systemverilog中许多关键概念的核心。 

1. 和其他面向对象的语言相似，类是面向对象的关键
2. 类对象通过引用传递，而其他Systemverilog类型通过值传递。 
3. 类是systemverilog随机化的关键。 
4. 约束(constraint)必须被申明为类成员。

### Systemverilog中的对象

​    在很多方面，Systemverilog中的类和对象的概念像Java胜过像C++. 

​    如同Java一样, Systemverilog实现了一个垃圾回收器，用户负责创建对象，只要此对象的所有引用都没有了，此对象就被标记销毁。  这个特点很方便，用户不需要关心内存泄漏。尽管如此，用户仍然需要保证对对象的引用尽量减少，特别要关心容器类对象；用户需要保证没有用的数据常常被清理掉。 

​     Systemverilog仅仅将类的实例视为对象。对象总是映射到堆中的一段内存（使用new()构造函数），而语言内建数据类型(以及struct/union/enum等)不会被映射到堆上。不允许对内建数据类型使用new操作，而是根据其申明的方式来决定到底映射到栈中还是静态内存区域。

​    当申明一个对象（类的实例）的时候，仅仅创建了一个类的引用【reference】，此引用被初始化为null(空引用)。  引用和C/C++中的指针相似。Systemveilog中的关键字null和C/C++中的空指针NULL相似。在Systemverilog中，对象通过引用和变量进行绑定(在语言参考手册中也被称为句柄【handle】). 变量不能够通过值来绑定对象。 

   除了null值之外，类实例的变量可以赋值为其他相同类型对象的句柄，或者赋值为一个从构造函数new()返回的全新句柄。 

   在Systemverilog中，当对象作为函数的参数，总是以引用的方式进行传递；另一方面，内建类型总是以值进行参数传递的。同样的，函数总是以引用来返回对象，但是总是通过值来返回内建数据类型。 

   当一个对象赋值给其他变量，或者作为函数参数传递，或者被放入一个容器，则对象引用计数加1。当对象的变量离开了作用域，则引用计数减1。如果没有任何变量绑定对象的句柄，那么此对象就被垃圾收集器标记。 

​     对象通过引用传递这个特点在两个方面特别重要：首先，类被用来创建复杂的数据结构，将会比原始内建数据类型消耗更多的内存资源，因此，通过引用传递对象在运行时将更有效率。更重要的是，通过引用传递对象是面向对象设计的范式.一个类对象常常会在其申明的作用域之外还保持生命，保持自生的状态；因此，即使通过函数传递，也需要保持一致性。因为对象是通过引用传递的(并不是通过复制方式)，所以他们在不同的函数作用域中都保持了一致性(补充：指向同一块memory)。 

### 类的创建

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

  

​     使用类来进行行为级建模以及编写验证组件。类结构不能被合成，因此不能被用来进行寄存器传输级设计。 

   在验证中，类被用来对所有东西建模，从简单的交易【transaction】，到复杂的产生器【generator】，驱动器【driver】，观察器【monitor】，记分牌【soreboard】。即使通讯通道【channel】也使用类来建模。 

​    很多时候设计类，是通过其公共方法和客户进行交互。类的数据成员代表对象的状态，用户不能通过直接操作修改对象的状态。通常在面向对象设计中，数据成员被申明为私有(使用local关键字)或者保护(protected关键字)。此原则在systemverilog中并不是总是适用，在Systemverilog中，通常将可以随机化的数据成员以及其对应的约束申明为公共(public)。

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

| 系统任务                    | 任务说明                                                     |
| --------------------------- | ------------------------------------------------------------ |
| \$sscanf(str,format,args);  | \$sscanf 将字符串按照某个模板格式进行扫描，其字符串格式和C语言中的printf()函数类似 |
| \$sformat(str,format,args); | \$sformat是\$sscanf的反函数。将字符串按照给定的格式填入相应的参数args中 |
| \$display(format,args);     | \$display就是Verilog的printf语句，在stdout上显示格式化的字符串 |
| \$sformatf(format,args);    | $sformatf任务和\$sformat相似，除了其返回字符串结果。字符串作为\$sformatf的返回值，而不是像\$sformat一样放在第一个参数上。 |

### **注意**

- 一个流行的\$sformatf的替代方式是\$psprintf. 它实际上是由Vera遗留下来的。$sformtf  在后来成为了SystemVerilog的语言标准。然而，大部分流行的SystemVerilog编译器都支持\$psprintf, 尽管它没有成为标准。如果想符合标准，建议使用\$sformatf. 

### **参考**

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
...
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

**参考文献**

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

**队列的声明和使用**

​     SystemVerilog中的队列声明是通过在标识符加后缀[$]. 例如，声明一个unsigned int类型的队列，使用如下方式：

| 1    | int unsigned intQ[$]; |
| ---- | --------------------- |
|      |                       |

​         缺省时，队列没有任何成员。正如数组一样，队列可以在声明的时候被初始化为一些成员. 这些用来初始化队列的成员可以是文字型数组，也可以来自另外一个已经存在的数组:

| 1  2  3  4  5  6  7 | // Initializing a Queue  int unsigned intQ[$] = {1, 2, 3, 5, 8,  13};     // An already existing array may be used  to  // initialize a queue  int unsigned intDA[] = {1, 2, 3, 5, 8,  13};  int unsigned intQalt[$] = intDA; |
| ------------------- | ------------------------------------------------------------ |
|                     |                                                              |

​    队列声明可以限制长度。限制长度的队列成员将不会超过最大长度. 如果你只需要一个小队列，并且确认不会超过某个范围，那么最好声明成限制长度的队列。特别是队列本身是类的一部分。如果没有声明为限制长度，那么队列至少会消耗一整个内存块(如上描述的架构). 一个内存块一般占用1K甚至更多(由编译器实现决定). 为了让队列消耗更少的内存，队列声明为限制长度的：

| 1  2 | // A queue that can grow only up to a  size of 32  int unsigned intBQ[$:32]; |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

​     一旦队列被声明，其成员可以像普通的数组一样被访问。实际上，SystemVerilog手册认为队列是通用未打包的数组的一种[[3\]](http://systemverilog.co/wiki/Queues#cite_note-2)[[4\]](http://systemverilog.co/wiki/Queues#cite_note-3). 类似其他数组，队列通过值传递。如果需要传递队列的引用，需要将参数声明为ref.

队列可以和其他队列以及其他类型的数组直接进行复制[[5\]](http://systemverilog.co/wiki/Queues#cite_note-4).

 

**队列的方法和操作**

 

​    除了所有的数组都支持的索引和片选操作，SystemVerilog队列还支持许多其他操作：

| **队列的方法和操作**  |                                                              |      |      |      |      |      |      |      |      |
| --------------------- | ------------------------------------------------------------ | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| **方法****/****操作** | **功能描述**                                                 |      |      |      |      |      |      |      |      |
| q[a]                  | 索引操作——返回对应索引的成员。如果超出范围，**则返回此成员类型的缺省值。** |      |      |      |      |      |      |      |      |
| q[a:b]                | 片选操作——返回指定范围的队列成员。如果a为负数，则从0开始，如果b大于最后成员的索引，则截至到最后一个成员。 |      |      |      |      |      |      |      |      |
| q.size()              | 返回队列中成员的个数                                         |      |      |      |      |      |      |      |      |
| q.insert(index,item)  | 指定索引处插入item. 第1个参数index必须是整数，第2个参数item必须和队列成员类型一致。如果队列已经到达设定的大小(队列满)，则最后一个成员被隐式的删除 |      |      |      |      |      |      |      |      |
| q.delete(index)       | 从队列中删除对应索引的成员                                   |      |      |      |      |      |      |      |      |
| q.delete()            | delete方法被重载。当没有索引参数的时候，整个队列被清除，等价于: q = {}. |      |      |      |      |      |      |      |      |
| q.push_back(item)     | 将item增加到队列尾部，如果队列满了，操作将失败               |      |      |      |      |      |      |      |      |
| q.pop_back()          | 删除并返回队列的最后一个成员                                 |      |      |      |      |      |      |      |      |
| q.push_front(item)    | 将item加到队列前面。如果队列已经满了，**则最后一个成员将被隐式的丢掉** |      |      |      |      |      |      |      |      |
| q.pop_front()         | 删除并返回队列的第一个成员                                   |      |      |      |      |      |      |      |      |

 

 **参考文献** 

1. [↑](http://systemverilog.co/wiki/Queues#cite_ref-0) *Wikipedia*, "[Double-ended queue](http://en.wikipedia.org/wiki/Double-ended_queue)", accessed on August     25 2010 
2. [↑](http://systemverilog.co/wiki/Queues#cite_ref-1) Nicolai M. Josuttis, *Addison Wesley*,     1999, "[Deque, Section 6.3 of The C++ Standard Library](http://books.google.com/books?id=n9VEG2Gp5pkC)",     accessed on September 02 2010 
3. [↑](http://systemverilog.co/wiki/Queues#cite_ref-2) *IEEE 1800-09*, Section 7.4 
4. [↑](http://systemverilog.co/wiki/Queues#cite_ref-3) *SVUG*,     "[SV language query related to dynamic array](http://svug.org/Forum/tabid/57/view/topic/postid/997/Default.aspx)",     accessed on September 02 2010 
5. [↑](http://systemverilog.co/wiki/Queues#cite_ref-4)     Puneet, *Coverification Blog*, 2009, "[Copying arrays and queues in SystemVerilog](http://coverification.com/2009/07/24/copying-arrays-and-queues-in-systemverilog/)",     accessed on September 02 2010 

 