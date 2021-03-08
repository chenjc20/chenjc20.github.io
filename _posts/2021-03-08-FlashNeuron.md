---
title: FlashNeuron SSD-Enabled Large-Batch Training of Very Deep Neural Networks
tags: FAST’21 Sys4AI ML SSD
categories: FAST
author: 魏钧宇
---

本文来自韩国首尔国立大学以及三星电子的Jonghyun Bae等人。突出强调了深度学习训练中的内存墙问题，设计了一种将训练过程的中间结果下移到NVMe-SSD中的方法，采用这样的方法能够提高BatchSize的大小，提高训练的吞吐率，同时还减少GPU和CPU之间的打扰。

**整体介绍**
在深度学习训练中，由于GPU显存受限，因此通常不能够加大BatchSize的大小。但是小BatchSize通常不足以充分发挥GPU的性能，从而会造成训练吞吐率的降低。过去的工作采用的方法通常是将GPU训练的中间结果（比如FeatureMap）下移到HostMemory之中，采用该方法能够实现大的吞吐，但是因为CPU和GPU之间来回传输数据的开销，将会造成吞吐的下降，同时还会存在使用HostMemory的计算任务的互相干扰。

本文将训练的中间结果Offloading到NVMe-SSD上，从而实现了对于大BatchSize的支持，但是由于NVMe-SSD的带宽受限，需要采用精细化调度来优化数据传输，同时使用GPU-direct的策略，直接实现GPU和SSD的数据传输，最大程度地降低对于CPU的打扰。

**背景：GPU访存的内存墙问题**
本工作的动机建立在现有的深度学习训练存在显存不够这个问题的基础之上，关于这一点，之前的工作的做法都是将额外的tensor都放到DRAM中。

本文的动机叙述有两个角度，第一个角度是现有的深度学习训练通常需要更大的BatchSize，如果采用小BatchSize会造成GPU利用率的下降，但更大的BatchSize又会出现GPU显存不够的问题，因此需要将Forward得到的中间结果移出到其他设备上，等到Backward的时候再加以使用。另一方面，现有的深度学习训练任务很多都需要CPU来完成与处理的工作，这些预处理工作需要使用到内存带宽，而如果将tensor移到DRAM上就不可避免地会和这些预处理任务竞争带宽，造成预处理任务性能的下降（但这里有一个问题，预处理所需要的内存带宽是否有想象中那么大？）

**方法概述**
*内存管理*
FlashNeuron通过cudaMemoryAllocate分配得到了一个比较大的连续空间，然后自己在上面完成内存管理。首先是Tensor的释放和分配，为了防止碎片化，本文将常驻内存的tensor分配在低端，将需要来回迁移的tensor放到了高端。其次是采用了压缩技术来减少和SSD交互所需要的通信量，具体来说采用的压缩技术有CSR压缩和半精度训练。

*Offloading策略*
Offloading的基本思路是，首先有一个profiling的过程，在该过程中，可以通过建模确定每一个下降的tensor的大小和压缩率。然后在第一阶段，首先逐个下放tensor，直到内存空间足够训练为止，在这个过程中测量下放tensor的过程所需要的时间和整个前向训练所用的时间，比较这样的两个时间，如果发现下放的用时太长了，就尝试从最后一个下放的tensor开始做松弛，然后不断地添加其他的压缩率更高的tensor直到下放tensor的用时要小于训练的用时，这样就可以将下放tensor所用的时间隐藏了。

*直接SSD访问*
本文的第三个创新点是使用了一个直接SSD访问的策略，不需要对于CPU产生打扰，主要就是使用了两个硬件提供的接口，一个是GDRCopy策略（可以实现GPU内存的直接访问），另一个是Intel SPDK能够实现用户态对于block-level的直接访问。写由七个步骤完成：首先发出Request，接下来分配LBA并记录，然后下发请求到队列中，队列将对应请求下发，然后使用GDRCopy+SPDK将数据连续写入到NVMe SSD上，在写入完成之后更新元数据表。在下一次写的时候，检查元数据表，确认已经更新完成。

---

了解更多请关注: [论文原文及ppt](https://www.usenix.org/conference/fast21/presentation/bae) 