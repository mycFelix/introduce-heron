# Setting Up Local File System State Manager

Heron 可以使用本地文件系统做为状态管理器来持久化诸多信息。该模式并不适合集群部署，建议仅仅在单节点或笔记本电脑上使用此模式，而且相关参数也都是为终端设备而设计的。Heron 的开发者们可以使用这个模式来进行 Heron 组件的本地开发和验证。

### Local File System State Manager Configuration

可以通过修改 `statemgr.yaml` 文件中的参数来调整 Heron 集群对本地文件系统的使用，其中包括以下参数：

* `heron.class.state.manager` --- 设置 State Manager 的加载类，系统会通过反射的方式加载它。在本地文件系统模式下，请将其设置为 `com.twitter.heron.statemgr.localfs.LocalFileSystemStateManager`。

* `heron.statemgr.connection.string` --- 如果使用本地文件系统，请将其设置为 `LOCALMODE`.

* `heron.statemgr.root.path` --- 设置 State Manager 根路径，我们强烈建议使用一个独立的根节点。否则，请确保`/tmasters`, `/topologies`, `/pplans`, `/executionstate`, `/schedulers`的可用性。

* `heron.statemgr.localfs.is.initialize.file.tree` --- 设置 Zookeeper 的节点是否按树状展开，默认为 `True`

### Example Local File System State Manager Configuration

可以通过修改 `statemgr.yaml` 文件中的参数来调整 State Manager 对本地文件系统的使用。其中包括以下参数：

```yaml
# local state manager class for managing state in a persistent fashion
heron.class.state.manager: com.twitter.heron.statemgr.localfs.LocalFileSystemStateManager

# local state manager connection string
heron.statemgr.connection.string: LOCALMODE

# path of the root address to store the state in a local file system
heron.statemgr.root.path: ${HOME}/.herondata/repository/state/${CLUSTER}

# create the sub directories, if needed
heron.statemgr.localfs.is.initialize.file.tree: True
```
