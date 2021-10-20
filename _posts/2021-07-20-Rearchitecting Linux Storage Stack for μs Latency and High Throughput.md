---
title: Rearchitecting Linux Storage Stack for µs Latency and High Throughput
tags: 
  - tag: Operating System
  - tag: NVM
categories: OSDI
author: 王少布
---

这篇文章发表在OSDI2021，通过对Linux Block Layer进行修改，提出了blk-switch架构，在保持高带宽情况下实现了低延迟的需求。

## Overview

随着NVM和RDMA的引入，现代存储设备的访问性能越来越好：（1）能同时接受更多访问队列（2）访问延迟和带宽都极大改善。本文认为随着发展，这些存储设备的接口模式接近网络设备，而块请求的处理需求也类似网络中交换机，所以采用交换机的一些理念对操作系统 blk 层进行改善，提出了blk-switch的架构。

blk-switch的系统架构如下图所示。

![avatar](\assets\img\papers\20210720_blk-switch_sysarc.png)



## Design

### 1 Prioritize L-app request processing

本文第一个技术点是将L-app（延迟敏感应用）和T-app（带宽敏感的应用）请求分队列进行处理，并优先处理L-app的请求，这样防止了小体量L-app请求被大体量T-app请求阻塞，有效减小了L-app请求的延迟。

策略表现如图所示：

![avatar](\assets\img\papers\20210720_blk-switch_design_1.png)

### 2 Request Steering for transient loads

本文第二个技术点是请求转移，主要是关注到了T-app会因为低优先级饥饿的情况，对于短时期的负载压力，blk-switch会将受到阻塞的T-app请求转移到其他空闲核进行处理，以享受更多的处理资源，缓解饥饿带来的带宽降低问题。

策略表现如图所示：

![avatar](\assets\img\papers\20210720_blk-switch_design_2.png)

### 3 Application Steering for transient loads

本文第三个技术点是应用整体转移，当某一个核上应用过多，受到了长时间压力时，blk-switch会将这个核上的T-app转移到其他的空闲核上进行处理，这样缓解了整体长时间的带宽压力。

策略表现如图所示：

![avatar](\assets\img\papers\20210720_blk-switch_design_3.png)



## Evaluation

### Latency and Throughput

如图所示，blk-switch有效改善了延迟（尤其是尾延迟）和带宽性能。

![avatar](\assets\img\papers\20210720_blk-switch_experiment_1.png)

### Performance Breakdown

如图所示，自左向右从Linux原生系统增加策略，观察到每个优化策略对带宽和尾延迟的影响。

![avatar](\assets\img\papers\20210720_blk-switch_experiment_2.png)

## Else

本文还有一点亮点是小修改，因为修改内容少，对原有系统影响也小，保留了更多的feature和扩展性。

了解更多请关注: [论文原文](https://www.usenix.org/conference/osdi21/presentation/hwang) 