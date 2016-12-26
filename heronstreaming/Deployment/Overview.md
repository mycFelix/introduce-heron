# [部署 Heron](http://twitter.github.io/heron/docs/operators/deployment)

Heron 的设计理念是可被部署在以调度系统为驱动的集群环境中。它既可以部署在共享集群(multi-tenant)上，也可以已专有集群(dedicated)形式提供服务。另外，Heron 还支持多集群部署，以便用户可以将拓扑提交到指定的集群之中。每一个 Heron 集群都会有一个独立的`调度器(scheduler)`。下图是一个典型的 Heron 部署示意图。

<br />
![Heron Deployment](http://twitter.github.io/heron/img/heron-deployment.png)
<br/>

多个组件一起协同工作，才能让 Heron 集群正常运行。他们包括：

***笔者注释：为避免歧义，组件将使用英文名描述***

* **Scheduler** --- Heron 需要一个调度器来运行拓扑。Heron 可以被部署在目前主流的大数据调度系统中，目前支持：
  * [Aurora](http://twitter.github.io/heron/docs/operators/deployment/schedulers/aurora/)
  * [Local](schedulers/local)
  * [Slurm](schedulers/slurm)
  * [YARN](../Schedulers/YARN-Cluster.md)

* **State Manager** --- Heron 状态管理器负责管理拓扑的运行时状态，包括：逻辑计划，物理计划，执行状态。Heron 目前支持一下两种状态管理器:
  * [Local File System](../State-Managers/Local-FS.md)
  * [Zookeeper](../State-Managers/Zookeeper.md)

* **Uploader** --- Heron 上传器负责上传拓扑 jar 至服务器。目前支持的上传器包括：
  * [HDFS](http://twitter.github.io/heron/docs/operators/deployment/uploaders/hdfs)
  * [Local File System](http://twitter.github.io/heron/docs/operators/deployment/uploaders/localfs)
  * [Amazon S3](http://twitter.github.io/heron/docs/operators/deployment/uploaders/s3)

* **Metrics Sinks** --- Heron 会在收集拓扑运行时的多个指标。这些指标可被导出至外部存储系统用以离线分析，目前支持：
  * `File Sink`
  * `Graphite Sink`
  * `Scribe Sink`

* **Heron Tracker** --- Tracker 是集群与外部通信的桥梁。可用它提供的 REST API 查询各种各样的信息，如：拓扑逻辑计划、拓扑物理计划还有拓扑的运行时信息等等。

* **Heron UI** --- Heron UI 为用户提供了拓扑的可视化查询界面。我们可以在 UI 上清晰的看到拓扑的逻辑计划和实际的物理执行过程，同时也可以在 UI 上查询拓扑日志，查看堆内存信息(heap dump)，调取内存直方图(memory histograms)。

---
***笔者后记***

总体来说，个人认为 Heron 的集群部署是比较简单的。

Heron 是可以被运行在现有的各种资源调度系统中的，也就是说，我们并不需要像 Storm 一样在多台物理机上安装程序，然后设置端口进行部署。我们仅仅需要告诉 Heron ：现在想使用什么集群？如何连接这些集群？。至于第一个问题，我们可以通过在执行命令变更集群参数来完成消息传递；第二个问题，我们可以根据 Heron 变更配置文件信息，来完成相关的参数配置，让拓扑在运行时与资源调度系统(如 YARN)对接，然后拓扑就会运行在调度系统中了。这个核心问题便是由 **Scheduler** 负责的。此时，我们可以说，我们仅需要一台能够运行 `heron submit` 命令的客户端和一个已有的资源调度系统就能运行拓扑了。

**State Manager** 是另外一个非常重要的组件，在拓扑被提交之前，必须要明确的知道使用什么方式进行拓扑状态管理：Local File System?Zookeeper？拓扑会在状态变更时往相应的系统上写入变更信息。这时要想到另外一个问题，我们之前说 Tracker 追踪拓扑运行时状态，既然拓扑状态是写入指定系统的，那么 Tracker 从哪儿获知这个信息呢？这是一开始部署 Heron 容易混淆的问题。没错，我们在启动 Tracker 的时候也必须告诉它(配置文件) **State Manager** 的工作系统是什么，路径在哪里。这时 Tracker 才能真正起到监听、追踪的作用。至于拓扑是向调度系统申请资源、在资源沙箱内如何正常运行等等这些问题，我们都不用关心，Heron 的开发者们已经为此做好了充足准备。

我们现在再来捋一下思路：Heron 可以被部署在多个集群上，每个集群都有自己的调度器和拓扑状态管理系统，Tracker 可以监听多个状态管理系统，UI 负责展现。

还要说明的一点是：目前调度器还支持 Mesos Cluster Locally。
