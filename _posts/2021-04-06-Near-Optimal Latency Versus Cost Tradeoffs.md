---
title: Near-Optimal Latency Versus Cost Tradeoffs in Geo-Distributed Storage
tags: 
  - tag: Distributed Storage
  - tag: Near-Optimal
  - tag: Tradeoffs
categories: NSDI
author: 滕云
---

这篇文章介绍了基于地理分布的接近最优的tradeoff方法Pando。Pando解决了已有方法延迟和开销的均衡问题，一方面通过延迟隐藏缩减了延迟，一方面利用纠删码降低了存储开销，性能非常接近理论最优方法。

## 问题背景

在Pando提出之前，EPaxos被认为是最优的tradeoff方法。但是由于复制导致每一个备用站点都存储完整的副本，需要较大的开销。之后有的团队提出了基于纠删码的Paxos用来降低开销，然而这使得数据分散到多个站点，在保证一致性时需要保证多个站点的重叠从而提高了延迟，所以RS-Paxos基于存储开销的优化并没有取得很好的收益，因而需要一个能做到更好的tradeoff的方法。

## 主要挑战

文章从RS-Paxos出发进行优化，该方法主要存在以下挑战：
- 两阶段写所带来的延迟很高
- 使用纠删码导致的多个站点的重合也会严重影响延迟


## 解决方法

### 接近一轮写延迟的两轮写

![avatar](\assets\img\papers\pando-cutlatency.png)

如上图所示，Pando使第一轮写的站点数量少于第二轮，并找到一个作为代理的站点，从代理站点开始第二轮写的过程，从时间上来讲相较于每一轮写都从服务器到最远的节点的方法隐藏了将近一半的延迟。

### 单个站点重合（正常情况）

![avatar](\assets\img\papers\pando-onesite.png)

为实现读写重合的站点尽量小，Pando将系统分为正常情况和故障情况，正常情况只需要1个站点重合就可以根据少量站点通过纠删码构建出整个数据，而不需要有k个重合（k为纠删码一个条带的数据块数量），当版本更新错误的时候也就是故障情况，存在少量的站点没有成功更新，Pando感知到故障存在将重叠的站点个数扩大为k，修复错误之后恢复到重叠站点数量只有1个，因为发生这种错误的概率比较小，所以这种优化是比较有效果的。



## 实验结果

![avatar](\assets\img\papers\pando-gapvolume.png)

上图为Pando和几种其他方法进行比较的的相对于最优下界lower bound的测试结果，可以看到Pando获得了最接近最优下界方法的延迟。
 
了解更多请关注: [论文原文及ppt](https://www.usenix.org/conference/nsdi20/presentation/uluyol) 
