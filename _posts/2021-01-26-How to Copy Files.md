---
title: How to Copy Files
tags: 
  - tag: File System
  - tag: COW
  - tag: Be-Tree
categories: FAST
author: 陈军超
---

这篇论文是关于如何高效率的对文件目录树进行快速克隆操作，由来自北卡罗来纳大学教堂山分校的Yang Zhan，Yizheng Jiao以及 罗格斯大学，佩斯大学，石溪大学，VMWare研究中心的相关研究人员合作开发完成的。

现有的文件系统已经实现了一些“逻辑拷贝”功能，有别于“物理拷贝”，逻辑拷贝一般在实现时只进行一些元数据级别的拷贝，而只有在后续的修改中，才按需对文件数据进行拷贝并修改，这种功能称作COW（copy-on-write），这种方法在拷贝的时效性和空间的利用率方面都得到了较大的提升，因此现在已经成为克隆的基础技术。

然后对于经典的COW技术，主要存在着“拷贝粒度”的问题，很容易造成写放大，造成内存的碎片化，使得文件系统后续的读取速度会很慢。因此这里作者提出了自己对高效克隆的看法，需要满足如下4个特性才能被称作是理想的，作者称为“Nimble clones”：
* 必须能快速的完成克隆操作；
* 必须有良好的read locality；
* 必须有良好的写性能；
* 空间利用率必须要好，写放大应该尽量保持比较低的水平。

作者认为，这里的关键之处是应该将COW中的copy和write的大小进行解耦，即不应该使用一样的大小，如果是进行大的文件修改，那copy大块然后再覆盖写大块当然是合理的，但是如果是微小的修改，显然，应该将这次微小修改暂存起来，让多次微小修改进行聚合，在达到一定数量后，再批量处理显然会更合理一些。

这篇论文在现有的BetrFS基础上，实现了一种高效的克隆机制，以同时满足前述的Nimble clones需要具备的特征。作者引入了称作CAW（Copy-on-Abundant-Write）的技术，以暂存微小数据修改，等合适的时机再进行COW。另外，这篇论文在3方面对原来的Bε-tree数据结构进行了改进与增强。
1. 将原来的Bε-tree树结构改造成Bε-DAG结构（directed acyclic graph，有向无环图），以满足对整棵树进行更好的遍历。
2. 引入了GOTO message，可以快速的持久化克隆操作。
3. 引入“translation prefix（前缀转换）”,用来满足对“延后数据拷贝”和 部分共享数据的查询。在引入这些优化和改进后，实验结果显示，对不同的模型，至少有33%到6.8倍的性能提升。

论文的贡献点归结为：
* 设计和实现了Bε-DAG数据结构，作为Nimble clones的基础。该方法扩展了Bε-tree,用来聚集小的修改，再择机进行批量写入。
* 写优化的克隆实现。在进行克隆操作时，仅需简单的向DAG root节点写入一个GOTO message消息。
* 量化的渐进分析表明，增加克隆不会影响其它的操作。克隆算法开销是对数级别的。
* 全面的测试表明，优化过的BetrFS并没有对原来的BetrFS基线产生不利的影响，而在clone特性上，对比传统的支持克隆特性的文件系统，在性能上有3-4倍的提高，而对比不支持克隆特性的文件系统，则有2个数量级的提高。

该篇文章是在作者团队开发的BetrFS文件系统的基础上进行文件系统的优化的，且作者团队针对该系统的论文已发表数10篇。

---

了解更多请关注: [论文原文](https://www.usenix.org/conference/fast20/presentation/zhan) 

了解更多BetrFS文件系统的知识，请访问：[BetrFS官网](http://www.betrfs.org/) 

本内容参照：[论文解读](http://www.ctoutiao.com/2672341.html)

本文引用: Zhan, Yang, Alexander Conway, Yizheng Jiao, Nirjhar Mukherjee, Ian Groombridge, Michael A. Bender, Martin Farach-Colton et al. "How to copy files." In 18th USENIX Conference on File and Storage Technologies (FAST'20), pp. 75-89. 2020.