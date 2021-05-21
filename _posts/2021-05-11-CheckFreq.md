---
title: CheckFreq:Frequent,Fine-Grained DNN Checkpointing
tags: 
  - tag: ML
  - tag: Sys4AI
  - tag: Duplication
categories: FAST
author: 胡世鹏
---

这篇文章介绍了一个频繁的、细粒度的深度学习训练检查点系统CheckFreq。CheckFreq能够在DNN训练过程中自动地调整产生检查点的频率，通过adaptive tuning避免负载变化带来额外的开销，在做检查点时通过两阶段法将该过程与GPU计算过程流水化，减少做检查点的开销，并且实现了可恢复的data loader。

## 问题背景

目前在DNN训练过程中通用的检查点策略是以epoch为边界人为地设置检查点，这样可以防止在训练过程中因为服务器crash而造成训练模型丢失，但是随着目前的神经网络模型结构越来越复杂，每个epoch的训练时间不断加长，从而迫切需要细粒度的检查点技术。CheckFreq能够以mini-batch为粒度对模型做检查点，同时为了减少高频率检查点的额外开销，使用了频率自动调节以及检查点流水化的方式。

## 主要挑战

实现高频率、细粒度的深度学习检查点系统主要存在以下挑战：
- 如何避免高频率产生检查点的巨大开销
- 如何恢复上一个检查点的状态
- 如何确定产生检查点的频率

## 解决方法

### 两阶段检查点流水化技术

![avatar](\assets\img\papers\two_phase_pipeline_checkpoint.png)

如上图所示，CheckFreq将检查点的产生过程分为两个节点：Snapshot()和Persist()，其中Snapshot()指将模型的权重从GPU内存复制到CPU内存，Persist()指将模型的权重从CPU内存写到磁盘。如果按照传统的检查点方法，在执行Snapshot()和Persist()时GPU会产生停滞，但是CheckFreq通过将这两个操作与GPU计算过程流水化处理，避免了GPU长时间停滞，减少了检查点开销。

### 可恢复的数据迭代器

![avatar](\assets\img\papers\resumable_data_iterator.png)

在DNN训练过程中，每一轮epoch都需要遍历数据集中的所有数据刚好一次，这样才能保证训练过程具有较好的收敛性；由于CheckFreq以mini-batch而不是epoch为边界做检查点，因此当系统发生故障需要恢复至上一个检查点时，需要能够记录上一个epoch都在哪些数据上完成了训练。如上图所示，CheckFreq实现了一个可恢复的数据迭代器，通过以epoch作为随机数种子，记录目前完成多少mini-batch的训练，从而恢复至上一个检查点的状态。

### 检查点频率自动调节

为了在保证低开销的同时尽可能提高检查点频率，从而降低训练恢复时间，CheckFreq提出了一种检查点频率自动调节系统，该系统首先对每轮训练迭代的时间、Snapshot()的时间、Persist()的时间、磁盘带宽等进行profiling，然后自动决策相应的检查点频率，避免造成GPU停滞。同时，由于系统负载可能会不断变化，CheckFreq会动态地对系统进行监测，从而实现检查点频率的自动调节。

## 实验结果

![avatar](\assets\img\papers\exp_training_overhead.png)

上图为CheckFreq检查点技术相比于baseline降低的开销，CheckFreq通过two-phase checkpoint并且与GPU计算并行化，大大降低了产生检查点的开销。

![avatar](\assets\img\papers\exp_low_recover_time.png)

上图为CheckFreq的高频率、细粒度检查点策略相比于baseline降低的故障恢复时间。
 
了解更多请关注: [论文原文及ppt](https://www.usenix.org/conference/fast21/presentation/mohan) 
