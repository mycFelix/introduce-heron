# [使用 Heron CLI 管理拓扑](http://twitter.github.io/heron/docs/operators/heron-cli/)

我们使用 Heron CLI 管理整个拓扑的生命周期。

## 部署 `heron` CLI 可执行环境

下载 `heron-client-install` 对应平台的[可执行文件(release binaries)](https://github.com/twitter/heron/releases) 并运行脚本，以 `0.14.5` 为例：

```bash
$ chmod +x heron-client-install-0.14.5-darwin.sh
$ ./heron-client-install-0.14.5-darwin.sh --user
Heron client installer
----------------------

Uncompressing......

Heron is now installed!

Make sure you have "/Users/$USER/bin" in your path.

See http://heronstreaming.io/docs/getting-started.html on how to use Heron!

....
```

或者，您也可以通过[编译 Heron](../Heron-Developers/Compiling.md) ，自己生成可执行文件，安装到指定的机器上。

### 通用命令参数 (Common CLI Args)

所有的拓扑管理命令(`submit`、`activate`、`deactivate`、`restart`、`update`以及`kill`)都需要一下参数：

* `cluster` --- 集群的具体名称。

* `role` --- 针对哪一个角色执行这一命令。如不指定，则使用当前 unix 用户名。

* `env` --- 环境名称。这个相当于给运行拓扑打上指定的标签。如生产环境可以用 `PROD` 表示，开发环境可以用 `DEVEL` 表示。如不指定，`default` 为默认值。

`cluster`、`role` 以及 `env` 可以通过 `/` 链接起来形成一个参数指令如 `cluster/role/env` 对应 `local/ads/PROD`，表示：以 `ads` 用户身份，在 `local` 集群上启动一个负责在 `PROD` 环境运行的拓扑。

### 可选 CLI 参数 (Optional CLI Flags)

在执行管理命令(`submit`、`activate`、`deactivate`、`restart`、`update`以及`kill`)时，Heron 还提供了一些可选参数：

* `--config-path` --- 每一个 Heron 集群都必须加载指定的配置文件才能正常运行。如不指定配置文件路径，Heron 会从 `${HERON_HOME}/conf/${CLSTER_NAME}` 路径中获取指定的集群名称。这个参数是可以让自定义配置文件路径，正常执行命令到您所指定的集群。

* `--config-property` --- Heron 拓扑运行时可能会需要一些额外的配置，我们可以指定这个参数值来达成这一目标，参数形式如 `key=value`。

* `--verbose` --- 当添加这个参数时，在 `heron` 命令执行过程中，会打印所有详细日志

下面是一个命令执行样例：

```bash
$ heron activate --config-path ~/heronclusters devcluster/ads/PROD AckingTopology
```

## 提交拓扑 (Submitting a Topology)

我们可以使用 `submit` 命令将拓扑提交至集群，我们还可以在提交时控制拓扑的激活状态，默认为激活态(activated)。如下为语法示例：

```bash
$ heron help submit
usage: heron submit [options] cluster/[role]/[env] topology-file-name topology-class-name [topology-args]

Required arguments:
  cluster/[role]/[env]  Cluster, role, and env to run topology
  topology-file-name    Topology jar/tar/zip file
  topology-class-name   Topology class name

Optional arguments:
  --config-path (a string; path to cluster config; default: "/Users/$USER/.heron/conf")
  --config-property (key=value; a config key and its value; default: [])
  --deploy-deactivated (a boolean; default: "false")
  --topology-main-jvm-property Define a system property to pass to java -D when running main.
  --verbose (a boolean; default: "false")
```

`submit` 的参数说明:

* **cluster/[role]/[env]** --- 指定集群名，拓扑所属角色以及运行环境名称，如 `local/ads/PROD` 或 `local`(省略 role 和 env)。

* **topology-file-name** --- 拓扑可执行文件路径。对于使用 Java 语言编写的拓扑而言，可能是一个 Jar 文件；其他语言可能是一个 tar 文件。以 Jar 文件为例：`/path/to/topology/my-topology.jar`

* **topology-class-name** --- 拓扑可执行 `main class`，如： `com.example.topologies.MyTopology`

* **topology-args** (可选) --- 拓扑启动时需要的参数。可以在 `main class` 里编写拓扑自身所需的参数信息

### 拓扑提交示例 (Example Topology Submission Command)

下面，我们将往名为 `devcluster` 的集群上提交 `my-topology.jar` 中的 `com.example.topologies.MyTopology` 拓扑，并且使用 `--config-path` 参数来配置 `devcluster` 集群。命令如下：

```bash
$ heron submit --config-path ~/heronclusters devcluster /path/to/topology/my-topology.jar \
    com.example.topologies.MyTopology my-topology
```

### 其他提交选项 (Other Topology Submission Options)

| 参数                            | 含义                                      |
|:-------------------------------|:------------------------------------------|
| `--deploy-deactivated`         | 如果设置，则拓扑已非激活态(deactivated)方式提交 |
| `--topology-main-jvm-property` | 设置运行时 JVM 的相关参数，参数会传递给 java -D |

## 激活拓扑 (Activating a Topology)

拓扑默认以激活态形式提交至集群。我们可以使用 `activate` 命令将拓扑状态由非激活态(deactivated)更改为激活态(activate)。下面是基本语法说明：

```bash
$ heron help activate
usage: heron activate [options] cluster/[role]/[env] topology-name

Required arguments:
  cluster/[role]/[env]  Cluster, role, and env to run topology
  topology-name         Name of the topology

Optional arguments:
  --config-path (a string; path to cluster config; default: "/Users/$USER/.heron/conf")
  --config-property (key=value; a config key and its value; default: [])
```

`activate` 命令参数说明：

* **cluster/[role]/[env]** --- 指定集群名，拓扑所属角色以及运行环境名称，如 `local/ads/PROD` 或 `local`(省略 role 和 env)。

* **topology-name**  --- 需要被激活的拓扑名称。

### 激活命令示例 (Example Topology Activation Command)

```bash
$ heron activate local/ads/PROD my-topology
```

## 暂停拓扑 (Deactivating a Topology)

我们可以使用 `deactivate` 命令暂停拓扑，下面是命令的基本语法：

```bash
$ heron help deactivate
usage: heron deactivate [options] cluster/[role]/[env] topology-name

Required arguments:
  cluster/[role]/[env]  Cluster, role, and env to run topology
  topology-name         Name of the topology

Optional arguments:
  --config-path (a string; path to cluster config; default: "/Users/$USER/.heron/conf")
  --config-property (key=value; a config key and its value; default: [])
  --verbose (a boolean; default: "false")

```

`deactivate` 命令参数：

* **cluster/[role]/[env]** --- 指定集群名，拓扑所属角色以及运行环境名称，如 `local/ads/PROD` 或 `local`(省略 role 和 env)。

* **topology-name**  --- 需要被暂停的拓扑名称。

## 重启拓扑 (Restarting a Topology)

我们可以使用 `restart` 命令重启拓扑(假定拓扑没有被杀死(Kill))。命令语法示例如下：

```bash
$ heron help restart
usage: heron restart [options] cluster/[role]/[env] topology-name [container-id]

Required arguments:
  cluster/[role]/[env]  Cluster, role, and env to run topology
  topology-name         Name of the topology
  container-id          Identifier of the container to be restarted

Optional arguments:
  --config-path (a string; path to cluster config; default: "/Users/${USER}/.heron/conf")
  --config-property (key=value; a config key and its value; default: [])
  --verbose (a boolean; default: "false")
```

`restart` 命令参数：

* **cluster/[role]/[env]** --- 指定集群名，拓扑所属角色以及运行环境名称，如 `local/ads/PROD` 或 `local`(省略 role 和 env)。

* **topology-name**  --- 需要被重启的拓扑名称。

* **container-id** (可选) --- 可指定传递 container ID 指定重启拓扑的某一个 container。

### 重启命令示例 (Example Topology Restart Command)

```bash
$ heron restart local/ads/PROD my-topology
```

## 更新拓扑 (Updating a Topology)

可以通过 `update` 参数变更拓扑中任意组件的并行度，命令语法示例如下：

```bash
$ heron help update
usage: heron update [options] cluster/[role]/[env] <topology-name> --component-parallelism <name:value>

Required arguments:
  cluster/[role]/[env]  Cluster, role, and environment to run topology
  topology-name         Name of the topology

Optional arguments:
  --component-parallelism COMPONENT_PARALLELISM
                        Component name and the new parallelism value colon-
                        delimited: [component_name]:[parallelism]
  --config-path (a string; path to cluster config; default: "/Users/${USER}/.heron/conf")
  --config-property (key=value; a config key and its value; default: [])
  --verbose (a boolean; default: "false")
```

**cluster/[role]/[env]**  和 **topology-name** 参数的说明与之前命令一致，新增 **--component-parallelism** 参数

* **--component-parallelism** --- 在变更拓扑组件并行度时，该参数可以被调用多次。

### 更新命令样例 (Example Topology Update Command)

```bash
$ heron update local/ads/PROD my-topology \
  --component-parallelism=my-spout:2 \
  --component-parallelism=my-bolt:4
```

## 杀死拓扑 (Killing a Topology)

如果您想从 Heron 集群上永久删除某个拓扑，您可以使用 `kill` 命令。命令语法示例如下：

```bash
$ heron kill <killer-overrides> <topology>
```

`kill` 命令的参数说明：

* **cluster/[role]/[env]** --- 指定集群名，拓扑所属角色以及运行环境名称，如 `local/ads/PROD` 或 `local`(省略 role 和 env)。

* **topology-name**  --- 需要被杀死的拓扑名称。

### 杀死命令样例 (Example Topology Kill Command)

```bash
$ heron kill local my-topology
```

## Other Commands

### Version

运行 `verion` 命令显示当前 Heron 版本：

```bash
$ heron version
heron.build.version : 0.13.5
heron.build.time : Wed May 11 23:49:00 PDT 2016
heron.build.timestamp : 1463035740000
heron.build.host : mbp-machine
heron.build.user : userwhobuilt
INFO: Elapsed time: 0.000s.
```
