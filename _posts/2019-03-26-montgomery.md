---
layout: post
tagline: "Supporting tagline"
category: 算法
tags: [montgomery, math]
title: "Montomery算法"
---


<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#sec-1">1. 蒙哥马利预备知识</a>
<ul>
<li><a href="#sec-1-1">1.1. 蒙哥马利约减</a></li>
<li><a href="#sec-1-2">1.2. 蒙哥马利乘模</a></li>
<li><a href="#sec-1-3">1.3. 蒙哥马利幂模</a></li>
</ul>
</li>
</ul>
</div>
</div>

# 1 综述

这篇文章为大家梳理一下整个蒙哥马利算法的本质，蒙哥马利算法并不是一个独立的算法，而是三个相互独立又相互联系的算法集合，其中包括

1.  蒙哥马利乘模，是用来计算  $x\cdot y\ (mod\ N)$​ 
2.  蒙哥马利约减，是用来计算 $t\cdot \rho^{-1}\ (mod\ N)​$
3.  蒙哥马利幂模，是用来计算 $x^y\ (mod\ N)​$

其中蒙哥马利幂乘是RSA加密算法的核心部分。

# 2 基本概念

梳理几个概念，试想一个集合是整数模N之后得到的
$Z_N=\left\{0,1,2,\cdots,N-1\right\}$

注：N在base-b进制下有 $l_N​$ 位。 比如10进制和100进制，都属于 $base-10​$ 进制，因为$100=10^2​$, 所以b=10。在10进制下，667的 $l_N=3​$

这样的集合叫做N的剩余类环，任何属于这个集合$Z_N​$的$x​$满足以下两个条件：

1.  正整数
2.  最大长度是 $l_N​$

这篇文章中讲到的蒙哥马利算法就是用来计算基于$Z_N$集合上的运算, 简单讲一下原因，因为RSA是基于大数运算的，通常是1024bit或2048bit，而我们的计算机不可能存储完整的大整数，因为占空间太大，而且也没必要。因此，这种基于大数运算的加密体系在计算的时候都是基于 $Z_N$ 集合的，自然，蒙哥马利算法也是基于 $Z_N$.

在剩余类环上，有两种重要的运算，一类是简单运算，也就是加法和减法，另一类复杂运算，也就是乘法。我们比较熟悉的是自然数集上的运算，下面看下怎么从自然数集的运算演变成剩余类环上的运算。

- 对于加法运算，如果计算 $x\pm y\ (mod\ N), (0\leqslant x,y \lt N)$
    试想自然数集上的 $x\pm y$,
    $$\qquad 0\leqslant x+y\leqslant 2\cdot(N-1)​$$

    $$-(N-1)\leqslant x-y\leqslant (N-1)$$

    我们可以简单的通过加减N来实现从自然数到剩余类集的转换.

- 另外一类是乘法操作，也就是 $x\cdot y\ (mod\ N), (0\leqslant x,y\lt N)​$, 那么

    $0\leqslant x\cdot y\leqslant (N-1)^2​$

    如果在自然数集下，令 $t=x\cdot y​$, 那么对于 $\mod N​$ 我们需要计算
    $$t-（N\cdot \lfloor\frac{t}{N}\rfloor）​$$

    加减操作很简单，具体的算这里就不细说了，我们用 $Z_N-ADD$ 来代表剩余类环上的加法操作。
    既然我们可以做加法操作，那么我们就可以扩展到乘法操作，算法如下

    ![img](../img/r01.jpg)

    ```python
    def Zn_Mul(x, y):
        ''' 0<= x, y< N, 为二进制表示的整数环 
        return t = xy mod N
        '''
        t = 0
        for i in reversed(range(len(bin(y))-2)): # len(bin) -2 等于y的二进制数的总位数
            t = (t + t) % N 
            yi = (y>>i) & 1 # yi代表y的二进制的第i位
            if yi : t = (t + x) % N
        return t    
    ```

    

    但是这并不是一个好的解决方案，因为通常来说，我们不会直接做w位乘w位的操作，这个后面会用蒙哥马利的乘法来代替解决。

对于取模操作，一般有以下几种方法

1. 根据以下公式，来计算取模操作
    $t-（N\cdot \lfloor\frac{t}{N}\rfloor）$
    这种解法有以下特征：

    整个计算过程是基于标准的数字表示，不需要预计算（也就是提前计算一些变量，以备使用）
    涉及到一个除法操作，非常费时和复杂。
