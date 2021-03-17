---
title: Fast RDMA-based Ordered Key-Value Store using Remote Learned Cache
tags: RDMA ML KVS
categories: OSDI
author: 陈军超
---

目前，KVS存储越来越受到研究人员的关注，特别是作为分布式系统的数据库、图像存储以及网络应用存储等。RDMA作为访问KVS系统的有效技术方法，在访问远端具有线性结构（例如哈希表）的KVS数据时效果比较好，但是当远端KVS是基于排序的树形存储结构时则有很大的性能瓶颈，一般需要多次RTT进行数据的传输。传统的两种访问设计是使用以服务器为中心的设计和客户端直接访问的设计，前者造成服务器端CPU的大量消耗，后者将树形索引的一部分缓存到本地缓存，但由于树形结构庞大的内存占用，这将导致客户端内存的大量消耗以及当树更新时，客户端缓存结构的同步将导致性能的急剧下降。这篇论文认为机器学习模型是能够较为出色的解决基于树型索引结构的缓存模型，并设计实现了具有混合架构的基于RDMA的有序键值存储——XSTORE。

XSTORE在服务器上保留基于树的索引的体系结构以执行动态负载（会改变树结构的操作，例如插入和删除），并利用在客户端通过机器学习方法学习到的缓存以执行静态工作负载（不改变树结构的操作，例如读取和扫描）。客户端采用了神经网络+线性回归的两级机器学习模型来学习服务器端KVS树型索引结构，通过以降低精度的方式将树型索引结构学习到本地缓存，并引入一层转换表，用于当服务端的物理地址未排好序时，使用排好序的逻辑地址与物理地址进行映射。由于会损失一定的精度，通过机器学习方法建立的缓存要比实际树型索引结构占用的内存小很多，通常100M的KVS只需要500K的线性回归模型就能够实现服务端缓存的全覆盖。当客户端每次进行键值查找时，先由缓存的学习模型确定键值存储的大概位置（逻辑地址），并通过一个RDMA的RTT将该范围内的键址数据（键与存储的物理地址的对应关系表）从服务器取到客户端，本地客户端在键址表中查找到对应的物理地址后，再通过另一个RTT将数据拿到本地，从而完成服务端数据的获取。

引入机器学习的XSTORE不仅能够减少服务器CPU的负载，减少RDMA的RTT次数，同时也能节省大量的客户端缓存，并借助机器学习的计算能力来代替内存的检索开销，从而大幅度提高访问远端KVS数据的性能。实验结果表明XSTORE比同期最先进的基于RDMA的键值存储方式（例如DrTM-Tree、Cell等）的性能提升3.7到5.9倍，同时也有效减少了客户端的存储压力。

---

了解更多请关注: [论文原文](https://www.usenix.org/conference/osdi20/presentation/wei)  
