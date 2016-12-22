# Heron 架构

***笔者注释：这应该是最难翻译的一篇了。笔者会尽自己所能展现原文语义，并努力争取得到 Heron Core Committer 的校对***

Heron 是继 [Apache Storm](http://storm.apache.org) 之后的下一代实时计算框架。它全面颠覆了 Storm 的架构体系，但依旧保证了对 Storm 的后向兼容性。

本小节旨在说明 [Heron 和 Storm的区别](#与-storm-的关系-relationship-with-apache-storm)、描述 [Heron 的设计目标](#设计目标-heron-design-goals)并介绍架构中的[核心组件](#拓扑组件-topology-components)。

## 代码库 (Codebase)

关于代码库的更多内容，我们在[代码库](http://twitter.github.io/heron/docs/contributors/codebase/)页面中做了更详细的介绍。

## 拓扑 (Topologies)

拓扑是处理流式数据的具体实体。通俗的讲，Heron 集群以自动化的方式管理着这些拓扑的生命周期。请参考[拓扑](../Heron-Concepts/Heron-Topology.md)页面，了解拓扑的更详细信息。

## 与 Storm 的关系 (Relationship with Apache Storm)

Heron 是继[Apache Storm](http://storm.apache.org)之后的下一代实时计算框架。在设计上，有如下两个实现目标：

1. 基于进程处理模型(process-based)替代 Storm 线程处理模型(thread-based)，在性能、可靠性以及其他方面超越 Storm。
2. 全部保留 Storm 数据模型和[拓扑API](http://storm.apache.org/about/simple-api.html)。

更多深入实现，可以参考论文 [Twitter Heron:Stream Processing at Scale](http://dl.acm.org/citation.cfm?id=2742788)

## 设计目标 (Heron Design Goals)

请参考 [Heron 设计目标](../Heron-Concepts/Heron-Topology.md)章节，全面了解 Heron 的设计理念。

## 拓扑组件 (Topology Components)

Heron 拓扑有如下核心组件，本小节会依次对其进行深入的介绍：

* [Topology Master](#topology-master)
* [Container](#container)
* [Stream Manager](#stream-manager)
* [Heron Instance](#heron-instance)
* [Metrics Manager](#metrics-manager)
* [Heron Tracker](#heron-tracker)

***笔者注释：为减少歧义，在文中会保持核心组件的英文名称。***

### Topology Master

Topology Master(简称 TM)是 Heron 的重要组件之一，它的主要职责是管理拓扑的生命周期。从拓扑提交开始，一直到拓扑被杀死，TM 的作用始终贯穿其中。Heron 部署拓扑时，会启动一个独立的 TM 和多个 [containers](#container)。 TM 会在 [ZooKeeper](http://zookeeper.apache.org) 上创建一个节点(node)用来保证同拓扑只有一个 TM 实例运行，同时该实例也可以被拓扑内的任何进程轻松定位。TM 还负责基于不同组件所需构建拓扑的[物理执行计划](../Heron-Concepts/Heron-Topology.md#物理执行计划)。

![Topology Master](http://twitter.github.io/heron/img/tmaster.png)

#### Topology Master 的配置 (Topology Master Configuration)

TM 有大量的可配置参数贯穿拓扑的整个生命周期，你可以在[配置参数](http://twitter.github.io/heron/docs/operators/configuration/tmaster/)小节中找到相应的答案。

### Container

每一个 Heron 拓扑包含若干个 **containers**。其中每一个 container 内又包含着多个 [Heron Instances](#heron-instance)，1 个 [Stream Manager](#stream-manager) 和 1 个 [Metrics Manager](#metrics-manager)。Container 负责与 TM 通信，以确保拓扑的有向无环图可以正常工作。

***笔者注释：container 的概念使得 Heron 可以快速的部署在任何一个基于资源共享的调度系统中。***

### Stream Manager

**Stream Manager** (简称 SM)负责管理拓扑组件之间 tuple 的路由关系。拓扑中的每一个 [Heron Instance](#heron-instance) 绑定到本地的 SM 实例，通过网络将拓扑所需所有的 SM 实例链接起来。下图展现了 SM 的网络结构：

![Heron Data Flow](http://twitter.github.io/heron/img/data-flow.png)

作为一个数据路由引擎，SM 负责在拓扑需要的时候启动背压机制。下图描述了背压(back pressure)状况：

![Back Pressure 1](http://twitter.github.io/heron/img/backpressure1.png)

如上图所示，假定 bolt **B3** (在container **A** 中)接收 spout **S1** 的所有输出，此时与其他组件相比，**B3** 运行的略慢(标红)一些。container **A** 中的 SM 组件感知到了 **B3** 有处理压力，它会拒绝从其他 container 如 **C** 和 **D** 的 SM 中接收数据，这时可能会撑爆其他 containers 的缓存区，引起系统异常(throughput collapse).

类似这种情况，Heron 的背压机制会自动介入。container **A** 中的 SM 会广播消息通知其他的 SM。然后 SM 们会根据物理执行计划计划展开自检，并暂时切断 bolt **B3** 的数据流入(也就是图中的spout **S1**)。

![Back Pressure 2](http://twitter.github.io/heron/img/backpressure2.png)

一旦处理较慢的 bolt **B3** 重新开始正常工作，container **A** 中的 SM 会通知其他的 SM 恢复正常的数据流作业。

***笔者注释***

```
针对 Heron 的背压机制可以打个比方就是：上游的兄弟们，我这儿快忙不过来了，你们先歇会儿，等我忙完了，你们在发数据。

“由谁来感知“忙不过来呢？”  -- 由当前 container 所运行SM。

“用什么样的方式来告知兄弟们呢？” -- 由 SM 来告知它兄弟们。

“由谁来暂时切断数据流入呢？” -- 由当前 container 所运行SM。

直观来看，背压机制有效的防止了拓扑的因缓冲区爆满而导致异常崩溃。但同时要意识到，一旦触发了背压机制，可能意味着这段时间内拓扑的处理量已经达到系统瓶颈，应做适当调整。
```

#### Stream Manger 的配置 (Stream Manger Configuration)

与 Topology Master 一样，Stream Manger 也有很多配置参数供开发选择。可参考[配置参数](http://twitter.github.io/heron/docs/operators/configuration/tmaster/)小节

### Heron Instance

一个 **Heron Instance**(简称 HI) 具体对应处理一个 [spout](../Heron-Concepts/Heron-Topology.md#spouts) 或是一个 [bolt](../Heron-Concepts/Heron-Topology.md#bolts)。通过这种方式，可以让开发者调试和调优。

目前，Heron 仅支持 Java，所以所有的 HI 都是一个独立的 [JVM] (https://en.wikipedia.org/wiki/Java_virtual_machine)进程，未来会支持更多语言。

***笔者注释：Heron 的 0.14.5 版本已经支持 Python on Heron 了***

#### Heron Instance 配置 (Heron Instance Configuration)

Heron Instance 有很多配置参数供开发选择。可参考[配置参数](http://twitter.github.io/heron/docs/operators/configuration/tmaster/)小节

### Metrics Manager

每一个拓扑还会运行一个 Metrics Manager 实例，用来收集和导出 [container](#container) 中各个组件的运行时信息。它也会与 [Topology Master](#topology-master) 通信，同时还支持其他采集器系统如：[Scribe](https://github.com/facebookarchive/scribe)，[Graphite](http://graphite.wikidot.com/)。

你也可以根据[自定义 metrics](http://twitter.github.io/heron/docs/contributors/custom-metrics-sink) 章节的指引自定义相关开发。

## 集群级别的组件 (Cluster-level Components)

之前所描述的组件运行在每一个拓扑之中，而接下来我们要介绍的集群级别(cluster-level)的组件用于集群的功能管理。

### Heron CLI

Heron CLI 工具的命令符为 `heron` ，相关文档可以查阅[管理拓扑](http://twitter.github.io/heron/docs/operators/heron-cli/)小节.

### Heron Tracker

Heron Tracker (或者简称 Tracker) 可以理解为集群范畴(cluster-wide)的拓扑信息获知器，如拓扑是否正常运转、是否已启动、是否已被杀死等等。它依赖于拓扑暴露的 [ZooKeeper](http://zookeeper.apache.org) 节点(node)获知信息，同时对外提供了 JSON REST API 以供外部调用。Tracker 既可以被部署在 Heron 集群之中，也可以独立于 Heron 集群之外。

关于 Tracker 的 JSON API 文档可以参考 [Heron Tracker](http://twitter.github.io/heron/docs/operators/heron-tracker/) 小节。

### Heron UI

**Heron UI** 是拓扑可视化信息展现页面。通过 Heron UI 上面的颜色区(color-coded)来展现集群上每一个拓扑的[逻辑](../Heron-Concepts/Heron-Topology.md#逻辑执行计划-logical-plan)和[物理](../Heron-Concepts/Heron-Topology.md#物理执行计划-physical-plan)计划

了解更多信息，请参考 [Heron UI](http://twitter.github.io/heron/docs/operators/heron-ui/) 页面。

## 拓扑提交时序 (Topology Submit Sequence)

在[拓扑生命周期](../Heron-Concepts/Heron-Topology.md#拓扑生命周期-topology-lifecycle)小节描述了 Heron 拓扑的各个状态。下图展现了 Heron 架构中的各个组件在执行 `submit` 和 `deactivate` 时的客户端交互时序图。同时，也揭示了拓扑何时能在 Heron UI 页面上得到展现。

<!--
The source for this diagram lives here:
https://docs.google.com/drawings/d/10d1Q_VO0HFtOHftDV7kK6VbZMVI5EpEYHrD-LR7SczE
-->
<img src="http://twitter.github.io/heron/img/topology-submit-sequence-diagram.png" style="max-width:140%;!important;" alt="Topology Sequence Diagram"/>

### 拓扑提交流程简述 (Topology Submit Description)

下面简述一下在使用本地调度器(local scheduler)的条件下，拓扑在提交和启动时是如何工作的。

* 客户端 (Client)

   当使用 `heron submit` 命令提交拓扑时，系统会首先执行拓扑中的 `main` 方法，随后会创建一个包含着拓扑逻辑执行计划的 `.defn` 文件。这时，系统会运行 `com.twitter.heron.scheduler.SubmitterMain` ，它负责为拓扑调用 uploader 和 launcher。uploader 会把拓扑的可执行包上传到指定路径。同时 launcher 也会向 State Manager 注册拓扑的逻辑计划和运行状态，并调用主调度器 main scheduler。


* 共享服务 (Shared Services)

    当 main scheduler (`com.twitter.heron.scheduler.SchedulerMain`) 被 launcher 调用后，它会从拓扑存储区 (topology storage) 获取拓扑信息 (topology artifact，直译)，初始化 State Manager，准备物理执行计划(用来确定哪几个 instances 归属于哪一个 container)。然后，它会运行指定的 scheduler 如 `com.twitter.heron.scheduler.local.LocalScheduler` 以供每一个 container 执行 `heron-executor` 命令。

* 拓扑 (Topologies)

    `heron-executor` 用于启动每一个 container，同时也负责启动分配给 container 的 Topology Master 或 Heron Instances (Bolt/Spout)。需要注意的是目前，Topology Master 永远被分配给 container 0。当 `heron-executor` 执行到对应的 Heron Instances 时，它总先启动 Stream Manager 和 Metrics Manager，再通过调用 `com.twitter.heron.instance.HeronInstance` 启动每一个对应的实例。

    每一个 Heron Instance 有两个线程：gateway 和 salve 线程。Gateway 线程主要负责与 Stream Manager 和 Metrics Manager 通信，Stream Manager 和 Metrics Manager 分别使用 `StreamManagerClient` and `MetricsManagerClient` 来响应 gateway 线程，也负责接收和发送来自 slave 线程的 tuple 信息。另外，根据物理执行计划创建的每一个 spout 或 bolt 都会持有一个 slave 线程。

    当 Heron Instance 运行后，`StreamManagerClient` 会向 stream manager 注册并与其建立一个稳定的链接。注册成功后，gateway 线程开始发送物理计划给 slave 线程，然后运行相应的已分配的实例。 ***笔者注释：关于双线程处理模型 [Twitter Heron: Stream Processing at Scale](http://dl.acm.org/citation.cfm?id=2742788) 中有非常详尽的描述。***

---

***笔者后记***

本小节应该说是 Heron 的核心部分，如果想更深入的了解相关信息，强烈建议阅读这篇论文 [Twitter Heron: Stream Processing at Scale](http://dl.acm.org/citation.cfm?id=2742788)。从文中可以看到，Heron 的整体架构相较于 Storm，可以说是脱胎换骨。首先架构中没有了 Nimbus，取而代之的是利用 Zookeeper 来管理拓扑。同时对于每一个拓扑，都有一个自己的 Topology Master 来贯穿自己的生命周期。TM 的设计思想可以和 YARN 中的 Application Master 类似但也不尽相同。然后提出了 container 概念，可在其中运行多个 Heron Instance，做到资源限制，同时也使得 Heron 可以近乎无缝的接入到各个共享资源调度系统中，如 Mesos、Yarn。再次使用独立的进程来负责 spout 或 bolt 正常运转，大大的方便了开发人员的调试和调优，也实现其设计目标中的隔离理念。最后 Stream Manager 的提出可以让拓扑之间的数据流转更加灵活，当然也有声音说，Stream Manager 有可能成为系统的性能瓶颈，笔者认为这确实可能是一个缺点，但并不能完全掩盖 Heron 带来的闪光点。
