# Heron 拓扑 (Heron Topology)

***笔者注释：如果对 Storm 有了解的同学可以跳过这一章节，Heron Topology 和 Storm Topology 并无本质差别***

Heron 拓扑本质上是一个用于流式数据处理的[有向无环图](https://en.wikipedia.org/wiki/Directed_acyclic_graph)。Heron 拓扑包括三个基本概念：[spouts](#spouts) and [bolts](#bolts)，以及建立二者之间流式关系的 [tuples](http://twitter.github.io/heron/docs/developers/data-model)。 下面这张图展现了一个简单的拓扑：

![Heron topology](http://twitter.github.io/heron/img/topology.png)

Spout 负责将 tuple 发送到拓扑中，bolt 负责接收并处理这些 tuple 。如上图所示，spout S1 发送 tuple 至 bolt B1 和 B2 用于后续处理；同时 B1 将处理后的数据结果已 tuple 形式发送给 bolt B3 和 B4 。同理，B2 将处理后的 tuple 发送给 bolt B4.

这只是一个简单的示例。你可以任意组合 spout 和 bolt 构建复杂的拓扑。***笔者注释：spout 可以理解为数据源，bolt 则负责具体处理数据。拓扑的构建一定是从 spout 到 bolt 的有向无环图***

## 拓扑生命周期 (Topology Lifecycle)

在正确配置 [Heron cluster](http://twitter.github.io/heron/docs/operators/deployment) 后，你就能通过 Heron [CLI tool](http://twitter.github.io/heron/docs/operators/heron-cli) 来管理拓扑了 ***笔者注释：Heron UI 可以用来进行拓扑状态监控，但目前还没有提供可视化的拓扑状态管理的入口。目前只能通过命令行的方式来管理拓扑状态*** 。拓扑可能的生命周期如下所示：

1. 提交([submit](http://twitter.github.io/heron/docs/operators/heron-cli#submitting-a-topology))拓扑至集群。此时拓扑已随时做好被激活的准备。

2. 激活([activate](http://twitter.github.io/heron/docs/operators/heron-cli#activating-a-topology))拓扑。拓扑会按照你的构建方式进行数据处理或计算。

3. 重启([restart](http://twitter.github.io/heron/docs/operators/heron-cli#restarting-a-topology))拓扑。如果你需要更新某些拓扑配置，比如一些新的参数，你可以选择重启拓扑。

4. 暂停([deactivate](http://twitter.github.io/heron/docs/operators/heron-cli#deactivating-a-topology))拓扑。拓扑会停止计算，但还会驻留在集群中。

5. 杀死([kill](http://twitter.github.io/heron/docs/operators/operators/heron-cli#killing-a-topology))拓扑。执行此命令后，集群会把拓扑移除。拓扑一旦被 kill 若想重新运行相关计算，只能重新提交(re-submit)。

## Spouts

Heron **spout** 是整个流处理的数据源，负责生成 tuples。例如，spout 会从 Kestrel 队列或者是 Twitter 读取推文的 API 中读取数据并将其转化为 tuple 发送给后续的 bolt。***笔者注释：这不仅仅是 Heron spout 的概念，Strom spout 也是同样的理念***

可以参考[构建 Spouts](http://twitter.github.io/heron/docs/developers/java/spouts) 页面，了解更多的相关信息。

## Bolts

A Heron **bolt** 从 spout 端消费 tuple 数据流。用户可以在 bolt 中定义任何处理算法，例如：数据转换，持久化存储，聚合多流数据或者是发送 tuple 给其他 bolt 进行后续处理，等等。***笔者注释：这不仅仅是 Heron bolt 的概念，Strom bolt 也是同样的理念***

可以参考[构建 Bolt](http://twitter.github.io/heron/docs/developers/java/bolts) 页面，了解更多的相关信息。


## 数据模型 (Data Model)

Heron 构建在基于 tuple 驱动的数据模型之上。可参考[Heron 数据模型](http://twitter.github.io/heron/docs/developers/data-model) 页面了解更多信息。

## 逻辑执行计划 (Logical Plan)

拓扑的**逻辑执行计划**和数据库的[查询计划](https://en.wikipedia.org/wiki/Query_plan)类似。本文中的第一张图可以理解为拓扑的逻辑执行计划展现图。

## 物理执行计划 (Physical Plan)

拓扑的**物理执行计划**和逻辑执行计划相关但不相同。其核心区别是：物理计划是拓扑的逻辑计划的物理映射，更通俗的讲，是有多少台机器正在运行着多少个 spout 和 bolt 。关于物理执行计划的更直观表达可以参看下图：

![Topology Physical Plan](http://twitter.github.io/heron/img/physicalplan.png)

***笔者注释：Heron 现在支持开发者可以通过命令行的方式查看物理执行计划，比如 spout 的并行度是多少，占用了系统资源等等***

---

***笔者后记***

本小节内容同样适用于 Storm 拓扑。对于拓扑的开发者来讲，不仅仅要在编写代码时准确的构建**逻辑执行计划** ，还需要在运行拓扑时调整**物理执行计划**。逻辑执行计划相当于整个拓扑的数据从输入到输出的整个流转过程，首先要在逻辑上能够覆盖你得计算。物理执行计划相当于如何通过资源的分配，来让这个拓扑运行的更好更快。比如，在编写拓扑时要想到，某个数据处理环节是否可以通过提高并行度方式加速处理，比如在一些聚合操作上，是否满足这样的需求；有没有可能因为数据流转分组不均，造成绝大多数数据分摊到了某个机器上。上述举例虽然属于物理执行计划的范畴，但是其本质是在拓扑的设计环节没有很好的构造**逻辑执行计划**。
