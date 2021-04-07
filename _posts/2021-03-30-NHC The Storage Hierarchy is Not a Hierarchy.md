---
title: The Storage Hierarchy is Not a Hierarchy:Optimizing Caching on Modern Storage Devices with Orthus
tags: 
- tag: Cache
- tag: Storage
- tag: Hierarchy
categories: FAST
author: 姜天洋
---

本文介绍了一种新型存储架构NHC。基于现有存储架构下的观察:caching策略中performance device和capacity device的性能差距越来越小，本文设计了一中新型存储架构，旨在发挥capacity device的全部性能优势，增加整个cache系统的服务带宽。本工作由University of Wisconsin Madison的Kan Wu等人完成。

## 问题分析
分级存储架构是计算机设计伊始的核心，早先的CPU-Cache-DRAM-Disk就是这让的分级存储结构，每层结构之间性能差距巨大，caching策略的核心在于最大化hit ratio，使得请求尽量落在performance device上，以获得低延迟和高带宽。但是，随着存储技术的发展，越来越多功能不同的存储设备出现，存储架构中不同层级设备的性能差距也在逐渐缩小甚至抹平。因此，也需要capacity device发挥应有的作用，为系统贡献带宽输出。
Tiering策略相比Caching，会对数据进行静态的分区，将热数据放在performance device上，冷数据放在capacity device上提供服务。数据分区的调整由后台完成，因为tiering避免了数据访问时关键路径经过performance device，因此tiering只支持较粗粒度的数据迁移（否则需要额外的元数据维护位置信息）。在这种架构下，capacity device也可以发挥其优势贡献带宽。
在相同的hit ratio下，作者针对DRAM/NVM, NVM/Optane SSD, Optane SSD/SSD三种storage hierachy进行了测试。通过模拟实验和真实场景测试，tiering被证明相比caching拥有最高达一倍的吞吐提升。因此，caching在现有storage hierachy下并不是最优方案。

## 解决方案
值得注意的是，caching策略在大多数环境下还是适用的。如果performance device的负载处于较低水平，那它显然比capacity device更应该向上提供服务。因此NHC的策略是: 当performance device满负荷后，将发往performance device的读请求重定向到capacity device，让后者进行服务。
performance device在原有caching架构中会将一部分的带宽浪费在和capacity device的数据迁移上，通过关闭这种数据迁移，将performance device的带宽全部用于前台负载。
具体的实现分为两个阶段:
Step 1
使用传统的cache替换策略进行替换，监控cache hit ratio，如果cache hit ratio稳定，则进行step 2.
Step 2
对于read miss，直接将请求发送到capacity device上，并且不将该数据缓存到performance device上（即关闭数据替换）。
对于read hit，动态调整发到performance device上的负载比例load_admit，将部分负载转移到capacity device上。
问题的优化目标可供选择，包括负载的throughput/average latency/tail latency，根据这些优化目标，采取梯度下降的方法调整load_admit的数值。如果出现以下两种情况，则返回到Step 1.
1 load_admit=100%时效果最好，说明performance device上的负荷可能尚未满载。
2 发现目前的cache hit ratio相比进入Step 2时明显降低，说明负载locality可能出现巨大变化，需要动态迁移数据来进行调整。

本文的解决方案和各种cache替换策略是正交的，其切换方案也不涉及cache的各种写方案（write through/write back/write around），但是对于不同的写方案，NHC的效果不同。对于write through和write around，由于capacity device中的数据不会旧于performance device，所以在Step 2中对于read hit有更灵活的负载平衡策略，而对于write back，很多read请求只能从performance device响应，这对于调度有所限制。

## 实验结果

本文实验部分实现了一个caching的内核模块，以及一个caching的LSM，并在不同storage hierachy环境下进行了比较。此外，本文还展示了NHC针对动态变化负载的适应能力，当负载locality变化时，如何感知这种变化并及时调整策略进行相应。


了解更多请关注: [论文原文及ppt](https://www.usenix.org/conference/fast21/presentation/wu-kan) 