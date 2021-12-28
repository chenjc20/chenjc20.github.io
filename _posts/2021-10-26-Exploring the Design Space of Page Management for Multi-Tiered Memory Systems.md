---
title: Exploring the Design Space of Page Management for Multi-Tiered Memory Systems
tags: 
  - tag: Operating System
  - tag: NVM
categories: ATC
author: 王少布
---

这篇文章发表在ATC 2021，通过对现有Tiering Memory的Page进行分析，提出了AutoTiering架构，提供了高效的Memory Page管理方式。

NVM 可持久性地存储数据，并且延迟带宽读写性能明显比 SSD HDD 更优，同时许多 NVM 介质（e.g. Intel Data Center Persistent Memory Module，DCPMM）使用了 3D XPoint 技术，所以相对于 DRAM 而言有着更大的容量，这样的性能很符合高性能计算领域 Large Memory System 场景的需求。 

![avatar](\assets\img\papers\20211026_AutoTiering_DCPMM.jpg)

但是DCPMM读写性能差于DRAM，如何能高效同时利用DRAM和DCPMM，搭建tiering memory，成为

## Design

### 1 Conservative Promotion or Migration

本文的第一个技术点是打通了层与层之间的页转移关系，原先Linux的AutoNUMA管理机制，会将不能成功Promotion到最优位置的页取消迁移，但是实质上我们可以通过转移到其他几个同级或者上级存储中获取到更好的性能效果。

策略表现如图所示：

![avatar](\assets\img\papers\20211026_AutoTiering_design_1.png)

### 2 Opportunistic Promotion or Migration

本文第二个技术点是控制上层存储中页的热度，通过 LAP 算法，在上层存储满的时候，将上层热度不高的页和下层热度更高的页进行对换，使得热度更高的页能待在高速的 DRAM 中。

策略表现如图所示：

![avatar](\assets\img\papers\20211026_AutoTiering_design_2.png)

### 3 Background Demotion

第三个技术点目的在于掩盖技术点二中换页的开销，它通过在上层存储中保留一定区域，在需要上下换页的情况将下层页先转移到这个区域内，然后后台再起线程将页转移到下层，通过这种异步的解耦方式减小了关键路径开销。

策略表现如图所示：

![avatar](\assets\img\papers\20211026_AutoTiering_design_3.png)

## Evaluation

### Performance

如图所示，AutoTiering 有效提高了应用整体的内存利用能力，并进一步提高了应用的执行性能。

![avatar](\assets\img\papers\20211026_AutoTiering_experiment_1.png)

### Effectiveness

#### CPM

如图所示，CPM策略下分级存储的利用更为均衡，拥有更好的执行效率。

![avatar](\assets\img\papers\20211026_AutoTiering_experiment_2.png)

#### OPM

如图所示，OPM策略下冷热数据又更好的分配，拥有更好的执行效率。

![avatar](\assets\img\papers\20211026_AutoTiering_experiment_3.png)

## Else

了解更多请关注: [论文原文](https://www.usenix.org/conference/atc21/presentation/kim-jonghyeon) 