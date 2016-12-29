# [Heron UI](http://twitter.github.io/heron/docs/operators/heron-ui/)

**Heron UI** 是用户获得拓扑信息的可视化界面，通过不用着色展现拓扑的相关信息，如每个拓扑的[逻辑计划](../Heron-Concepts/Heron-Topology.md#逻辑执行计划-logical-plan)和[物理计划](../Heron-Concepts/Heron-Topology.md#物理执行计划-physical-plan)。它与 [Heron Tracker](../Heron-Concepts/Heron-Architecture#heron-tracker) 交互获取具体信息。

## 编译 Heron UI (Building Heron UI)

Heron 使用 [bazel](http://bazel.io/) 构建。可以查看[编译 Heron](../Heron-Developers/compiling.md) 小节了解安装、配置 bazel 的具体信息。

```bash
# Build heron-ui
$ bazel build heron/tools/ui/src/python:heron-ui

# The location of heron-ui pex executable is
# bazel-bin/heron/tools/ui/src/python/heron-ui
# To run using default options:
$ ./bazel-bin/heron/tools/ui/src/python/heron-ui
```

`heron-ui` is a self executable
[pex](https://pex.readthedocs.io/en/latest/whatispex.html) archive.

### Heron UI 启动参数 (Heron UI Args)

* `--port` - 指定 heron-ui 的启动端口，默认端口 `8889`。
* `--tracker_url` - 指定 tracker url。 heron-ui 从 track 处获取相关信息。Track 缺省的 url 是 `http://localhost:8888`.

```bash
$ heron-ui
# is equivalent to
$ heron-ui --port=8889 --tracker_url=http://localhost:8888
```
