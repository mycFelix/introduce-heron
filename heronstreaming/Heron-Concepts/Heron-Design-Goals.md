# Heron 设计目标 (Heron Design Goals)

***笔者注释：这是比较专业的且核心的部分，笔者尽力还原原文语义，如有偏差还请指出***

下面主要介绍一系列 Heron 的核心设计目标：

### 隔离 (Isolation)

跳出拓扑基于线程的处理思路，争取让拓扑之间的每一步处理过程相互隔离，以便调试(debugging)、展现(profiling)、问题定位(troubleshooting)。

### 资源限制 (Resource constraints)

拓扑仅会在自己申请的资源区域内运行，并对拓扑设置明确的资源上界。这样 Heron 可以在资源共享的架构体系(shared infrastructure)中安全的运行。

### 兼容性 (Compatibility)

Heron 应该在 API 和数据模型上兼容 [Apache Storm](http://storm.apache.org)，这样可以让开发者更从容的切换。

### 背压机制 (Back pressure)

像 Heron 这样的分布式系统，很难保证系统中的每一个组件都按照同一速度齐头并进。Heron 已经提供了一个背压机制来确保当有组件延迟时，系统能够自适应的做出响应调整。 ***笔者注释：这部分内容会在 Heron 架构中具体讨论***

### 性能 (Performance)

Heron 的设计者们期望 Heron 能够比 Storm 提供更高的吞吐量和更低的延迟率，同时对此还提供可配置选项，供开发者调优。

### 计算语义 (Semantic guarantees)

Heron 支持[最多一次和最少一次(at-most-once and at-least-once)](https://kafka.apache.org/08/design.html#semantics) 的计算语义

### 效率 (Efficiency)

Heron 希望尽可能使用最少的资源来实现以上所有目标。

---

***笔者后记***

这是比较难翻译的一篇，有些地方加入了笔者的理解，可能也会有一些偏颇。如想更深入的了解 Heron 的设计目标以及动机(motivation)，可以了解一下 Heron 的论文：[Twitter Heron: Stream Processing at Scale](http://dl.acm.org/citation.cfm?id=2742788)。也欢迎大家就本文的翻译内容进行讨论。
