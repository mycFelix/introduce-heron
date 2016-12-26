# [配置集群](http://twitter.github.io/heron/docs/operators/deployment/configuration/)

在启动 Heron 集群之前，有一些配置文件需要设置。每个文件对应着一个 Heron 计算框架的组件。

***笔者注释：为避免歧义，组件将使用英文名描述***

* **scheduler.yaml** --- 为了让 Scheduler 正常工作，我们需要在这个文件中指定 launcher 和 scheduler 的加载类，以及一些其他的 Scheduler 运行时参数。

* **statemgr.yaml** --- 这个文件用于设置 State manager 的加载类，以便它能正常管理拓扑的逻辑计划、物理计划、调度状态和执行状态。

* **uploader.yaml** --- 这个文件用于设置 Uploader 的加载类，以便拓扑 Jar 能正常上传到指定存储器中。拓扑启动时，会从存储中加载这些 Jar。

* **heron_internals.yaml** --- 这是 Heron 核心参数控制文件。对 Heron 非常了解的开发者，可以通过修改这个文件的参数进行调优工作。对于初学者而言，可以使用我们提供的默认配置。一旦你熟悉了整个系统，你就可以利用这个文件中的若干参数来提高吞吐量或是降低延迟率。

* **metrics_sinks.yaml** --- 这个文件用于指定 Metrics Sink 信息，默认是 `file sink` 和 `tmaster sink`。另外目前也支持 `scribe sink` and `graphite sink`。

* **packing.yaml** --- 这个文件用于指定`打包算法(packing algorithm)`，默认为鲁棒算法

* **client.yaml** --- 这个文件用于管理 `heron` 命令客户端，这是一个可选配置文件。
