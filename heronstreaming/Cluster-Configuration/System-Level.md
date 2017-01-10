# [系统参数](http://twitter.github.io/heron/docs/operators/configuration/system/)

***笔者注释：对相关参数含义的正确理解有助于开发者进一步的调优。笔者在翻译的同时，在页面上保留了英文原文，以防在翻译过程中遗漏信息。同时，如有纰漏，也欢迎各位指正***

如下参数为系统级别参数，针对这些参数的调整无法对任何指定的组件产生影响。

## 通用

参数 | 描述 | 默认值
:----- |:------- |:-------
`heron.check.tmaster.location.interval.sec` | 确认 topology master 是否已被侦测到的秒级间隔 | 120
`heron.metrics.export.interval` | 组件向 topology's Metrics Manager 上传信息的秒级间隔

Config | Meaning | Default
:----- |:------- |:-------
`heron.check.tmaster.location.interval.sec` | The interval, in seconds, after which to check if the topology master location has been fetched or not | 120
`heron.metrics.export.interval` | The interval, in seconds, at which components export metrics to the topology's Metrics Manager

## 日志

参数 | 描述 | 默认值
:----- |:------- |:-------
`heron.logging.directory` | 日志的相对路径 | `log-files`
`heron.logging.maximum.size.mb` | 日志文件最大可用空间(单位：GB) | 100
`heron.logging.maximum.files` | 最多的日志文件数 | 5
`heron.logging.prune.interval.sec` | Heron 裁剪(prune)日志间隔（单位：秒） | 300
`heron.logging.flush.interval.sec` | Heron 写入(flush)日志间隔（单位：秒） | 10
`heron.logging.err.threshold` | 日志报错门限 | 3

Config | Meaning | Default
:----- |:------- |:-------
`heron.logging.directory` | The relative path to the logging directory | `log-files`
`heron.logging.maximum.size.mb` | The maximum log file size (in megabytes) | 100
`heron.logging.maximum.files` | The maximum number of log files | 5
`heron.logging.prune.interval.sec` | The time interval, in seconds, at which Heron prunes log files | 300
`heron.logging.flush.interval.sec` | The time interval, in seconds, at which Heron flushes log files | 10
`heron.logging.err.threshold` | The threshold level to log error | 3
