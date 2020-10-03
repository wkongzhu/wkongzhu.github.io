---
layout: post
tagline: "Supporting tagline"
category: Digital
tags: [Design]
title: "防毛刺时钟切换电路的设计思想"
---

# 前言

EETIMES上有一篇时钟切换电路的文章，

[non_glitch_mux]: https://www.eetimes.com/techniques-to-make-clock-switching-glitch-free/	"Techniques to make clock switching glitch free"

这篇文章讲述了时钟切换的时候毛刺(glitch)带来的危害，以及如何设计防止毛刺发生的时钟切换电路。但是没有讲到电路设计的构思从何而来，大家看了之后知道直接用这个电路，但是假如不看这篇文章，自己从头设计还是无从下手。

​        在这里，我们换另外一个角度，通过电路设计技巧来阐述防毛刺时钟切换电路的设计思路。希望看过之后，不用参考文章就能够自己设计出这个电路。

# 设计思路

​     对于一个时钟切换电路，输入为两个异步时钟clk0、clk1，以及一个选择信号sel。

(1) 假设不考虑glitch，直接使用Mux就可以完成切频。电路如下：

![](/img/clkmux0.svg)

  由于clk0/clk1/sel之间是异步关系，时钟切换会发生在任意时刻，有一定的概率会发生glitch. glitch的危害文章里已经详述，这里不再重复。

(2)  由于sel和clk0和clk1都是不同步的，我们可以从sel同步的方向入手，假如sel需要和clk0和clk1进行同步，那么sel必须分成两路，一个和clk0同步，一个和clk1同步，同步之后的sel讯号再和clk0/clk1 gating起来，就可以让问题简单化。为了将sel分成两路，并且clk0/clk1需要分别gating,  那么可以将mux逻辑用and/or设计出来，如下：

![clkmux1](/img/clkmux1.svg)

​    当然此Mux电路还可以用两个or加上一个and来实现，都可以。 注意G0和G1两点就是分别对clk0和clk1进行gating.  毛刺的产生原因就是因为这两点变化的时间无法和其控制的时钟源同步。为了解决这个问题，需要思考如何在G0、G1点插入同步DFF。 

(3) 将上面电路拆开成两部分，一部分电路通过sel产生sel0和sel1两路，另一部分电路是gating mux电路, 如下：

[![clkmux2](/img/clkmux2.svg)](http://img8.ph.126.net/fnvfeZfyHlereZHxs_ZS2Q==/1306888316884757658.jpg)   

只需要将sel0接上G0, sel1接上G1就是一个mux电路。将电路分开，是为了后续技巧性的功能替换。 

(4) 将part0电路换成同样功能的带反馈的组合电路(为何要这样做？属于电路设计直觉和技巧)。最常见带反馈的电路是RS触发器，因此可以将part0换成如下电路。

[![select](/img/clkmux3.svg)](http://img4.ph.126.net/vtFcdxQ-r_R5HqppTNlzTQ==/2555511313573227624.jpg) 

(5) 将part0_a或者part0_b替换原来的part0电路，功能不变。如下：

![clkmux4](/img/clkmux4.svg) 

不过，此时插入同步DFF的地方就多了一个选择， 如果直接在G0, G1插入同步DFF, clk0和clk1的gating时间先后顺序不确定，还是有可能发生毛刺。而在s0和s1处插入同步DFF, 正好利用反馈，让时钟切换按照安全的顺序进行：

​              (1) 先gating住之前选择的时钟

​              (2) 然后再放开将要选择的时钟

​              在(1)和(2)之间， 输出时钟一直都是无效状态（对于2and + 1or的mux来说，无效状态就是0)

(6) 按照上面的分析，得到电路如下：

![](/img/clkmux5.svg)

  注意几点： 

​          (1) 对s0插入的DFF需要用clk0作为时钟, 对于s1插入的DFF需要用clk1作为时钟。

​          (2) 后一级的DFF必须使用clock下降沿，因为是用AND门进行gating(如果用上升沿，则更容易出现毛刺)。如果换成2个OR+1个AND的MUX, 则必须用上升沿。

​          (3) 必须插入两级DFF防止metal stable, 前一级可以用上升沿，也可以用下降沿，用上升沿是为了节省时间。

​          (4) 所有的DFF 复位值都是0，即让clk_out处于无效状态。

​          (5) 必须满足先gating后放开的顺序，如果不满足，可以在G0/G1处各插入一个反相器。(用part0_b搭配part1的时候需要插入反相器，如下图)

![image](/img/clkmux6.svg)

​          (6)搭配不同的part0电路和part1电路，经过稍许修改，都可以完成防毛刺切频电路的设计。