2. 用Barrett reduction算法，这篇文章不细说，但是有以下特征：
    基于标准的数字表示，不需要预计算，需要 $2 \cdot (l_N+1) \cdot (l_N+1)$ 次数乘运算
3. 用蒙哥马利约减，也就是下面要讲的算法，有以下特征：
    不是基于标准的数字表示（后文中有提到，是基于蒙哥马利表示法）
    需要预计算, 共$2 \cdot (l_N) \cdot (l_N)$ 次数乘运算

# 3 蒙哥马利预备知识

在将蒙哥马利算法之前，先看一下在自然数下的乘法公式

计算 $x\cdot y$, 想象一下我们常用的计算乘法的方法，用乘数的每一位乘上被乘数，然后把得到的结果相加，总结成公式，可以写成如下的形式。

$x\cdot y=x\cdot \sum_{i=0}^{l_y-1}y_i \cdot b^i$

$\qquad=\sum_{i=0}^{l_y-1}y_i \cdot x \cdot b^i$

尝试下面一个例子，10进制下（也就是b=10），y=456（也就是 $l_n=3$ ）,计算 $x\cdot y$, 公式可演变如下：

$$x\cdot y=(y_{0}\cdot x\cdot 10^{0})+(y_{1}\cdot x\cdot 10^{1})+(y_{2}\cdot x\cdot 10^{2})$$
$$\qquad=(y_{0}\cdot x\cdot 0)+(y_{1}\cdot x\cdot 10)+(y_{2}\cdot x\cdot 100)$$
$$\qquad=(y_{0}\cdot x)+10\cdot(y_{1}\cdot x+10\cdot(y_{2}\cdot x\cdot +10\cdot(0)))$$

最后一次演变其实就是用霍纳法则(**Horner’s rule**)所讲的规则，关于霍纳法则，可以自行百度。

这个计算过程写成代码实现的算法应该是这样的：

![img](../img/r02.jpg)

```python
def Zn_Mul10(x, y):
    ''' 0<= x, y< N, 为十进制表示的整数环 
    return t = xy mod N
    '''
    t = 0
    for i in reversed(range(len(str(x)))): # len(str) 等于y的十进制数的总位数
		yi = int(str(x)[i]) # yi代表y的十进制的第i位
        t = (t * 10) % N 
        t = (t + yi*x) % N
        #以上两步合并为： t = (t*10 + yi*x) % N 
    return t   
```



接下来我们来看下面这样的计算，计算 $(x\cdot y)/1000​$, 由前面可以知道
$$x\cdot y=(y_{0}\cdot x)+10\cdot(y_{1}\cdot x+10\cdot(y_{2}\cdot x\cdot +10\cdot(0)))​$$

由此可以知道：

$$\frac{x\cdot y}{1000}=\frac{(y_{0}\cdot x\cdot 10^{0})+(y_{1}\cdot x\cdot 10^{1})+(y_{2}\cdot x\cdot 10^{2})}{1000}​$$

$$\qquad=\frac{(y_{0}\cdot x\cdot 0)+(y_{1}\cdot x\cdot 10)+(y_{2}\cdot x\cdot 100)}{1000}$$

$$\qquad=\frac{(y_{0}\cdot x)}{1000}+\frac{(y_{1}\cdot x)}{100}+\frac{(y_{2}\cdot x)}{10}$$

$$\qquad=(((((y_0\cdot x)/10)+y_1\cdot x)/10)+y_2\cdot x)/10​$$

这个计算过程写成代码实现的算法是这样的：

![img](../img/r03.jpg)

```python
def Zn_Mul_DIV(x, y):
    ''' 0<= x, y< N, 为十进制表示的整数环 
    return t = xy/(10^lN) mod N, lN为N的十进制总位数
    '''
    t = 0
    for i in range(len(str(N))): # len(str) 等于十进制数的总位数
		yi = int(str(y)[i]) if i < len(str(y)) else 0 # yi代表y的十进制的第i位
        t = (t + yi*x)/10 % N 
    return t   
```

接下来我们再来看在剩余类集合下的乘法操作 $x\cdot y/1000\ (mod\ 667)​$

我们知道剩余类集合 $Z_{667}=\left\{0,1 \cdots 666\right\}​$, 是不存在小数的, 而如果我们采用自然数集的计算方式的话,
就会出现小数, 比如前面的例子, 除10就会有小数。

这个问题是这样的, 我们知道 $u·667 \equiv 0\ (mod\ 667)$ （$\equiv$ 表示取模相等）,
所以我们可以选择一个合适的 $u$, 用 $u$ 乘667再加上 $r$, 使得和是一个可以除10没有小数, 这样在 $mod\ 667$ 之后依然是正确的结果。至于 $u$ 怎么算出来, 这篇文章会在后面的章节说明。

