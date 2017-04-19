---
title: 流计算核心概念解析之 window
date: 2017-04-19 16:14:54
updated: 2017-04-19 17:02:00
tags: [window, streaming, bigdata]
categories: 流计算
---

流计算是当前备受关注的一个大数据研究与应用领域，旨在实时或近实时地处理大量无界数据。区别于批处理，流处理中处理的数据集是无界的，所以聚集操作（如 reduce、sum、count 等）无法应用于整个数据集，否则聚集操作的结果可能永远都不会输出。我们需要将无界的数据集切分成一些有界的片段，将聚集操作应用于这些片段，从而能够在无界的数据集上得到持续的聚集操作输出结果，这一切分过程即被称为 window。流计算领域的许多概念、系统设计、实现细节都围绕着 window 展开，window 的支持程度也是我们对众多的流计算框架进行选型的重要依据。要理解 window，我们需要回答以下关键问题：

1. 如何根据window进行数据划分？
1. 如何定义在window相关的操作中涉及的时间概念？
1. 如何输出基于window的操作的处理结果？

下文将逐一回答这些关键问题，并在最后比较当前较热门的各流计算框架对 window 的支持程度。

<!--more-->

## 如何根据 window 进行数据划分？

由于流计算中的数据一般具有时间上的先后关系，window 切分一般基于数据的时间关系进行，常见的 window 划分策略包括（见图1[^image1-copyright]）：

- Tumbling window（也称为 fixed window）。Tumbling window 按固定的时间间隔对数据进行划分，将同一时间区间内的数据归为同一 window，每个数据只会属于一个 window；
- Sliding window。Sliding window 与 tumbling window 一样按固定时间间隔划分数据，每个时间区间被称为一个 window slide，而一个 window 区间可以包含多个 window slide，以固定的步长向后滑动，这意味着一个数据可以属于多个 window；
- Session window。Session window 使用数据间的时间间隔进行 window 切分，一旦前后两个数据间的时间间隔超过指定间隔，即将它们划入两个不同的 window，这种 window 划分策略适用于将数据划分为多个在时间上密切相关的 session。

