# [运行 Heron 单元测试](http://twitter.github.io/heron/docs/contributors/testing/)

Heron 使用 [Bazel](../Heron-Developers/Compiling.md#安装-bazel-installing-bazel) 来编译和运行单元测试。在具体执行单元测试样例前，请先按照[编译 Heron](../Heron-Developers/Compiling.md) 小节的指引配置本地的编译环境。

## 运行单元测试 (Running Unit Tests)

执行所有测试样例的命令如下：

```bash
$ bazel test --config=darwin heron/...
```

如需运行[指定的测试样例](http://bazel.io/docs/test-encyclopedia.html)，请传递目标名称，例如：

```bash
$ bazel test --config=darwin heron/statemgrs/tests/java:localfs-statemgr_unittest
```

## 查找目标单元测试样例 (Discovering Unit Test Targets)

如下命令展现了如何查看所有 Bazel 的测试单元：

```bash
$ bazel query 'kind(".*_test rule", ...)'
```

如下命令展现了如何查看所有 **Java** 的测试单元：

```bash
$ bazel query 'kind("java_test rule", ...)'
```

如下命令展现了如何查看所有 **C++** 的测试单元：

```bash
$ bazel query 'kind("cc_test rule", ...)'
```

如下命令展现了如何查看所有 **Python** 的测试单元：

```bash
$ bazel query 'kind("pex_test rule", ...)'
```

## 运行集成测试 (Running Integration Tests)

集成测试分为如下两个部分:

* 功能集成测试 (Functional integration tests)

    这一部分集成测试旨在重点测试 Heron 的功能，如运行拓扑(topologies)或数据分组(groupings)。可以在 Mac OS X 平台上执行如下命令运行测试：

    ```bash
    $ ./scripts/run_integration_test.sh
    ```

* 故障集成测试 (Failure integration tests)

    这一部分集成测试旨在重点测试指定进程遇到故障后的可恢复能力，如 Topology Master 或 Metrics Manager。可以在 Mac OS X 平台上执行如下命令运行测试：

    ```bash
    $ bazel build --config=darwin integration-test/src/...
    $ ./bazel-bin/integration-test/src/python/local_test_runner/local-test-runner
    ```
