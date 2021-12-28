---
title: Achieving Low Tail-latency and High Scalability for Serializable Transactions in Edge Computing
tags:
  - tag: Transaction
  - tag: Consistency
  - tag: Replication
categories: Eurosys
author: 姜天洋
---

这篇文章发表在Eurosys 2021上，主要内容是降低跨数据中心分布式事务的延迟。针对边缘计算的场景，用户希望系统能提供低且稳定延迟的事务，并且保证可串行化的隔离性等级。从这两点出发，本文分析了geo-distributed system中出现事务长尾延迟的原因。
文章将geo-distributed system中的事务分为CRT (cross-region transaction)和IRT (inner-region transaction)。原因主要包括以下两点。
- IRT被一个CRT的执行所阻塞，被阻塞的多个IRT延迟上升。
- CRT由于跨数据中心访问，执行时间长，在执行过程中与其他事务冲突，会引起高额的中止开销增加延迟。

## 解决方法

针对以上两个原因，本文设计了跨地理数据中心的分布式边缘数据库DAST，对于原有基于时间戳的事务协议进行了修改优化，主要包括以下两个技术点。
- 针对IRT可能被CRT阻塞的问题，DAST使用了可弹性的逻辑时间戳，允许IRT的执行穿插到CRT执行之前执行。
- 针对CRT执行有可能被中止引起重试，DAST使用了一个两阶段提交的协议，第一阶段先通过消息交互为CRT确定一个不会被中止的时间戳，再执行CRT。

## 实验结果

DAST在Janus的基础上进行了修改，并进行了实验。对比了多个分布式事务系统，Tapir，SLOG (确定性事务数据库)，Janus。
实验结果显示，DAST降低了87.9%~93.2%的IRT的尾延迟，和27.7%~70.4%的CRT的尾延迟。