![图1 window 策略](http://oo9foc9n0.bkt.clouddn.com/uchuhimo.me/blog/images/2017/04/windowing-strategies.jpg)
图 1：window 策略

除了以基于时间进行划分外，基于数量进行划分的 window 划分策略也较常见。基于数量的划分策略主要包括 tumbling window 和 sliding window，只是将里面涉及的固定时间间隔替换为固定数量，这里不再赘述。

在支持 trigger 的系统（如 Google Cloud Dataflow 和 Flink）中，用户可以自定义 window 划分策略。例如，在 Google Cloud Dataflow（以下简称 Dataflow）中，用户可以根据时间、元素数目、数据内容（如是否包含换行符）等进行组合定义新的 trigger，从而根据具体业务场景灵活划分 window。

## 如何定义在window相关的操作中涉及的时间概念？

当我们基于时间对数据进行划分时，我们是应该使用数据产生时的时间，还是数据进入系统时的时间，抑或是数据被某一操作处理时的时间？鉴于 window 操作涉及的时间概念如此复杂多样，流计算系统一般允许用户指定时间策略。常见的时间策略包含以下几种：

- Event time。Event time 指以数据自带的时间信息为作为时间基准，由于许多实时数据带有时间戳（如物联网设备的定时心跳数据），而围绕数据自带时间戳进行数据处理也是许多日志处理逻辑、周期统计逻辑的关键所在，event time 在流处理领域有着非常广泛的应用；
- Processing time。Processing time 指以数据实际被处理的时间作为时间基准，由于实现简单，几乎所有流计算系统都提供支持，但由于数据什么时候被处理依赖于具体的系统执行情况，processing time 基本只用于 event time 不可用的情况（如数据不带时间信息、系统不支持 event time 等）或与 event time 配合使用；
- Ingress time。Ingress time 指以数据进入流计算系统的时间作为时间基准，由于可以应用于无时间戳的数据、系统处理逻辑上类似于 event time 且能一定程度上避免 event time 数据乱序的问题，ingress time 在特定场景下可作为 event time 的合理替代方案。

由于 event time 能最精确地描述大量业务场景下的时间概念，支持 event time 能极大提升流计算系统的业务支持能力。但 event time 是流计算领域的“潘多拉魔盒”，包含许多复杂而微妙的问题。由于 event time 和数据处理时的时刻没有顺序对应关系，因此需要处理数据乱序问题。我们需要在下一个 window 的数据已经到达的情况下继续保持上一 window，以等待乱序数据的到达。由于数据集是无界的，而乱序数据什么时候到达并没有任何的保证，因此可能需要保持住所有的 window，这显然是不现实的。解决这一问题的常见方案是使用 watermark，即在数据源发送数据的过程中，在适当的位置插入 watermark 时间戳，表示在该 watermark 之后到达的数据中不包含早于时间戳所标记的时刻之前的数据，如此一来在接收到 watermark 时，早于 watermark 时间戳的 window 就能够被处理和清理。但在实际情况中，watermark 往往是不精确的，即在 watermark 到达后仍会有早于该时间戳的数据到达，造成 late data 问题。而 late data 的处理方式在不同系统中千差万别，存在着直接丢弃、更新之前 window 的状态、覆盖之前 window 的状态等多种处理方式，这里不再一一详述。

## 如何输出基于window的操作的处理结果？

由于不同系统的 window 切分策略以及对 late data 的处理策略不同，同一 window 可能会出现一或多个输出结果的情况。在不支持 trigger 的系统中，如果直接丢弃 late data 或使用 late data 持续更新 window 输出结果，那么通常 window 处理结果是单输出的。一旦支持 trigger，则可能会出现在一个 window 中多次触发输出的情况，而使用 late data 覆盖或追加先前 window 结果的策略也会导致在 late data 出现时 window 多输出的情况。除了 Dataflow 和 Flink 外，现存的大部分系统是 window 单输出的。

## 各流计算框架对window支持程度对比

下表总结了各热门的流计算框架对window支持程度的对比：

| | Google Cloud Dataflow | Apache Flink | Apache Storm | Spark Structured Streaming | Spark Streaming | Kafka Streams
- | - | - | - | - | - | -
版本 | --[^dataflow-version] | 1.1 | 1.0.0 | Alpha | 2.0.0 | 0.10.0
Tumbling window | **支持** | **支持** | **支持** | **支持** | **支持** | **支持**
Sliding window | **支持** | **支持** | **支持** | **支持** | **支持** | **支持**
Session window | **支持** | **支持** | _不支持_ | _不支持_ | _不支持_ | _不支持_
Count-based window | **支持** | **支持** | **支持** | _不支持_ | _不支持_ | _不支持_
Trigger | **支持** | **支持** | _不支持_ | _不支持_ | _不支持_ | _不支持_
Event time | **支持** | **支持** | **支持** | **支持** | _不支持_ | **支持**
Processing time | **支持** | **支持** | **支持** | _不支持_ | **支持** | **支持**
Ingress time | _不支持_ | _不支持_ | _不支持_ | _不支持_ | _不支持_ | **支持**
Window输出 | 多输出 | 多输出 | 单输出 | 单输出 | 单输出 | 单输出

可以看到，目前对 window 支持最完善的流计算框架是 Dataflow 和 Flink；Storm 在最新的 1.0.0 中加入内置的 window 支持，而 Kafka Streams 则是随 Kafka 0.10.0 版本一同释出的新流计算框架，两者对 window 的支持都较为完善，但是否稳定可靠则需要时间的验证；而 Spark 原有的流计算框架 Spark Streaming 由于只支持基于 mini-batch 的 processing-time window，对 window 的支持一直很薄弱，随 Spark 2.0.0 释出的 Structured Streaming 旨在支持 event-time window，补足 Spark 在流处理上的能力，但仍然处于相当早期的阶段，各种 window 特性的支持都有所缺失。鉴于流处理领域对于优秀流处理框架的旺盛需求，未来一定会有更多对 window 支持更完善的流处理框架涌现出来，让我们拭目以待。

## 参考链接

- [The world beyond batch: Streaming 102](https://www.oreilly.com/ideas/the-world-beyond-batch-streaming-102)
- [Apache Flink 1.1 Documentation: Windows](https://ci.apache.org/projects/flink/flink-docs-master/dev/windows.html)
- [Concepts — Confluent Platform 3.0.0 documentation](http://docs.confluent.io/3.0.0/streams/concepts.html)
- [Developer Guide — Confluent Platform 3.0.0 documentation](http://docs.confluent.io/3.0.0/streams/developer-guide.html)
- [Windowing Support in Core Storm](http://storm.apache.org/releases/1.0.0/Windowing.html)
- [Spark Streaming - Spark 2.0.0 Documentation](http://spark.apache.org/docs/latest/streaming-programming-guide.html)
- [Structured Streaming Programming Guide - Spark 2.0.0 Documentation](http://spark.apache.org/docs/latest/structured-streaming-programming-guide.html)

[^image1-copyright]: 本图摘自 https://www.oreilly.com/ideas/the-world-beyond-batch-streaming-102，版权归原作者所有
[^dataflow-version]: Dataflow 为 Google 提供的云服务，由于没有开源，并不清楚其内部版本号
