# Stream Manager(http://twitter.github.io/heron/docs/operators/configuration/stmgr/)

您可以通过以下参数配置 Stream Manager，包括调整背压机制的触发条件。

## 背压机制参数

参数 | 描述 | 默认值
:----- |:------- |:-------
`heron.streammgr.network.backpressure.threshold` | 在启动背压机制之前，Stream Manager 检查队列是否已被写满的次数 | `3`
`heron.streammgr.network.backpressure.highwatermark.mb` | The high water mark on the number of megabytes that can be left outstanding on a connection | `50`
`heron.streammgr.network.backpressure.lowwatermark.md` | The low water mark on the number of megabytes that can be left outstanding on a connection | `30`
`heron.streammgr.network.options.maximum.packet.mb` | SM 网络选项中的消息包大小，单位：MB | `100`

---
***笔者注释：***
关于背压机制 high/low watermark 的理解可以简述为：当 buffer size 到达 50MB(highwatermark) 以上时会触发背压机制；进入背压状态后，当且仅当 buffer size 小于 30MB(lowwatermark) 之后，Stream Manager 会停止背压状态。更深入的描述，可以参考 [Twitter-Heron](http://dl.acm.org/citation.cfm?id=2742788) 论文。

## Timeout Interval

参数 | 描述 | 默认值
:----- |:------- |:-------
`heron.streammgr.xormgr.rotatingmap.nbuckets` | TODO | `3`

## 其他参数 (Other Parameters)

参数 | 描述 | 默认值
:----- |:------- |:-------
`heron.streammgr.packet.maximum.size.bytes` | SM 发送数据的最大数据包大小，单位：字节 | `102400`
`heron.streammgr.cache.drain.frequency.ms` | SM tuple 缓存刷新频率，单位：毫秒 | `10`
`heron.streammgr.cache.drain.size.mb` | SM tuple 缓存空间大小，单位：MB | `100`
`heron.streammgr.client.reconnect.interval.sec` | SM 被其他 SM 重连的频率，单位：秒 | `1`
`heron.streammgr.client.reconnect.tmaster.interval.sec` | SM 重连 TM 的频率，单位：秒 | `10`
`heron.streammgr.tmaster.heartbeat.interval.sec` | 发送给 TM 的心跳间隔，单位：秒 | `10`
`heron.streammgr.connection.read.batch.size.mb` | SM 从 socket 中批量读取数据的大小，单位：MB | `1`
`heron.streammgr.connection.write.batch.size.mb` | SM 往 socket 中批量写入数据的大小，单位：MB | `1`
