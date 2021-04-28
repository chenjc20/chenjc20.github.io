---
title: Rethinking File Mapping for Persistent Memory
tags: 
  - tag: Persistent Memory
  - tag: File System
categories: FAST
author: 王少布
---

本文发表在 FAST 2021 上，主要关注点在持久性内存这样地场景下构造/优化文件系统中 file mapping 这个组件，相关组件受到各种相关变量的影响，以及实际负载下的具体表现

[TOC]

## 一 问题背景

### 1.1 persistent memory

持久性内存，又称非易失内存（non-volatile memory，NVM），是一种容量大、速度快的持久性存储介质，填补了传统存储层次金字塔中缺失的一层，弥补了持久性介质和非持久性介质之间的性能鸿沟。


![avatar](\assets\img\papers\2021-04-21\nvm.png)

### 1.2 file mapping

file mapping 是文件系统中非常重要的一个组件，主要负责将inode编号和文件逻辑块地址转换到对应的存储介质上的物理块地址。

传统工作中的 file mapping 与 file io ：

![avatar](\assets\img\papers\2021-04-21\file io.png)

## 二 解决方案

工作一共改进/构造了四种文件结构，其中两种（2.1 2.2）来自改进已有的工作，剩下的两种（2.3 2.4）是新构造的文件系统。

### 2.1 Extent Tree

Extent Tree 是 per-file 的结构，也就是每一个文件都有对应的一个映射数据结构，具体的构造类似 B-tree，每一个间接节点都需要搜索对应的逻辑节点号找到一个具体的 range（节点上的一个 key），这个 key 会索引到下一层节点，而最终的叶节点保存着具体的映射关系：

![avatar](\assets\img\papers\2021-04-21\extent tree.png)

### 2.2 Radix Tree

Radix Tree 也是 per-file 的结构，相对于 B-tree 而言，每一个节点上不需要搜索而是直接使用 逻辑块 十六进制表示的一部分进行数据索引，中间节点对应的项指向下一层节点，叶节点对应的项表示具体物理地址：

![avatar](\assets\img\papers\2021-04-21\radix tree.png)

### 2.3 cuckoo hash

cuckoo hash 使用的是 global 型的映射结构，就是全部的文件共用一个数据结构进行索引，具体而言，相关数据结构使用了 cuckoo hash 的算法解决哈希冲突问题，所以读性能会比较好：

![avatar](\assets\img\papers\2021-04-21\cuckoo hash.png)

### 2.4 HashFS

HashFS 也使用 global 型的映射结构，与 cuckoo hash 不同，它每一个 哈希表项 都对应了一个实际的物理块地址，这个关系自创立之初绑定，所以绕过了 block allocator 的使用，避免了相关的开销，哈希表开始时就确定了大小，也避免了 hash resize 的开销：

![avatar](\assets\img\papers\2021-04-21\hash fs.png)


## 三 实验

### 3.1 Locality
![avatar](\assets\img\papers\2021-04-21\exp1.png)
### 3.2 Fragment
![avatar](\assets\img\papers\2021-04-21\exp2.png)
### 3.3 File Size
![avatar](\assets\img\papers\2021-04-21\exp3.png)
### 3.4 IO Size
![avatar](\assets\img\papers\2021-04-21\exp4.png)
### 3.5 Space Ultilization
![avatar](\assets\img\papers\2021-04-21\exp5.png)
### 3.6 Concurrency
![avatar](\assets\img\papers\2021-04-21\exp6.png)
### 3.7 Page Caching
![avatar](\assets\img\papers\2021-04-21\exp7.png)
### 3.8 实际工作负载模拟
![avatar](\assets\img\papers\2021-04-21\exp8-1.png)
![avatar](\assets\img\papers\2021-04-21\exp8-2.png)


了解更多请关注: [论文原文](https://www.usenix.org/system/files/fast21-neal.pdf) 