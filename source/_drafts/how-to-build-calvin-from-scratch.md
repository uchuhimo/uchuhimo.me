---
title: 如何从头设计出 Calvin？
date: 2017-05-09 16:32:06
updated: 2017-05-09 16:32:06
tags: [分布式, calvin, database, ACID]
categories:
---

Calvin[^calvin] 是一种分布式事务数据库的设计架构，它在支持完整的 ACID 跨分区事务的同时保持线性可扩展的能力令人印象深刻。现在市面上有众多的号称线性可扩展的 NoSQL 数据库，但它们都在某种程度上牺牲了事务支持的能力——要么是只支持单分区的 ACID 事务，要么是没有完整的 ACID 支持。它们似乎达成了一种共识：线性扩展能力和完整的事务支持是相互矛盾的，只能二择一。而 Calvin 给我们提供了另外一种思路，即通过对事务类型的限定和系统架构的仔细设计同时支持两者。Calvin 的论文从 top-down 的视角提供了详细的设计架构与细节探讨，但相较于“怎么做”（how），我们往往更关心“为什么这么做”（why）：Calvin 是如何以一个不提供事务支持的单机存储系统为基础，逐步设计出来的呢？下面我们就用一种不同于论文的系统演进的角度重新认识 Calvin。

<!--more-->

## 目标与指标

任何系统在设计阶段就应该明确两样事情：目标（goal）和指标（metrics）。目标只有“完成”和“未完成”两种状态，没有 trade-off 的余地，是设计是否合格的基线；而指标则是可以 trade-off 的，它是设计优劣的衡量标准。比如说一个数据库系统“在上线时间达到 99.99% 的情况下端到端的平均读取时间为 10 ms”，那么“上线时间达到 99.99%”就是目标，而“端到端的平均读取时间为 10 ms”是指标。我们比较不同系统的方法就是在相同的目标下考察对比它们的指标，从而得出判断。

那么 Calvin 的目标和指标是什么呢？Calvin 想在一个不支持 ACID 事务的、提供 CRUD 接口的单机存储系统上构建一个分布式事务数据库，包括以下目标：

- share nothing：节点间不共享存储
- (near-)linear scalability：（近）线性可扩展，随着集群规模扩大，事务处理能力应当线性增长
- high availability：高可用，在系统发生非确定性错误时（比如硬件失效、进程失活等）能够继续提供服务
- strong consistency：强一致性，系统的可见状态在任何时刻保持一致
- full ACID transaction (span multiple partition)：完整的 ACID 跨分区事务支持

在达成这些目标的基础上，Calvin 使用以下指标衡量系统：

- transactional throughput：事务吞吐，即固定时间内事务的处理量
- transactional latency：事务延迟，即事务从进入系统到返回给客户端的端到端延迟

Calvin 这一系列目标和指标间有着错综复杂的依赖和影响：

- share nothing 是 linear scalability 的基础，共享存储的系统会随着集群规模的扩大而冲突迅速加剧，基本无法做到线性可扩展
- 支持 high availability 意味着系统需要某种形式的备份，而备份会带来一致性问题，引出了对 strong consistency 的需求
- 支持 strong consistency 通常意味着事务延迟的增加，在某些情况下也意味着事务吞吐的降低
- 支持完整的 ACID 跨分区事务常常意味着事务吞吐的下降和事务延迟的增加

在目标和指标间有复杂冲突的情况下如何权衡系统的设计呢？最简洁的做法是：先按目标确定系统的整体框架，再根据指标优化细节。不要在整体框架的设计阶段受到优化指标的压力。我们来看一下 Calvin 是如何按预定目标确定整体架构的。

## 整体架构



[^calvin]: Thomson, Alexander, et al. "Calvin: fast distributed transactions for partitioned database systems." Proceedings of the 2012 ACM SIGMOD International Conference on Management of Data. ACM, 2012.
