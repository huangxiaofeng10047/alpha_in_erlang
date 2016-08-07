# 参加邓辉解构Alpha Go培训班

>Implementing *"Mastering the game of Go with deep neural networks and tree search"* in Erlang

## 从对弈谈起

谈起AI与人类对弈的过往，对于AI如何战胜人类，大多数人给出的答案基本上利用计算机（AI）的运算能力，穷举可能的结果，并从可能
的结果优选最佳路径；对于其他棋类，这样的做法可能可行，但是在面对围棋时，计算机的运算能力受到了挑战，围棋棋盘上的空间可能
性超过现有计算机的能力极限，因此仍然采取早期内嵌专家系统的方法处理对弈，如[Deep Blue](https://en.wikipedia.org/wiki/Deep_Blue_(chess_computer))，
那么其结果可想而知——失败；AlphaGo选择了不同的方式，其原理简单地来说就是在有限的时间内，通过大量模拟获得落子位置获胜的
数学统计，进而逼近最优落子位置，实现在不依赖围棋定势基础上（专家系统）打败人类围棋选手。当然AlphaGo的真正实现细节不会如
此简单，但大体上通过Monte Carlo Tree Search配合神经网络是可以确定的。因此培训的核心内容便是理解与实现Monte Carlo Tree Search，
由于围棋实现的难度，培训选择了一种较容易实现的对弈游戏*Tic Tac Toe*。

## Tic Tac Toe

![img=ttt](http://www.craftsdirect.com/upload/project_images/tic-tac-toe-tile-board.jpg)
基本的Tic Tac Toe规则较为简单，并且棋盘空间也非常有限，在此基础上进行增强的[Ultimate Tic Tac Toe](http://bejofo.net/ttt)
是培训班上检验Monte Carlo Tree Search实现效果的工具，Ultimate Tic Tac Toe的简单说明如下：

>Ultimate Tic Tac Toe is 9 normal games of Tic Tac Toe played simultaneously. The board is made up of 9
>titles, each of which contains 9 squares. You move will dictate where your opponent can play however.
>So if you play your piece in the top right square of a title, then your opponent must play their next
>piece in the top right title

## 软件设计

![img=design](https://github.com/hxfirefox/alpha_in_erlang/blob/master/src/resources/alphaTTT%20design.png)

## 核心算法

整套系统的核心是[Monte Carlo method](https://en.wikipedia.org/wiki/Monte_Carlo_method)，通过该算法完成从盘面落点位置选择，
模拟从该位置的后续走势，直至对弈结束并统计结果。使用Monte Carlo method使得系统具备了两个特点：
- 时间盒，即可限定获得结果时间
- 无需具备专业的围棋知识

该算法的本质是依赖大量随机样本来逼近问题的解答，通常具有四个步骤：
- Selection
- Expansion
- Simulation
- Back Propagation

>**Tips:** 对于Monte Carlo Tree Search，邓辉老师建议不要纠结于字面上的tree，而是应该按集合空间去理解和实现其算法

以π的计算为例如下：

>For example, consider a circle inscribed in a unit square. Given that the circle and the square have a
>ratio of areas that is π/4, the value of π can be approximated using a Monte Carlo method:
>
>- Draw a square on the ground, then inscribe a circle within it.
>- Uniformly scatter some objects of uniform size (grains of rice or sand) over the square.
>- Count the number of objects inside the circle and the total number of objects.
>- The ratio of the two counts is an estimate of the ratio of the two areas, which is π/4. Multiply the result by 4 to estimate π.
>
>Erlang实现见src/mc_pi.erl

利用相同原理和步骤即可在Tic Tac Toe上得到应用，Erlang实现见src/mcts.erl，这种实现采用随机落子的方式。随机落子在效率上比
较浪费，已有的成果较少获得传承，这引出了另一个问题，即在有限的时间内，落子选择如何平衡“探索与利用现有成果”的问题。此类
问题可通过借助[**Multi-armed bandit**](https://en.wikipedia.org/wiki/Multi-armed_bandit)加以思考，最终引入UCB1算法，
Erlang实现见src/mcts_ucb1.erl。

## Mathematics

站在编程语言之外看待语言，诸如汇编、C++、Java、Erlang等，其本质都是数据，它们之间不相同的是模型——要解决的问题模型，这导致了语义层次的差异，同时也揭示了解决问题的途径——提升抽象层次，语义层次。

引用[《Category Theory for Programmers》](https://bartoszmilewski.com/2014/10/28/category-theory-for-programmers-the-preface/)的原文说明上述理论：

>Changes in hardware and the growing complexity of software are forcing us to rethink the foundations of programming. Just like the builders of Europe’s great gothic cathedrals we’ve been honing our craft to the limits of material and structure. There is an unfinished gothic cathedral in Beauvais, France, that stands witness to this deeply human struggle with limitations. It was intended to beat all previous records of height and lightness, but it suffered a series of collapses. Ad hoc measures like iron rods and wooden supports keep if from disintegrating, but obviously a lot of things went wrong. From a modern perspective, it’s a miracle that so many gothic structures had been successfully completed without the help of modern material science, computer modelling, finite element analysis, and general math and physics. I hope future generations will be as admiring of the programming skills we’ve been displaying in building complex operating systems, web servers, and the internet infrastructure. And, frankly, they should, because we’ve done all this based on very flimsy theoretical foundations. We have to fix those foundations if we want to move forward.

![img=gothic](https://bartoszmilewski.files.wordpress.com/2014/10/beauvais_interior_supports.jpg)

## Thinking

### 如何做到探索与利用现有成果平衡

如何平衡两种策略——探索与利用现有成果，也可以通过概率论上的**Multi-armed bandit**进行处理。

>**Multi-armed bandit**
>
>From Wikipedia, the free encyclopedia
>
>In probability theory, the multi-armed bandit problem (sometimes called the K or N-armed bandit problem) is a problem in which a gambler at a row of slot machines (sometimes known as "one-armed bandits") has to decide which machines to play, how many times to play each machine and in which order to play them.When played, each machine provides a random reward from a probability distribution specific to that machine. The objective of the gambler is to maximize the sum of rewards earned through a sequence of lever pulls.

### 如何用并行的方式，缩小宽度和深度

见src/mcts_p.erl及src/mcts_ucb1_p.erl

## Alpha Go

- mcts 框架 + policy network value network

> 监督式的学习——分类，带答案的问题(人类盘面的走法)，基于前面的学习作答未知问题，policy network 宽度
>
>增强式的学习——通过反馈来刺激学习，追求结果 value network 深度
>
>监督式的学习——回顾分析

## Fun
>**Tips:**
>
>1. 建模过程中是对数据的定义
>
>2. 对并发模型的一些思考

## 代码目录结构
