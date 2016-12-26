# [Setting Up Local File System Uploader](http://twitter.github.io/heron/docs/operators/deployment/uploaders/localfs/)

***笔者注释：Uploader 顾名思义，就是负责将 Jar 上传到指定共享路径，然后再被下载和分发。需要注意的是 Heron on Yarn 模式不需要设置 Uploader。***

当您提交拓扑时，Heron 会将拓扑的 Jar 上传到指定路径持久化。提交器将会把持久化路径告知 scheduler，并由 scheduler 广播给每个 container 。 Heron 可以使用本地文件系统做为拓扑 Jar 的持久化路径。有以下几个注意事项：

* 该模式主要和本地调度器结合使用
* 如果您想在单节点、笔记本、或者其他终端设备上运行 Heron，该模式是最理想的解决方案
* 该模式有助于 Heron 开发者对核心组件进行本地测试。

### Local File System Uploader Configuration

您可以在 `uploader.yaml` 文件中对设置 Uploader 为本地文件系统：

* `heron.class.uploader` --- 设置 Uplodaer 加载类为 `com.twitter.heron.uploader.localfs.LocalFileSystemUploader`

* `heron.uploader.localfs.file.system.directory` --- 设置持久化路径的目录名。每个集群应有唯一的目录名，你可以使用 Heron 的环境变量 `${CLUSTER}` 做目录名称。

### Example Local File System Uploader Configuration

下面是 `uploader.yaml` 的示例配置：

```yaml
# uploader class for transferring the topology jar/tar files to storage
heron.class.uploader: com.twitter.heron.uploader.localfs.LocalFileSystemUploader

# name of the directory to upload topologies for local file system uploader
heron.uploader.localfs.file.system.directory: ${HOME}/.herondata/topologies/${CLUSTER}
```
