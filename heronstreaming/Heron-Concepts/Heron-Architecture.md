# Heron 架构

***笔者注释：这应该是最难翻译的一篇了。笔者会尽自己所能展现原文语义，并努力争取得到 Heron Core Committer 的校对***

Heron 是继[Apache Storm](http://storm.apache.org)之后的下一代实时计算框架。Heron 全面颠覆了 Storm 的架构体系，但是依旧保证了对 Storm 的后向兼容性。

本小节旨在说明 [Heron 和 Storm](#与 Storm 的关系)的区别、描述 [Heron 的设计理念]("#设计目标")并介绍架构中的[核心组件](#拓扑组件)。

## 代码库 (Codebase)

关于代码库的更多内容，我们在[代码库](http://twitter.github.io/heron/docs/contributors/codebase/)页面中做了更详细的介绍。

## 拓扑 (Topologies)

拓扑是处理流式数据的具体实体。通俗的讲，Heron 集群以自动化的方式管理着这些拓扑的生命周期。请参考[拓扑](../Heron-Concepts/Heron-Topology.md)页面，了解拓扑的更详细信息。

## 与 Storm 的关系 (Relationship with Apache Storm)

Heron 是继[Apache Storm](http://storm.apache.org)之后的下一代实时计算框架。在设计上，有如下两个实现目标：

1. 基于进程处理模型(process-based)替代 Storm 线程处理模型(thread-based)，在性能、可靠性以及其他方面超越 Storm。
2. 全部保留 Storm 数据模型和[拓扑API](http://storm.apache.org/about/simple-api.html)。

更多深入实现，可以参考论文：[Twitter Heron:Stream Processing at Scale](http://dl.acm.org/citation.cfm?id=2742788)

## 设计目标 (Heron Design Goals)

请参考 [Heron 设计目标](../Heron-Concepts/Heron-Topology.md)小节，全面了解 Heron 的设计理念。

## 拓扑组件 (Topology Components)

Heron 拓扑有如下核心组件，本小节会依次对其进行深入的介绍：

* [Topology Master]({{< ref "#topology-master" >}})
* [Container]({{< ref "#container" >}})
* [Stream Manager]({{< ref "#stream-manager" >}})
* [Heron Instance]({{< ref "#heron-instance" >}})
* [Metrics Manager]({{< ref "#metrics-manager" >}})
* [Heron Tracker]({{< ref "#heron-tracker" >}})

***笔者注释：为减少歧义，在文中会保持核心组件的英文名称。***

### Topology Master

Topology Master(简称 TM)是 Heron 的重要组件之一，它的主要职责是管理拓扑的生命周期。从拓扑的提交开始，一直到拓扑被杀死，TM的作用始终贯穿其中。Heron 部署拓扑时，会启动一个独立的 TM 和多个 [containers]({{< ref "#container" >}})。 TM 会在 [ZooKeeper](http://zookeeper.apache.org) 上创建一个节点(node)用来保证对一同一个拓扑只有一个 TM 实例运行，同时该实例也可以被拓扑内的任何进程轻松定位。TM 同时负责基于不同组件所需构建拓扑的[物理执行计划](../Heron-Concepts/Heron-Topology.md#物理执行计划)。

![Topology Master](http://twitter.github.io/heron/img/tmaster.png)

#### Topology Master 的配置 (Topology Master Configuration)

TMs have a variety of [configurable
parameters](../../operators/configuration/tmaster) that you can adjust at each
phase of a topology's [lifecycle](../Heron-Concepts/Heron-Topology.md#topology-lifecycle).

TM 有大量的可配置参数贯穿拓扑的整个生命周期，你可以在[配置参数](http://twitter.github.io/heron/docs/operators/configuration/tmaster/)小节中找到相应的答案。

### Container

每一个 Heron 拓扑包含若干个 **containers**。其中每一个 container 内又包含着多个 [Heron Instances]({{< ref "#heron-instance" >}})，1 个 [Stream Manager]({{< ref "#stream-manager" >}}) 和 1 个 [Metrics Manager]({{< ref "#metrics-manager" >}})。Container 负责与 TM 通信，以确保拓扑的有向无环图可以正常工作。

***笔者注释：container 的概念使得 Heron 可以快速的部署在任何一个基于资源共享的调度系统中***

### Stream Manager

**Stream Manager** (简称 SM)负责管理拓扑组件之间 tuple 的路由关系。拓扑中的每一个 [Heron Instance]({{< ref "#heron-instance" >}}) 绑定在本地的 SM 实例，所有拓扑所需的的 SM 实例通过网络链接起来。下图展现了 SM 的网络结构：

![Heron Data Flow](http://twitter.github.io/heron/img/data-flow.png)

作为一个数据路由引擎，SM 负责在拓扑需要的时候启动背压机制。下图展现了 SM 的背压机制：

![Back Pressure 1](http://twitter.github.io/heron/img/backpressure1.png)

如上图所示，假定 bolt **B3** (在container **A** 中)接收 spot **S1** 的所有输出，此时与其他组件相比，**B3** 运行的略慢一些。container **A** 中的 SM 组件感知到了 **B3** 有处理压力，它会拒绝从其他 container 如 **C** 和 **D** 的 SM 中接收数据，这时可能会撑爆其他 containers 的缓存区，引起系统异常(throughput collapse).

类似这种情况，Heron 的背压机制会自动介入。container **A** 中的 SM 会广播消息通知其他的 SM。然后 SM 会根据物理执行计划计划展开自检，并暂时切断 bolt **B3** 的数据流入(也就是图中的spout **S1**)。

![Back Pressure 2](http://twitter.github.io/heron/img/backpressure2.png)

一旦处理较慢的 bolt **B3** 重新开始正常工作，container **A** 中的 SM 会通知其他的 SM 恢复正常的数据流作业。

***笔者注释：针对 Heron 的背压机制可以打个比方就是：上游的兄弟，咱这儿快忙不过来了，你先歇会儿，等我忙完了，你在发数据。直观来看，背压机制有效的防止了拓扑的因缓冲区爆满而导致异常崩溃。但同时要意识到，一旦触发了背压机制，可能意味着这段时间内拓扑的处理量已经达到系统瓶颈，应做适当调整。***

#### Stream Manger 的配置 (Stream Manger Configuration)

与 Topology Master 一样，Stream Manger 也有很多配置参数供开发选择。可参考 [配置参数](http://twitter.github.io/heron/docs/operators/configuration/tmaster/)小节

### Heron Instance

一个 **Heron Instance**(简称 HI) 具体对应处理一个 [spout](../Heron-Concepts/Heron-Topology.md#spouts) 或是一个 [bolt](../Heron-Concepts/Heron-Topology.md#bolts)。通过这种方式，可以让开发者调试和调优。

目前，Heron 仅支持 Java，所以所有的 HI 都是一个独立的[JVM](https://en.wikipedia.org/wiki/Java_virtual_machine)进程，未来会支持更多语言。

***笔者注释：Heron 的 0.14.5 版本已经支持 Python on Heron 了***

#### Heron Instance 配置 (Heron Instance Configuration)

Heron Instance 有很多配置参数供开发选择。可参考 [配置参数](http://twitter.github.io/heron/docs/operators/configuration/tmaster/)小节

### Metrics Manager

每一个拓扑还会运行一个 Metrics Manager 实例，用来收集和导出 [container]({{< ref "#container" >}}) 中各个组件的运行时信息。它也会与 [Topology Master]({{< ref "#topology-master" >}}) 通信，同时还支持其他采集器系统如：[Scribe](https://github.com/facebookarchive/scribe)，[Graphite](http://graphite.wikidot.com/)。

你也可以根据[自定义 metrics](http://twitter.github.io/heron/docs/contributors/custom-metrics-sink) 章节的指引自定义相关开发。

## 集群级别的组件 (Cluster-level Components)

之前所描述的组件运行在每一个拓扑之中，而接下来我们要介绍的集群级别(cluster-level)的组件用于集群的功能管理。

### Heron CLI

Heron CLI 工具的命令符为 `heron` ,相关文档可以查阅 [管理拓扑](http://twitter.github.io/heron/docs/operators/heron-cli/)小节.

### Heron Tracker

Heron Tracker (或者简称 Tracker) 可以理解为集群范畴(cluster-wide)的拓扑信息获知器，如拓扑是否正常运转、是否已启动、是否已被杀死等等。它依赖于拓扑暴露的 [ZooKeeper](http://zookeeper.apache.org) 节点(node)获知信息，同时对外提供了 JSON REST API 以供外部调用。Tracker 既可以被部署在 Heron 集群之中，也可以独立于 Heron 集群之外。

关于 Tracker 的 JSON API 文档可以参考 [Heron Tracker](http://twitter.github.io/heron/docs/operators/heron-tracker/) 小节。

### Heron UI

**Heron UI** 是拓扑可视化信息展现页面。通过 Heron UI 上面的颜色区(color-coded)来展现集群上每一个拓扑的[逻辑](../Heron-Concepts/Heron-Topology.md#logical-plan)和[物理](../Heron-Concepts/Heron-Topology.md#physical-plan)计划

了解更多信息，请参考 [Heron UI](http://twitter.github.io/heron/docs/operators/heron-ui/) 页面。

## 拓扑提交时序 (Topology Submit Sequence)

在[拓扑生命周期](../Heron-Concepts/Heron-Topology.md#topology-lifecycle)小节描述了 Heron 拓扑的各个状态。下图展现了 Heron 架构中的各个组件在执行 `submit` 和 `deactivate` 时的客户端交互时序图。同时，也揭示了拓扑何时能在 Heron UI 页面上的到展现。

<!--
The source for this diagram lives here:
https://docs.google.com/drawings/d/10d1Q_VO0HFtOHftDV7kK6VbZMVI5EpEYHrD-LR7SczE
-->
<img src="/img/topology-submit-sequence-diagram.png" style="max-width:140%;!important;" alt="Topology Sequence Diagram"/>
