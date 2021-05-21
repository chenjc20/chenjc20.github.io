---
title: REMIX:Efficient Range Query for LSM-trees
tags: 
  - tag: LSMTree
  - tag: KVS
  - tag: Range Query
categories: FAST
author: 魏钧宇
---
## 问题背景
本文设计了一个LSMTree上专门针对读进行优化的数据结构，这个结构保存了多个SSTable对应的run上的全局视图，从而实现了在Tired的压缩方法下的高效读。

LSMTree在很多场景下都非常常见，具体包括KV存储，关系数据库，数序数据库等领域。它本身成为了社交媒体、电子商务、实时分析等任务的重要组成部分。

LSMTree的基本打包方式有两种：一种是Tiered压缩，每当上层满了，就把上层对应的所有的SSDTable Merge起来然后直接丢到下面一层去。

第二种是Leveled压缩，这个方法在上层满了之后，将把下层的内容读上来，然后将上层的内容和下层的做一个Merge再写到下层去。简单分析这样的两种方法，Tiered压缩对于写是友好的，存在较小的写放大问题，Leveled压缩对于读是友好的，每一层的不同的SSDTable组成的结构是有序的，但问题在于Tiered的方式对于读效果不理想，Leveled方式对于写效果不理想

![avatar](\assets\img\papers\2021-05-20\img1.png)

本文的研究范式是为了同时达到高效的写和高效的读，本文先从写较高效的Tiered方法出发，观察该方法读性能差的原因，然后尝试去优化这种读性能。

本文总结了之前的TieredCompaction方法在读的时候的三个问题：1）对于每一个区间都需要执行一次二分搜索以确定初始位置，2）推进每一个区间指针都需要和所有的候选指针做比对；3）即使一个区间完全不包含任何key也能够被访问。

因此本文设计了Remix，它将全局相对稳定的有序视图存储了下来，然后使用较多随机的IO来换取软件栈开销的降低

## Remix数据结构

![avatar](\assets\img\papers\2021-05-20\img2.png)

Remix的数据结构主要由三个部分组成，本文将全局的有序视图切分成了若干个段，然后存储每一个的第一个Key作为AnchorKey，然后存储对于每一个SSDTable来说，当前段在这个SSDTable中出现的第一个Key对应的Offset（Cursor offsets），并存储按照顺序去遍历段内的每一个Key的时候对应访问的SSDTable分别是哪几个（Run selector）

有了这样三个部分，Remix就可以实现对于一个区间内数字的查询，主要优势包括：只需要对于锚点做一次二分，每一次推进指针都不需要再进行比对，同时如果有一个区间完全不包含待检索的区间就可以完全跳过。

## RemixDB
在Remix的基础上，本文设计了RemixDB，它本质上就是一个单层的LSMTree的结构，能够同时实现高效的读和高效的写，它的打包操作主要有三个，一个是Minor压缩（将写下去的NewData转换为NewTable），一个是Major压缩（将多个OldTable合并成一个NewTable），一个是Split压缩（将不同的NewTable划分成不同的Partition）

## 实验
主要有两个实验：微观实验设置，宏观实验设置
微观实验测试了50个区间的读，在涉及5个TableFiles时可以提升2倍，在涉及16TableFiles时可以提升3.1倍，在宏观实验上，纯写的实验（Load），A，B，C，E，F效果都较为明显，D则因为大多数读都在Buffer中，所以效果并不明显。
