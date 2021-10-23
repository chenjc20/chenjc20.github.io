---
title: Jump-Starting Multivariate Time Series Anomaly Detection for Online Service Systems
tags: 
  - tag: Anomaly dectection
  - tag: Compression
categories: ATC
author: 朱心远
---

这篇文章是发表在ATC21上的一篇文章，主要讲述了在多元时间序列上进行异常检测的问题。作者观察到基于深度学习的方法需要大量训练数据，会有很长的启动时间。为了缩短启动时间，作者引入了**压缩感知**的技术，这是一种恢复采样数据的技术，可以以ms级的时间从采样点中恢复出原图像。同时，作者设计一种outlier-resistent的采样方法，这样将恢复图像和原图进行比对，判断某个数据点是否异常。作者根据这些技术，提出了JumpStarter的异常检测框架，如下图。

![avatar](\assets\img\papers\20210817_framework.png)

## 实验结果

![avatar](\assets\img\papers\20210817_result.png)

JumpStarter用远短于其他方法的启动时间（20分钟），取得了SOTA的F1-score。

了解更多请关注: [论文原文](https://www.usenix.org/conference/atc21/presentation/ma) 

