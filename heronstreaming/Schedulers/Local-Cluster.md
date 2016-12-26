# [Local Cluster](http://twitter.github.io/heron/docs/operators/deployment/schedulers/local)

为了让开发者更好的了解 Heron 的新特性，本地测试新功能，Heron 为我们提供了单机伪分布式集群部署方案。

以下两种状态管理器都可以被用在 Local Cluster 模式中：

* [Local File System](../State-Managers/Local-FS.md)
* [Zookeeper](../State-Managers/Zookeeper.md)

**注意**：伪分布式环境并不会影响 [simulator mode](http://twitter.github.io/heron/docs/developers/simulator-mode/) 的正常工作。模拟器可以在集群环境中模拟 JVM 进程，以供调试，而伪分布式环境仅仅是运行在单一节点而已。

## How Local Deployment Works

使用本地调度器(local scheduler)的方法与使用其他调度器类似，都是调用 Heron 命令行对拓扑进行部署和管理，核心区别仅仅是配置不同。

## Local Scheduler Configuration

可以在 `scheduler.yaml` 文件中设置以下参数来让 Heron 在 Local Cluster 模式下工作。

* `heron.class.scheduler` --- 设置 `Scheduler` 的加载类为 `com.twitter.heron.scheduler.local.LocalScheduler`

* `heron.class.launcher` --- 设置 `Launcher` 的加载类为 `com.twitter.heron.scheduler.local.LocalLauncher`

* `heron.scheduler.local.working.directory` --- 设置 `Scheduler` 本地工作路径。该路径会保存 topology jar、Heron 核心可执行文件、拓扑日志等重要信息。

* `heron.package.core.uri` --- Heron 核心可执行文件的 uri 路径 ***笔者注释：在最新的 Release 版本中已去除该参数***

* `heron.directory.sandbox.java.home` --- 设置 `${JAVA_HOME}`

---

***笔者后记***

其实笔者关于 Local Cluster 模式有很多话要说。该模式有诸多益处：

* 它是学习 Heron 命令行工具的最佳途径。在这个环境下我们并不担心会做错什么。
* 它可能是学习 Heron 运行机理的最佳选择之一。我们可以通过查看 `heron.scheduler.local.working.directory` 的目录和文件了解 Heron 的运行过程，分析 Heron Instance 的日志内容。
* 如想要开发 Heron 核心组件，Local Cluster 是最棒的测试平台。开发这样的大型分布式计算框架，没有什么比本地开发，本地编译，本地测试更能让程序员感到舒爽的了。当然，本地测试通过后，集群测试也是必不可少的环节。
* 可以很方便的将 Local Cluster 模式变成拓扑的测试环境。我们编写好了一个拓扑，在正式提交之前可以在 Local Cluster 模式下先进行功能验证，而后再提交集群。当然，功能验证的方式方法还有很多，比如编写逻辑覆盖完全的`单元测试样例`就是一个非常良好的习惯。

总之，Local Cluster 是我们的好朋友，是 Heron 初学者必备佳品，也是 Heron 老司机的忠实伴侣。
