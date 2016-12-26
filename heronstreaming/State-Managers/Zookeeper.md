# [ZooKeeper State Manager](http://twitter.github.io/heron/docs/operators/deployment/statemanagers/zookeeper/)

Heron 需要利用 Zookeeper 做一些组织协调工作(coordination tasks)。我们既可以复用已有的 Zookeeper 集群，也可以为 Heron 搭建专有的集群。

在具体设置 Zookeeper 集群之前，有一些注意事项：

* Heron 只利用 Zookeeper 做协调工作(coordination)，并不承担任何消息传递(message passing)的工作。这就意味着，Zookeeper 的负载将会被控制在很低的水平。无论是单实例的节点，还是多节点的集群都能满足 Heron 的需求，按需搭建即可。
* 与 Storm 相比，Heron 对 Zookeeper 的利用率是比较高的。因此 Heron 可以不使用专门的 Zookeeper 集群，但至少得有一个。
* 我们强烈推荐使用有[监督模式的 Zookeeper](http://zookeeper.apache.org/doc/r3.3.3/zookeeperAdmin.html#sc_supervision)。

### ZooKeeper State Manager Configuration

可以通过修改 `statemgr.yaml` 文件中的参数来调整 State Manager 对 Zookeeper 的使用。其中包括以下参数：

* `heron.class.state.manager` --- 设置 State Manager 的加载类，系统会通过反射的方式加载它。在 Zookeeper 模式下，请将其设置为 `com.twitter.heron.statemgr.zookeeper.curator.CuratorStateManager`。

* `heron.statemgr.connection.string` --- Zookeeper 连接串，必须包含 IP、端口(port)等信息，形如：`127.0.0.1:2181`。

* `heron.statemgr.root.path` --- 设置 State Manager 根路径，我们强烈建议使用一个独立的根节点。否则，请确保`/tmasters`, `/topologies`, `/pplans`, `/executionstate`, `/schedulers`的可用性。

* `heron.statemgr.zookeeper.is.initialize.tree` --- 设置 Zookeeper 的节点是否按树状展开，默认为 `True`

* `heron.statemgr.zookeeper.session.timeout.ms` --- 设置 Zookeeper 会话(session)超时时间，单位是毫秒。

* `heron.statemgr.zookeeper.connection.timeout.ms` --- 设置 Zookeeper 连接(connection)超时时间，单位是毫秒。

* `heron.statemgr.zookeeper.retry.count` --- 设置 Zookeeper 连接重试次数。

* `heron.statemgr.zookeeper.retry.interval.ms` --- 设置 Zookeeper 重连间隔时间，单位是毫秒。

### Example ZooKeeper State Manager Configuration

下面是 Zookeeper 状态管理器的 `statemgr.yaml` 默认配置，示例中 ZK 运行在 `localhost` 上：

```yaml
# local state manager class for managing state in a persistent fashion
heron.class.state.manager: com.twitter.heron.statemgr.zookeeper.curator.CuratorStateManager

# local state manager connection string
heron.statemgr.connection.string:  "127.0.0.1:2181"

# path of the root address to store the state in a local file system
heron.statemgr.root.path: "/heron"

# create the zookeeper nodes, if they do not exist
heron.statemgr.zookeeper.is.initialize.tree: True

# timeout in ms to wait before considering zookeeper session is dead
heron.statemgr.zookeeper.session.timeout.ms: 30000

# timeout in ms to wait before considering zookeeper connection is dead
heron.statemgr.zookeeper.connection.timeout.ms: 30000

# timeout in ms to wait before considering zookeeper connection is dead
heron.statemgr.zookeeper.retry.count: 10

# duration of time to wait until the next retry
heron.statemgr.zookeeper.retry.interval.ms: 10000
```
