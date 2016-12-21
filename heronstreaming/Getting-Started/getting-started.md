# 快速上手指南

学习 Heron 的最佳实践是安装并运行 Heron 提供的可运行版本。当前 Heron 仅支持以下版本：

* Mac OS X
* Ubuntu >= 14.04

如果需要在其他平台上运行 Heron，请参考文档[Heron Developers](http://twitter.github.io/heron/docs/developers/compiling/compiling/)，按照所需平台，自行编译源代码。


## 第一步 --- 下载可执行脚本 

请点击 Heron 的[releases page](https://github.com/twitter/heron/releases)，并下载对应平台的安装脚本，形如：

* `heron-client-install-{{% heronVersion %}}-PLATFORM.sh`
* `heron-tools-install-{{% heronVersion %}}-PLATFORM.sh`

举个例子，如果你是 Mac OS X 用户(`darwin`)，你需要下载 `heron-client-install-{{% heronVersion %}}-darwin.sh`，`heron-tools-install-{{% heronVersion %}}-darwin.sh` 这两个脚本. ***笔者注释：如果是 centos 用户，请对应下载 centos 版本***

脚本下载完毕之后，你就可以使用 `--user` 参数来运行脚本并安装 Heron 了。具体执行命令可以参考

```
$ chmod +x heron-client-install-VERSION-PLATFORM.sh
$ ./heron-client-install-VERSION-PLATFORM.sh --user
Heron client installer
----------------------

Uncompressing......
Heron is now installed!

Make sure you have "${HOME}/bin" in your path.
...
```

接下来，设置环境变量

```
$ export PATH=$PATH:~/bin
```

现在，我们可以继续安装 Heron tools，具体执行命令可以参考

```
$ chmod +x heron-tools-install-VERSION-PLATFORM.sh
$ ./heron-tools-install-VERSION-PLATFORM.sh --user
Heron tools installer
---------------------

Uncompressing......
Heron Tools is now installed!
...
```

这时，我们可以执行以下命令来验证 Heron 是否安装成功：

```
$ heron version
heron.build.version : {{% heronVersion %}}
heron.build.time : Sat Aug  6 12:35:47 PDT 2016
heron.build.timestamp : 1470512147000
heron.build.host : ${HOSTNAME}
heron.build.user : ${USERNAME}
heron.build.git.revision : 26bb4096130a05f9799510bbce6c37a69a7342ef
heron.build.git.status : Clean
```

## 第二步 --- 执行样例拓扑 

***笔者注释：Heron 的开发组为我们提供了丰富的样例拓扑，以便我们学习 Heron 的运行机制。同时运行样例拓扑也是一个不错的学习 Heron CLI 命令的途径***

如果在安装 Heron 时设置了 `--user` 参数，样例拓扑将会在 `~/.heron/examples` 文件夹下。你可以使用 [Heron CLI 命令](http://twitter.github.io/heron/docs/operators/heron-cli/) 提交[拓扑](http://twitter.github.io/heron/docs/concepts/topologies/)

```
# Submit ExclamationTopology locally in deactivated mode.
$ heron submit local \
~/.heron/examples/heron-examples.jar \
com.twitter.heron.examples.ExclamationTopology \
ExclamationTopology \
--deploy-deactivated

INFO: Launching topology 'ExclamationTopology'
...
[2016-06-07 16:44:07 -0700] com.twitter.heron.scheduler.local.LocalLauncher INFO: \
For checking the status and logs of the topology, use the working directory \
$HOME/.herondata/topologies/local/${ROLE}/ExclamationTopology # working directory

INFO: Topology 'ExclamationTopology' launched successfully
INFO: Elapsed time: 3.409s.
```

此时，拓扑将会被*提交*到本地的 Heron 集群，但此时拓扑还处于未激活状态，激活过程可以参考第五步

需要注意的是，输出信息会显示拓扑是否已经被提交到指定的工作目录中。

如需进一步确认，可以执行以下命令：

```
$ ls -al ~/.herondata/topologies/local/${ROLE}/ExclamationTopology
-rw-r--r--   1 username  role     2299 Jun  7 16:44 ExclamationTopology.defn
-rw-r--r--   1 username  role        5 Jun  7 16:44 container_1_exclaim1_1.pid
-rw-r--r--   1 username  role        5 Jun  7 16:44 container_1_word_2.pid
drwxr-xr-x  11 username  role      374 Jun  7 16:44 heron-conf
drwxr-xr-x   4 username  role      136 Dec 31  1969 heron-core
-rwxr-xr-x   1 username  role  2182564 Dec 31  1969 heron-examples.jar
-rw-r--r--   1 username  role        5 Jun  7 16:44 heron-executor-0.pid
-rw-r--r--   1 username  role        0 Jun  6 13:33 heron-executor.stderr
-rw-r--r--   1 username  role    17775 Jun  7 16:44 heron-executor.stdout
-rw-r--r--   1 username  role        5 Jun  7 16:44 heron-shell-0.pid
-rw-r--r--   1 username  role        5 Jun  7 16:44 heron-tmaster.pid
drwxr-xr-x  25 username  role      850 Jun  7 16:44 log-files
-r--r--r--   1 username  role     4506 Jun  8 12:05 metrics.json.metricsmgr-0.0
-rw-r--r--   1 username  role        5 Jun  7 16:44 metricsmgr-0.pid
-r-xr-xr-x   1 username  role      279 Dec 31  1969 release.yaml
-rw-r--r--   1 username  role        5 Jun  7 16:44 stmgr-1.pid
```

`log-files` 是 Heron 运行时所有实例的日志文件夹，可以执行以下命令查看：

```
$ ls -al ~/.herondata/topologies/local/${ROLE}/ExclamationTopology/log-files
total 1018440
-rw-r--r--   1 username  role   94145427 Jun  8 12:06 container_1_exclaim1_1.log.0
-rw-r--r--   1 username  role   75675435 Jun  7 16:44 container_1_word_2.log.0
-rw-r--r--   1 username  role  187401024 Jun  8 12:06 gc.container_1_exclaim1_1.log
-rw-r--r--   1 username  role  136318451 Jun  8 12:06 gc.container_1_word_2.log
-rw-r--r--   1 username  role      11039 Jun  8 11:16 gc.metricsmgr.log
-rw-r--r--   1 username  role        300 Jun  7 16:44 heron-shell.log
-rw-r--r--   1 username  role      29631 Jun  7 16:44 heron-ExclamationTopology-scheduler.log.0
-rw-r--r--   1 username  role    2382215 Jun  7 15:16 heron-stmgr-stmgr-1.username.log.INFO
-rw-r--r--   1 username  role       5976 Jun  7 16:44 heron-tmaster-ExclamationTopology2da9ee6b-c919-4e59-8cb0-20a865f6fd7e.username.log.INFO
-rw-r--r--   1 username  role   12023368 Jun  8 12:06 metricsmgr-0.log.0

```

## 第三步 -- 启动 Heron Tracker

***笔者注释：Heron Tracker 可以理解为是 Heron 提供的监听器，是 Heron Cluster 与 Heron UI 之间的通信桥梁。Heron Tracker 是 heron-tools 的主要组成部分。***

[Heron Tracker](http://twitter.github.io/heron/docs/operators/heron-tracker/) 是收集 Heron Cluster 信息的 Web 服务。可以通过 `heron-tracker` 命令来启动它。

```
$ heron-tracker
... Running on port: 8888
... Using config file: $HOME/.herontools/conf/heron_tracker.yaml
```

此时可以在浏览器中输入 [http://localhost:8888](http://localhost:8888) 来查看 Heron Tracker，你将会看到如下信息：
![alt_tag](http://twitter.github.io/heron/img/heron-tracker.png)

如果想了解 Heron Tracker 的更多信息，请参考文档 [Heron Tracker Rest API](http://twitter.github.io/heron/docs/operators/heron-tracker-api/)

## 第四步 -- 启动 Heron UI

Heron UI 是 Heron 为用户提供的可视化拓扑信息查看页面，类似于 Storm UI，它的正常启动依赖于 Heron Tracker 的正常运行。可以使用如下命令运行 Heron UI. ***笔者注释：默认端口是 8889，可以使用 --port=${port} 参数来指定启动端口***

```
$ heron-ui
... Running on port: 8889
... Using tracker url: http://localhost:8888
```

如想了解 Heron UI 的更多信息，请参考文档 [Heron UI Usage Guide](http://twitter.github.io/heron/docs/developers/ui-guide/)

## 第五步 -- 探索 Heron CLI 命令

我们在第二步中已经将拓扑提交到本地集群。Heron CLI 命令集提供了更多有意思的命令，如 `activate`、`deactivate` 还有 `kill` 等等。

```
$ heron activate local ExclamationTopology
$ heron deactivate local ExclamationTopology
$ heron kill local ExclamationTopology
```

对于第一条激活命令，其可能正确的输出为：

```
INFO: Successfully activated topology 'ExclamationTopology'
INFO: Elapsed time: 1.980s.
```

如想了解更多命令，请参考文档 [topology
lifecycles](http://twitter.github.io/heron/docs/concepts/topologies/#topology-lifecycle)

如想查看所有的 Heron CLI 命令，请运行 `heron`：

```
usage: heron <command> <options> ...

Available commands:
    activate           Activate a topology
    deactivate         Deactivate a topology
    help               Prints help for commands
    kill               Kill a topology
    restart            Restart a topology
    submit             Submit a topology
    version            Print version of heron-cli

For detailed documentation, go to http://heronstreaming.io
```

如进一步想查看帮助文档，请运行 `heron help $COMMAND`，举例如下：

```
$ heron help submit
usage: heron submit [options] cluster/[role]/[environ] topology-file-name topology-class-name [topology-args]

Required arguments:
  cluster/[role]/[env]  Cluster, role, and environ to run topology
  topology-file-name    Topology jar/tar/zip file
  topology-class-name   Topology class name

Optional arguments:
  --config-path (a string; path to cluster config; default: "$HOME/.heron/conf")
  --config-property (key=value; a config key and its value; default: [])
  --deploy-deactivated (a boolean; default: "false")
  -D DEFINE             Define a system property to pass to java -D when
                        running main.
  --verbose (a boolean; default: "false")
```

## 第六步 -- 样例拓扑的说明

***笔者注释：不逐一翻译每一个样例拓扑的具体工作目的，开发者可以依次运行用以练习***

样例拓扑的源代码已在 GitHub 上托管，可以点击[链接](https://github.com/twitter/heron/tree/master/heron/examples/src/java/com/twitter/heron/examples).

样例拓扑包括：

* `AckingTopology.java` --- A topology with acking enabled.
* `ComponentJVMOptionsTopology.java` --- A topology that supplies JVM options
  for each component.
* `CustomGroupingTopology.java` --- A topology that implements custom grouping.
* `ExclamationTopology.java` --- A spout that emits random words to a bolt that
  then adds an exclamation mark.
* `MultiSpoutExclamationTopology.java` --- a topology with multiple spouts.
* `MultiStageAckingTopology.java` --- A three-stage topology. A spout emits to a
  bolt that then feeds to another bolt.
* `TaskHookTopology.java` --- A topology that uses a task hook to subscribe to
   event notifications.

## 问题定位

如有任何问题请参考文档 [Quick Start Troubleshooting](http://twitter.github.io/heron/docs/getting-started-troubleshooting/)

---

***笔者后记***

- **第一次接触 Heron 的同学可能会产生一个疑问，不需要我们做任何单机设置，仅仅运行安装文件我们就能运行拓扑了么？** *的确，与 Storm 或者其他大数据计算框架不同的是，如果仅仅想运行样例拓扑，并不需要任何额外、繁琐的设置。Heron 的开发者们我们提供了非常便利的运行环境。*

- **示例拓扑运行在哪里？** *示例拓扑以进程形式运行在你当前机器的内存中，相关工作目录可以在`$HOME/.herondata/topologies/local/${ROLE}/ExclamationTopology`查看，其中 ${ROLE} 为 `--user`账户名。*

- **Heron Tracker 一定要运行么？** *这是个非常好的问题，如果你不需要运行 Heron UI 也不需要用任何可视化的查看拓扑的运行状况，那么你完全没有必要运行 Heron-Tracker。如果反之，那么你必须保证 Heron Tracker 的正常运转。*

- **搭建 Storm 集群时我需要很多设置，那么如何搭建 Heron Cluster？** *严格意义上讲，Heron 并不需要像 Storm 那样构建集群，Heron 可以利用现有的任何资源调度系统运行拓扑如 Aurora、Mesos 或者是 YARN。当以 Local 模式运行时，Heron 的开发者们为我们提供了默认的调度器从而省去了很多繁琐的设置。*

- **如何将拓扑部署到资源调度系统上，如 YARN？** *这将在后续进行更深入的讲解。*

- **cluster/[role]/[env] 分别代表什么？** *这可能是很多 Storm 使用者迷惑的点。通俗来讲，cluster 是你拓扑运行的集群模式，如 local 表示本地集群；role 表示拓扑运行的角色，这个角色有权限的概念，A 提交的拓扑只能由 A 进行相关操作，role 默认使用当前提交者 ${Home}；env 表示运行环境，如 test/prd/default 等等。在执行 Heron CLI 命令时，Heron 会校验三者正确性，如任意一个校验失败，命令无法继续执行。*



