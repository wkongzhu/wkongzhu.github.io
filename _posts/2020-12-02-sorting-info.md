---
layout: post
tagline: "Supporting tagline"
category: ICDesign
tags: [ICDesign]
title: "用信息论分析排序需要多少次比较"
---



用两个示例来阐述信息论在设计中的应用.

# 题目1

输入4个随机数, 要求得到4个数从小到大的排序, 那么两两比较, 需要多少次?

**分析:**   

​			4个数, 随机产生, 那么从小到大的排列可能性共有 $P_4^4 = 4\times3\times2\times1 = 24$种可能性,  不妨设这24中可能性均匀, 则, 从这24种可能性种得到其中的一种, 其熵为:  $24\times1/24\times log_2^{24}>4bit$ , 而一次比较最多能够得到1bit的信息, 因此 要让4个数从小到大排列, 则至少需要5次比较.  实际操作中, 的确可以做到5次比较就可以获得4个数的完全排序.  理论上最少是5次, 因此没有必要也没有可能再做进一步的优化成4次比较.

# 题目2

输入随机4个数, 得到4个数中最大的那个数, 最少需要几次两两比较?

**分析:** 

​			4个随机数中, 最终挑选出1个值,  其熵为 $4\times1/4\times log_2^4=2$bit,  一次比较最多得到1bit信息, 因此理论上如果每次比较得到1bit信息, 那么只需要比较2次就可以得到.  但是, 限制条件为两两比较, 第一次进行两两比较, 只能排除其中1个, 因此排除的个数小于总数的一半, 也就是2个, 因此排除的信息量小于1bit, 无法做到一次比较得到1bit信息.  而每一次比较最多可以获得1bit信息, 所以,  总共需要大于2次, 至少3次比较才能确定找到最大值.  实际推演中, 每次比较淘汰一个, 一共淘汰3个数, 进行了3次比较, 得到最大值.

   下面, 我们来计算第一次比较获得了多少信息.

​	为了将问题简化, 我们假定有4个数, 1,2,3,4,  通过随机洗牌, 分配给A,B,C,D. 随机分配, 每个字母对应每个数字的概率都是相等的. 因此, A, B,C,D为最大的数的概率完全相等, 

* 定义事件Y的样本空间={A为最大值, B为最大值, C为最大值, D为最大值};  
        * 不妨设第一次比较的数为A和B,  定义这个事件X的样本空间={A>B,  A<B}. 
* 事件X发生之后, 消除了部分Y的确定性, 也就是判断谁是最大值容易了一些. 具体容易了多少, 也就是被消除的确定性, 被定义为互信息$I(X, Y) = H(Y) - H(Y\vert X)$

对于1,2,3,4四个数进行全排列, 共有4!=24种可能性, 这24种排列的可能性是完全相等的,  各占1/24.   可以得到A,B,C,D成为最大值4的可能性是都是1/4.

$H(Y) = 1/4 \times 4 \times log(4) = 2$

$H(Y\vert X)$ 的计算可以用如下图表示(由于对称性, 只画出上半部的情况, P(A>B)和P(A<B)的情况完全对称):

![I(X,Y)](/img/I(X,Y).svg)

$$
H(Y|X) = 2*P(A>B)(P(A最大|A>B)log(P(A最大|A>B)) + P(A最大|A>B)log(P(A最大|A>B)) \\
 + P(A最大|A>B)log(P(A最大|A>B)) + P(A最大|A>B)log(P(A最大|A>B))) \\
 = 6/12*log(1/(6/12)) + 0 + 2*3/12*log(1/(3/12)) = 1/2 + 1 = 1.5bit
$$


所以, $I(X,Y) = H(Y) - H(Y\vert X) = 2 - 1.5 = 0.5bit$,  也就是, 做第一次比较(X事件的发生)给Y提供的信息是0.5bit, 而不是1bit.

