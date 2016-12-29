# Heron Tracker

**Heron Tracker** 是实时收集 Heron 拓扑信息的 web 服务(service)，它也对外提供 [JSON REST API](http://twitter.github.io/heron/docs/operators/heron-tracker-api/)。关于 Tracker 的更多信息可以查看[链接](../Heron-Concepts/Heron-Architecture#heron-tracker)。

## 编译 Heron Tracker (Building Heron Tracker)

Heron 使用 [bazel](http://bazel.io/) 构建。可以查看[编译 Heron](../Heron-Developers/compiling.md) 小节了解安装、配置 bazel 的具体信息。

```bash
# Build heron-tracker
$ bazel build heron/tools/tracker/src/python:heron-tracker

# The location of heron-tracker pex executable is
# bazel-bin/heron/tools/tracker/src/python/heron-tracker
# To run using default options:
$ ./bazel-bin/heron/tools/tracker/src/python/heron-tracker
```

### Heron Tracker Config File

可以在 `yaml` 文件中找到相关的配置信息

#### 1. State Manager locations

告知 Tracker 您设置 State Manager 路径以便 Tracker 可以正常监听该路径，相关配置可参考：

```yaml
## Contains the sources where the states are stored.
# Each source has these attributes:
# 1. type - type of state manager (zookeeper or file, etc.)
# 2. name - name to be used for this source
# 3. hostport - only used to connect to zk, must be of the form 'host:port'
# 4. rootpath - where all the states are stored
# 5. tunnelhost - if ssh tunneling needs to be established to connect to it
statemgrs:
  -
    type: "file"
    name: "local"
    rootpath: "~/.herondata/repository/state/local"
    tunnelhost: "localhost"
#
# To use 'localzk', launch a zookeeper server locally
# and create the following path:
#  *. /heron/topologies
#
#  -
#    type: "zookeeper"
#    name: "localzk"
#    hostport: "localhost:2181"
#    rootpath: "/heron"
#    tunnelhost: "localhost" -
```

需要注意的是，tracker 是只读服务，自身并不会在 Zookeeper 上创建任何节点。不论是 Local 模式还是 Zookeeper 模式，在启动 tracker 前请确保所设 rootpath 已经存在。

#### 2. Viz URL Format

这是一个可选配置，如果设置，则在 UI 页面上会显示每一个拓扑的 viz 链接。如图所示

![Viz Link](http://twitter.github.io/heron/img/viz-link.png)

```yaml
# The URL that points to a topology's metrics dashboard.
# This value can use following parameters to create a valid
# URL based on the topology. All parameters are self-explanatory.
# These are found in the execution state of the topology.
#
#   ${CLUSTER}
#   ${ENVIRON}
#   ${TOPOLOGY}
#   ${ROLE}
#   ${USER}
#
# This is a sample, and should be changed to point to corresponding dashboard.
#
# viz.url.format: "http://localhost/${CLUSTER}/${ENVIRON}/${TOPOLOGY}/${ROLE}/${USER}"
```

### Heron Tracker Args

* `--port` --- 指定 heron-tracker 的启动端口，默认端口 `8888`。
* `--config-file` --- 指定 tracker 启动时加载的配置文件路径，默认路径：`~/.herontools/conf/heron_tracker.yaml`。

```bash
$ heron-tracker
# is equivalent to
$ heron-ui --port=8888 --config-file=~/.herontools/conf/heron_tracker.yaml
```