这个过程之后 $x\cdot y/1000\ (mod\ 667)$ 的计算算法可以写成如下的形式($N=667$)

![img](../img/r04.jpg)

```python
def Zn_Mul_DIV2(x, y):
    ''' 0<= x, y< 667, 为十进制表示的整数环 
    return t = xy/1000 mod 667
    '''
    t = 0
    for i in range(len(str(667))): # len(str) 等于十进制数的总位数
		yi = int(str(y)[i]) if i < len(str(y)) else 0 # yi代表y的十进制的第i位
        t = (t + yi*x +u*N)/10 % N 
    return t   
```

至此, 你可能还不明白上面说这一堆演变的原因, 其实很简单, 原来是一个 $(x\cdot y)\ (mod\ 667)​$ 的运算,
这个运算中的模操作, 正常情况下是要通过除法实现的, 而除法是一个特别复杂的运算, 要涉及到很多乘法,
所以在大数运算时, 我们要尽量避免除法的出现。而通过以上几个步骤, 我们发现 $(x\cdot y)/1000\ (mod\ 667)​$,
这个操作是不用除法的。等等, 算法中明明有个除10的操作, 你骗谁呢。不知道你有没有发现,
除数其实是我们的进制数, 除进制数在计算机中是怎么做呢, 其实很简单, 左移操作就ok了。所以这个计算方法是不涉及到除法操作的。

但是我们要计算的明明是 $(x_1\cdot y_1)\ (mod\ 667)$, 怎么现在变成了 $(x_2\cdot y_2)/1000\ (mod\ 667)$,
所以在下一步, 我们要思考的是怎么样让 $(x_1\cdot y_1)\ (mod\ 667)$ 转变成 $(x_2\cdot y_2)/1000\ (mod\ 667)$ 这种形式。

考虑这样两个算法

-   第一个是输入 $x​$ 和 $y​$, 计算 $x \cdot y \cdot \rho^{-1}​$
-   第二个算法, 输入一个 $t$, 计算 $t \cdot \rho^{-1}$
       $$x\cdot y\ (mod\ 667)=((x\cdot1000)\cdot(y\cdot1000)/1000)/1000\ (mod\ 667)$$

是不是变成了我们需要的 $(x\cdot y)/1000\ (mod\ 667)​$ 模式, 而且这个转变过程是不是可以通过上面两个算法来实现,
输入值如果是 $(x\cdot1000)​$ 和 $(y\cdot1000)​$, 则通过第一个算法可以得到 $((x\cdot1000)\cdot(y\cdot1000)/1000)​$,
把结果作为第二个算法的输入值, 则通过第二个算法可以得到 $((x\cdot1000)\cdot(y\cdot1000)/1000)/1000​$.

扯了一大顿, 终于引出了今天文章的主角, 前面讲到的两个算法, 第一个就是蒙哥马利乘模, 第二个就是蒙哥马利约减。
下面我们来讲这两个算法的详解。

正如前面提到的蒙哥马利算法的三个特性之一, 不是基于普通的整数表示法, 而是基于蒙哥马利表示法。
什么是蒙哥马利表示法呢, 其实也很简单, 上面我们提到,
要让 $(x_1\cdot y_1)\ (mod\ 667)$ 转变成 $(x_2\cdot y_2)/1000\ (mod\ 667)$ 这种形式,
我们需要将输入参数变成 $(x\cdot1000)$ 和 $(y\cdot1000)$, 而不是x和y本身,
而 $(x\cdot1000)$ 和 $(y\cdot1000)​$ 其实就是蒙哥马利表示法。

所以我们先定义几个概念：

