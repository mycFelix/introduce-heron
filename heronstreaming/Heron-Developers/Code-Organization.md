# [Heron 代码结构](http://twitter.github.io/heron/docs/contributors/codebase/)

本小节为初识 Heron 、想为 Heron 共享代码的开发者提供了 Heron 代码库的基本信息。Heron 代码库托管在 [github](https://github.com/twitter/heron) 上。

## 编程语言 (Languages)

Heron 使用 C++、Java 和 Python 编写而成。

* **C++ 11** 用于很多 Heron 的核心组件之中，包括 [Topology Master](../Heron-Concepts/Heron-Architecture.md#topology-master) 、
[Stream Manager](../Heron-Concepts/Heron-Architecture.md#stream-manager) 等。

* **Java 8** 用于很多 Heron 的基础 API 之中，以及 [Heron Instance](../Heron-Concepts/Heron-Architecture.md#heron-instance) 组件的编写。目前只能用 Java 语言编写拓扑，相关指南请参考[编写拓扑](../Topology-Writers/Writing-Java-Topologies.md)小节以及 [Javadoc](http://twitter.github.io/heron/api/com/twitter/heron/api/topology/package-summary.html)。另外，也可以使用 Java 7 或者之后的版本编写 Heron 拓扑，Java 8 并不是编写拓扑的必须项。

* **Python 2** (特别是 2.7) 用于编写 [Heron CLI](../User-Manuals/Heron-Client.md) 工具以及 UI 组，如 [Heron UI](../User-Manuals/Heron-UI-Runbook.md) 和 [Heron Tracker](../User-Manuals/Heron-Tracker-Runbook.md)。

## 主要工具 (Main Tools)

* **Build tool** --- Heron 使用 [Bazel](http://bazel.io/) 作为项目构建工具。相关安装、配置信息可以参考[编译 Heron](../Heron-Developers/compiling.md) 小节。

* **Inter-component communication** --- Heron 使用 [Protocol Buffers](https://developers.google.com/protocol-buffers/?hl=en) 做为组件间的通讯协议。绝大多数 `.proto` 定义可以在 [`heron/proto`](https://github.com/twitter/heron/tree/master/heron/proto) 文件夹下找到。

* **Cluster coordination** --- Heron 依赖 Zookeeper 做为集群分布式部署时的协调器。更多的 Zookeeper 信息可以参看 [State
Management](#state-management) 小节。

## 公共组件 (Common Utilities)

[`heron/common`](https://github.com/twitter/heron/tree/master/heron/common) 定义了 Heron 所需的、用于各种编程语言的组件，其中包括：公共常量、文件操作工具、网络通信接口等。

## 集群调度器 (Cluster Scheduling)

[`heron/schedulers`](https://github.com/twitter/heron/tree/master/heron/schedulers) 定义了 Heron 目前支持的所有调度器，他们都是用 Java 编程语言开发的。***笔者注释：因本小节文档略落后与版本迭代，此处翻译略有删减***

## 状态管理器 (State Management)

这部分代码在 [`heron/statemgrs`](https://github.com/twitter/heron/tree/master/heron/statemgrs) 文件夹中，它们大部分都与 [ZooKeeper](http://zookeeper.apache.org/) 相关。分别在 `src/${languages}` 文件夹下可以找到每种编程语言与 Zookeeper 的通信接口，它们被广泛的用于绝大部分 Heron 组件之中。

## 拓扑组件 (Topology Components)

### Topology Master

[Topology Master](../Heron-Concepts/Heron-Architecture.md#topology-master) 的 C++ 源代码在 [`heron/tmaster`](https://github.com/twitter/heron/tree/master/heron/tmaster) 文件夹中。

### Stream Manager

[Stream Manager](../Heron-Concepts/Heron-Architecture.md#stream-manager) 的 C++ 源代码在 [`heron/stmgr`](https://github.com/twitter/heron/tree/master/heron/stmgr) 文件夹中。

### Heron Instance

[Heron Instances](../Heron-Concepts/Heron-Architecture.md#heron-instance) 的 Java 源代码在 [`heron/instance`](https://github.com/twitter/heron/tree/master/heron/instance) 文件夹中。

### Metrics Manager

[Metrics Manager](../Heron-Concepts/Heron-Architecture.md#metrics-manager) 的 Java 源代码在 [`heron/metricsmgr`](https://github.com/twitter/heron/tree/master/heron/metricsmgr) 文件夹中。

## 开发 APIs (Developer APIs)

### Topology API

Heron 用于编写拓扑的 API 的 Java 源代码在 [`heron/api`](https://github.com/twitter/heron/tree/master/heron/api) 文件夹中。

### Simulator

Heron 提供了用于进行调试的 [`Simulator`](../../developers/simulator-mode) 模拟器。它的 Java API 可参看 [`heron/simulator`](http://twitter.github.io/heron/api/com/twitter/heron/simulator/package-summary.html)。

### Example Topologies

Heron 代码库中还包含了所有用 Heron API 编写的示例拓扑。它们的 Java 源代码在 [`heron/examples`](https://github.com/twitter/heron/tree/master/heron/examples) 文件夹中。

## 用户交互组件 (User Interface Components)

### Heron CLI

Heron 自带一个 `heron` CLI 命令，用于[管理集群拓扑](../User-Manuals/Heron-Client.md)或者是展现集群拓扑的一些细节。它的 Python 源代码在 [`heron/tools/cli`](https://github.com/twitter/heron/tree/master/heron/tools/cli) 文件夹下。

### Heron Tracker

[Heron Tracker](../User-Manuals/Heron-Tracker-Runbook.md) 的 Python 源代码在 [`heron/tools/tracker`](https://github.com/twitter/heron/tree/master/heron/tools/tracker) 文件夹下。

Tracker 用 Python 编写 Web 服务(web server)。它依赖于 [Tornado](http://www.tornadoweb.org/en/stable/)  框架。您可以在 [`main.py`](https://github.com/twitter/heron/tree/master/heron/tools/tracker/src/python/main.py) 和 [`handlers`](https://github.com/twitter/heron/tree/master/heron/tools/tracker/src/python/handlers) 目录中为 Tracker 添加新的 HTTP 请求处理。

### Heron UI

[Heron UI](../User-Manuals/Heron-UI-Runbook.md) 的 Python 源代码在 [`heron/tools/ui`](https://github.com/twitter/heron/tree/master/heron/tools/ui) 文件夹中。

与 Heron Tracker 一样，Heron UI 也是用 Python 编写的 Web 服务(web server)。它同样依赖于 [Tornado](http://www.tornadoweb.org/en/stable/) 框架。您可以在 [`main.py`](https://github.com/twitter/heron/tree/master/heron/web/tracker/src/python/main.py) 和 [`handlers`](https://github.com/twitter/heron/tree/master/heron/web/tracker/src/python/handlers) 目录中为 UI 添加新的 HTTP 请求处理。

## Tests

Heron 单元测试用例贯穿整个代码库中。更多信息请参考[运行 Heron 单元测试](../Heron-Developers/Running-Tests.md)
