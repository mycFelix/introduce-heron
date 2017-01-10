# [Heron 实例](http://twitter.github.io/heron/docs/operators/configuration/instance/)

您可以通过配置以下参数来进行 [Heron 实例](../Heron-Concepts/Heron-Architecture.md#heron-instance)的相关设置

## 内部配置 (Internal Configuration)

这些参数负责调整每一个实例的 TCP 读写队列的具体情况。

参数 | 描述 | 默认值
:----- |:------- |:-------
`heron.instance.internal.bolt.read.queue.capacity`     | bolt 从 Stream Manager 读取数据的缓冲区队列容量(记录条数)  | 128
`heron.instance.internal.bolt.write.queue.capacity`    | bolt 往 Stream Manager 写入数据的缓冲区队列容量(记录条数)  | 128
`heron.instance.internal.spout.read.queue.capacity`    | spout 从 Stream Manager 读取数据的缓冲区队列容量(记录条数) | 1024
`heron.instance.internal.spout.write.queue.capacity`   | spout 往 Stream Manager 写入数据的缓冲区队列容量(记录条数) | 128
`heron.instance.internal.metrics.write.queue.capacity` | 往 Metrics Manager 写入数据的缓冲区队列容量               | 128


## 网络设置 (Network Configuration)

我们提供了两种策略来配置 Heron 实例收集 & 传输数据：**基于时间方案**和**基于空间方案**。具体配置时，您只能在他们之间二选一。如果您选择基于时间方案，您可以调整 Heron 实例的 socket 批量读写数据的最大单位时间(毫秒);如果您选择基于空间的方案，您可以指定每批次数据的字节长度。

参数 | 描述 | 默认值
:----- |:------- |:-------
`heron.instance.network.read.batch.time.ms`     | 基于时间策略，实例从 Stream Manager 批量读取数据的时间 | 16
`heron.instance.network.read.batch.size.bytes`  | 基于空间策略，实例从 Stream Manager 批量读取数据的字节长度 | 32768
`heron.instance.network.write.batch.time.ms`    | 基于时间策略，实例往 Stream Manager 批量写入数据的时间 | 16
`heron.instance.network.write.batch.size.bytes` | 基于空间策略，实例从 Stream Manager 批量写入数据的字节长度 | 32768

### 其他网络参数 (Other Network Parameters)

如下是与时间或空间策略无关的参数

参数 | 描述 | 默认值
:----- |:------- |:-------
`heron.instance.network.options.socket.send.buffer.size.bytes` | Socket 发送数据缓冲区字节大小 | 6553600
`heron.instance.network.options.socket.received.buffer.size.bytes` | Socket 接收数据缓冲区字节大小 | 8738000
`heron.instance.reconnect.streammgr.interval.sec` | 重连 Stream Manager 的时间间隔，包含连接时请求超时的情况 | 5
`heron.instance.reconnect.streammgr.times` | 在 Stream Manager 被强制重启之前的最大重连次数 | 60

## Metrics Manager Configuration

以下这些参数可以控制 Heron 实例与 [Stream Manager](../Heron-Concepts/Heron-Architecture.md#stream-manager) 的交互状况

参数 | 描述 | 默认值
:----- |:------- |:-------
`heron.instance.metrics.system.sample.interval.sec` | 实例对系统维度的采样秒级间隔，如 CPU 负载 | 10
`heron.instance.reconnect.metricsmgr.interval.sec` | 重连 Metrics Manager 的时间间隔，包含连接时请求超时的情况 | 5
`heron.instance.reconnect.metricsmgr.times` | 在 Metrics Manager 被强制重启之前的最大重连次数 | 60

## 调优

以下这些参数用于动态调整读写队列的大小以期望于得到更优异的性能，同时也可以有效避免一些垃圾回收方面的问题。

参数 | 描述 | 默认值
:----- |:------- |:-------
`heron.instance.tuning.expected.bolt.read.queue.size`   | bolt 读取队列大小的期望值 | 5
`heron.instance.tuning.expected.bolt.write.queue.size`  | bolt 写入队列大小的期望值 | 5
`heron.instance.tuning.expected.spout.read.queue.size`  | spout 读取队列大小的期望值 | 512
`heron.instance.tuning.expected.spout.write.queue.size` | spout 写入队列大小的期望值 | 5
`heron.instance.tuning.expected.metrics.write.queue.size` | Metrics 队列写入大小的期望值 | 5
`heron.instance.tuning.current.sample.weight` | TODO | 0.8

## 其他参数

参数 | 描述 | 默认值
:----- |:------- |:-------
`heron.instance.set.data.tuple.capacity` | The maximum number of data tuples to batch in a `HeronDataTupleSet` protobuf message | 256
`heron.instance.set.control.tuple.capacity` | The maximum number of control tuples to batch in a `HeronControlTupleSet` protobuf message | 256
`heron.instance.ack.batch.time.ms` | The maximum time in ms for an spout to do acknowledgement per attempt, the ack batch could also break if there are no more ack tuples to process |128
`heron.instance.emit.batch.time.ms` | spout 实例每次批量发射(emit) tuple 最长时间，单位：毫秒 | 16
`heron.instance.emit.batch.size.bytes` | spout 实例每次批量发射(emit) tuple 的最大字节数 | 32768
`heron.instance.execute.batch.time.ms` | bolt 实例每次批量执行(execute) tuple 最长时间，单位：毫秒 | 16
`heron.instance.execute.batch.size.bytes` | bolt 实例每次批量执行(execute) tuple 的最大字节数 | 32768
`heron.instance.state.check.interval.sec` | 检查实例状态变化的时间间隔，例如检查 spout 是激活态还是非激活态 | 5
`heron.instance.acknowledgement.nbuckets` | TODO | 10

---

***笔者后记***

本小节内容是 Heron Instance 的相关参数。如果读过 [Twitter Heron: Stream Processing at Scale](http://dl.acm.org/citation.cfm?id=2742788) 这篇论文，就会了解本小节的主要内容旨在调整 Heron Instance 内部以及他们之间的通信参数。有一小部分参数是笔者在翻译过程中拿不准的，所以没有翻译，保留了原文。如果有翻译不准确的地方，也请指正，毕竟这些内容关系到拓扑的运行性能。