-   **蒙哥马利参数:** 给定一个 $N$, $N$ 在b进制（例如, 二进制时, b=2）下共有 $l$ 位, $gcd(N,b)=1$,
    先预计算以下几个值(这就是前面提到的特性之一, 需要预计算）：
    $\rho = b^k$ 指定一个最小的 $k$, 使得 $b^k>N$. 也就是$N$在$b$进制下有$k$位，$k=l_N$
    $\omega = -N^{-1} (mod\ \rho)$

    这两个参数是做什么用的呢, 你对照前面的演变过程可以猜到 $\rho​$ 就是前面演变中的 $1000​$, 而 $\omega​$
    则是用来计算前面提到的 $u​$ 的。

-   **蒙哥马利表示法:** 对于 $x​$, $0\leqslant x\leqslant N-1​$, $x​$ 的蒙哥马利表示法表示为 $x=x\cdot \rho\ (mod\ N)​$

# 4 蒙哥马利约减

蒙哥马利约减的定义如下:

给定一些整数 $t​$, 蒙哥马利约减的计算结果是 $t\cdot \rho^{-1}\ (mod\ N)​$

蒙哥马利约减的算法可表示为

![img](../img/r05.jpg)

```python
# 已知 b, lN, w=N^-1, N
def mont_reduce(t):
    'return r=t*p^-1 (mode N), base-b'
    r = t
    for i in range(lN):
        u = ri * w (mod b)
        r = r + u*N*(b**i)
    r = r/b**lN
    if r >= N : r -= N
    return r
```



蒙哥马利约减可以算作是下面要说的蒙哥马利模乘当 $x=1​$ 时的一种特殊形式,
同时它又是蒙哥马利乘模要用到的一部分, 这在下一部分讲蒙哥马利乘模的时候有讲到。

蒙哥马利约减可以用来计算某个值得取模操作, 比如我们要计算 $m(mod\ N)$, 只需要将 $m$
的蒙哥马利表示法 $m\cdot \rho$ 作为参数, 带入蒙哥马利约减, 则计算结果就是 $m(mod\ N)$.

# 5 蒙哥马利乘模

一个蒙哥马利乘模包括整数乘法和蒙哥马利约减, 现在我们有蒙哥马利表示法：

$$\hat{x}=x\cdot\rho\ (mod\ N)$$
$$\hat{y}=y\cdot\rho\ (mod\ N)$$

它们相乘的结果是

$$t=\hat{x}\cdot\hat{y}$$
$$\ =(x\cdot\rho)\cdot(y\cdot\rho)$$
$$\ =(x\cdot y)\cdot\rho^2$$

最后, 用一次蒙哥马利约减得到结果

$$\hat{t}=(x \cdot y) \cdot \rho\ (mod\ N)​$$

上面我们可以看出, 给出的输入参数是 $\hat{x}​$ 和 $\hat{y}​$,  得到的结果是 $(x \cdot y) \cdot \rho\ (mod\ N)​$,
所以蒙哥马利乘法也可以写成如下形式, 已知输入参数x和y, 蒙哥马利乘法是计算 $(x \cdot y) \cdot \rho ^ {-1}\ (mod\ N)​$

举个例子：
b=10, 也就是说在10进制下, N=667
让 $b^k>N​$ 的最小的k是3, 所以 $\rho=b^k=10^3=1000​$
$$\omega=-N^{-1}\ (mod\ \rho)=-667^{-1}\ (mod\ \rho)=997​$$

因为 $x=421​$, 所以 $\hat{x}=x\cdot\rho(mod\ N)=421\cdot1000(mod\ 667)=123​$

因为 $y=422$, 所以 $\hat{y}=y\cdot\rho(mod\ N)=422\cdot1000(mod\ 667)=456$

所以计算 $\hat{x}$ 和 $\hat{y}$ 蒙哥马利乘结果是

$$\hat{x}\cdot\hat{y}\cdot\rho^{-1}=(421\cdot1000\cdot422\cdot1000)\cdot1000^{-1}\ (mod\ 667)​$$
$$\qquad\qquad(421\cdot422)\cdot1000\ (mod\ 667)​$$
$$\qquad\qquad(240)\cdot1000\ (mod\ 667)​$$
$$\qquad\qquad547\ (mod\ 667)​$$

然后总结一下蒙哥马利约减和蒙哥马利乘法的伪代码实现, 这个算法其实就是从蒙哥马利预备知识讲到的算法演变来的。

![img](../img/r06.jpg)

```python
# 已知 b, lN, w=N^-1, N
def mont_mul(t):
    'return r=xy*p^-1 (mode N), 0<=x,y<N, in base-b'
    r = 0
    for i in range(lN):
        u = (r0+yi*x0) * w (mod b)
        r = (r + yi*x + u*N)/b
    if r >= N : r -= N
    return r
```



上面的例子用这个算法可以描述为

![img](../img/r07.jpg)

蒙哥马利算法是一套很完美的算法, 为什么这么说呢, 你看一开始已知 $x​$, 我们要求 $\hat{x}=x \cdot \rho​$,
这个过程可以通过蒙哥马利乘法本身来计算, 输入参数 $x​$ 和 $\rho^2​$, 计算结果就是 $\hat{x}=x \cdot \rho​$.
然后在最后, 我们知道 $\hat{x}=x \cdot \rho​$, 要求得 $x​$ 的时候,
同样可以通过蒙哥马利算法本身计算, 输入参数 $\hat{x}​$ 和 $1​$, 计算结果就是 $x​$.
有没有一种因就是果, 果就是因的感觉, 这就是为什么说蒙哥马利算法是一套很完美的算法。

# 6 蒙哥马利幂模

最后, 才说到我们最开始提到的RSA的核心幂模运算, 先来看一下普通幂运算的算法是怎么得出来的。

以下资料来自于百度百科快速模幂运算
针对快速模幂运算这一课题, 西方现代数学家提出了大量的解决方案, 通常都是先将幂模运算转化为乘模运算。
例如求 $D=C^{15}\ \mod\ N$
由于：$(a*b)\ mod\ n = (a\ mod\ n) * (b\ mod\ n)\ mod\ n$

所以令：
$$
C1 =C*C \ mod\  N =C^2 \ mod\  N   \\
C2 =C1*C \ mod\  N =C^3 \ mod\  N  \\
C3 =C2*C2 \ mod\  N =C^6 \ mod\  N \\
C4 =C3*C \ mod\  N =C^7 \ mod\  N  \\
C5 =C4*C4 \ mod\  N =C^{14} \ mod\  N \\
C6 =C5*C \ mod\  N =C^{15} \ mod\  N  \\
$$

即：对于E=15的幂模运算可分解为6个乘模运算, 归纳分析以上方法可以发现：

对于任意指数 $E​$, 都可采用以下算法计算 $D=C^E\ mod\ N​$

```python
1  D=1
2  while E>0 :
3     if E%2 == 0 :
4         C = C*C % N
5         E = E/2
6     else:
7         D = D*C % N :
8         E = E-1
9  return D
```

继续分析会发现, 要知道 E 何时能整除2, 并不需要反复进行减一或除二的操作, 只需验证 E 的二进制各位是0还是1就可以了,
从左至右或从右至左验证都可以, 从左至右会更简洁,

设 $E=\sum_{i=0}^{n} E_i*2^i, E_i=0,1​$

```python
1  D = 1
2  for i in reversed(range(n+1)): # n, n-1, n-2, ..., 0
3      D = D*D % N
4      if Ei == 1 :
5         D = D*C % N
6  return D
```

这样, 模幂运算就转化成了一系列的模乘运算。

算法可以写成如下的形式

![img](../img/r08.jpg)

如果我们现在用蒙哥马利样式稍作改变, 就可以变成如下的形式：

![img](../img/r09.jpg)

```python
def mont_exp(x, y):
    'r = x**y (mod N)'
    t = mont_mul(1, p**2)
    x_ = mont_mul(x, p**2)
    for i in reversed(range(lN)):
        t = mont_mul(t,t)
        if yi == 1 :
            t = mont_mul(t, x_)
    return mont_mul(t, 1)
```



以上就是蒙哥马利算法的全部, 通过蒙哥马利算法中的约减运算, 我们将大数运算中的模运算变成了移位操作, 极大地提高了大数模乘的效率。

但是在以上的算法, 可以发现还有两个变量的计算方式不是很清楚, 一个是 $\omega$, 前面说过 $\omega = -N^{-1} (mod N)$,
其实在算法中, 我们看到, $\omega$ 仅仅被用来做 $\mod b$ 操作, 所以事实上, 我们只需要计算 $\mod b$ 即可.

尽管N有可能是合数（因为两个素数的乘积不一定是素数）, 但通常N和 $\rho$ （也就是N和b）是互质的,
也就是说 $N^{\phi(b)}=1(mob\ b)$ (费马定理), $N^{\phi(b)-1}=N^{-1}(mod\ b)$
因为 $b=2^\omega$, 所以 $\phi(b)=2^{(\omega-1)}$, 写成算法是这样的

![img](../img/r10.jpg)

还有一个参数是 $\rho^2$, 还记得前面说过 $\rho$ 是怎么得出来吗, 选定一个最小的 $k$, 使得 $b^k>N$,
我们还知道 $N$ 在 $b$ 进制下是 $l_N$ 位, 所以当 $k=l_N$ 的时候肯定是符合要求。

$b=2^{\omega}$ 所以$\rho=b^k=({2^{\omega}})^k​$

$\rho^2={({2^w})^k)}^2=2^{2\cdot k\cdot \omega}=2^{2\cdot l_N\cdot \omega}$, 算法如下

![img](../img/r11.jpg)

至此整个蒙哥马利算法就全部说完了。通过这个算法, 我们可以实现快速幂模。