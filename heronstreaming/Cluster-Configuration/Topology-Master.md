# [Topology Master](http://twitter.github.io/heron/docs/operators/configuration/tmaster/)

您可以用如下参数配置 [Topology Master](../Heron-Concepts/Heron-Architecture.md#topology-master)

参数 | 描述 | 默认值
:----- |:------- |:-------
`heron.tmaster.metrics.collector.maximum.interval.min` | Topology Master 持有 metrics 最长时间间隔，单位：分钟 | 180
`heron.tmaster.establish.retry.times` | 重试部署 Topology Master 的最大尝试次数 | 30
`heron.tmaster.establish.retry.interval.sec` | 重试部署 Topology Master 的最长时间间隔 | 1
`heron.tmaster.network.master.options.maximum.packet.mb` | Topology Master 网络选项之一，用于调整网络连接 Stream Manager 时的消息包大小，单位：MB | 16
`heron.tmaster.network.controller.options.maximum.packet.mb` | Topology Master 网络选项之一，用于调整网络连接调度器(scheduler)时的消息包大小，单位：MB | 1
`heron.tmaster.network.stats.options.maximum.packet.mb` | Topology Master 网络选项之一，用于调整状态查询时的消息包大小，单位：MB | 1
`heron.tmaster.metrics.collector.purge.interval.sec` | Topology Master 从 socket 中清理 metrics 的时间间隔，单位：秒 | 60
`heron.tmaster.metrics.collector.maximum.exception` | 拓扑的维度收集器的最大异常上限，可防止潜在的 OOM 问题 | 256
`heron.tmaster.metrics.network.bindallinterfaces` | Metrics reporter 是否绑定到所有接口上 | `False`
`heron.tmaster.stmgr.state.timeout.sec` | 与 Stream Manger 连接超时门限(当前时间-上一次心跳时间)，单位：秒 | 60
