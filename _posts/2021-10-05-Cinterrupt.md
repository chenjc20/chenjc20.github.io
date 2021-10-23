---
title: Optimizing Storage Performance with Calibrated Interrupts
tags: 
  - tag: NVMe SSD
  - tag: interrupts
categories: OSDI
author: 李之悦
---

这篇文章是发表在OSDI 2021上的一篇文章，主要内容是探讨高速的NVMe设备上如何实现兼顾延迟与吞吐的中断生成策略。文章首先对现有的NVMe SSD中断生成策略进行了改进，提出了adptive interrupt策略，该策略将固定的阈值参数变为可以随负载变化的参数，能够很好的满足延迟敏感应用（Lat应用）与带宽敏感应用（BW应用）所混合的情况。此外，为了解决adaptive策略引入固定长度时延的问题，本文进一步提出对中断进行标定的方法（calibrated interrupts），由用户层显式指定一批请求的结尾，以降低请求延迟、提升总体吞吐量。

## NVMe SSD中断生成策略

决定NVMe SSD的中断产生时机的参数主要有两个，一个是时间参数timeout，一个是完成数量参数threshold。时间参数的含义是从上一次中断之后的第一个completion产生开始，如果达到了timeout时间，则会产生一个中断。完成数量参数的含义是从上一次中断之后，产生了threshold个completion信息之后，就会产生一个中断。两个产生中断的条件是一个或的关系。

## adaptive interrupts

该算法是对标准的NVMe SSD进行的改进，其主要特点有两个：①不再选用固定的timeout与threshold，而是根据负载和设备来选择，文章在实验章节的第一部分介绍了选择参数的方法。②条件一的判断做修改，不再是从上次中断后第一个completion产生的时间开始计时，而是每产生一个新的completion，都会重置计时器，这样做能在取较小timeout时间的情况下，有效防止BW型应用频繁产生completion信息而降低BW型应用的性能。

## calibrated interrupts

上述adpative interrupts算法有一个问题，就是对于每一个（或一批）完成信息，一般会引入一个timeout的等待，而这段时间会固定的加到请求延迟里。为了减少这一部分等待，该论文引入了calibrated interrupt的概念，该方法会让应用层显式的标定Lat型应用的请求（Urgent标记）以及BW型应用的最后一个请求（Barrier标记）。下层驱动在看到completion信息中存在Urgent或者Barrier标记时，会立刻向上层发出中断。这种策略能很好的兼顾Lat与BW型应用的需求。

## 实验结果

实验部分对比了两种不同的中断生成方法：每个请求都生成中断的方法（default方法）与adaptive方法。实验表明，本文提出的cinterrupt方法很好的用到了应用层提供的信息，能够在CPU使用率、吞吐与延迟上进行优化。

了解更多请关注: [论文原文](https://www.usenix.org/conference/osdi21/presentation/tai) 