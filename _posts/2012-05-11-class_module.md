---
layout: post
tagline: "Supporting tagline"
category: verification
tags: [systemverilog]
title: "用class封装verilog验证module"
---

UVM已经开始流行，对于新的项目，可以考虑用UVM进行全新的验证环境开发。 

但是，实际情况，项目是具有延续性的，好多以前的验证环境如今还是要继续使用。于是，出现一个问题：在升级的项目中，我想使用UVM，但是以前的bus behavior model 是用verilog  module写的。而使用uvm的编程风格，我们希望将某个协议相关的一系列class封装到一个package中，便于将来复用.  

这时候，问题出现了，在很多仿真器中，并不支持在package中进行直接层次调用静态module的函数。如下程序： 

```verilog
package tst;

class a;
  task try();
    $display("<In Class>");
    top.try();
  endtask:try
endclass:a

endpackage:tst

module top;
  import tst::*;

  task try();
    $display("<In Module> : module task");
  endtask

  initial begin
    a inst = new;
    inst.try();
    $finish;
  end

endmodule:top
```

以上这个程序在很多编译器中会报错误不通过；即使通过，这种层次调用对于package的封装和模块化也是一种不小的破坏。 

为了解决这个问题，可以通过在top层并列申明一个继承类。package中的基类相应的task定义为虚函数。改进之后如下： 

```verilog
package tst;

class a;
  virtual task try();
    $display("<In BASE Class>");
  endtask:try
endclass:a

endpackage:tst
///////////////////////////////////////////////////////
  import tst::*;
class ext extends a;
  task try();
    top.try();
  endtask:try
endclass:ext

module top;
  task try();
    $display("<In Module> : module task");
  endtask

  initial begin
    a inst;
    ext ext_inst = new();
    inst = ext_inst;
    inst.try();
    $finish;
  end

endmodule:top
```

说明： 

1. 为了兼容更多的仿真器，同时让package更加通用化，方便维护复用；我们遵循一个原则：在package中禁止直接调用静态层次module的相关内容。 
2. 在package的behavior driver class中，申明想调用的虚函数。便于重载。 
3. 在module静态环境中，$root空间，声明继承类，重载相关的behavior虚函数。在重载的函数中，通过静态层次module调用来使用相关的module task/function等。 
4. 在验证环境中，将重载类的对象赋值给基类指针。可以很方便使用基类指针完成层次函数调用。 