# [Apache Hadoop YARN Cluster](http://twitter.github.io/heron/docs/operators/deployment/schedulers/yarn/)

***笔者注释：Heron 现已发布 0.14.5 版本，本文档将略过 0.14.3 以下版本的配置&部署方案***

除了 Heron 自带的 [Aurora](http://twitter.github.io/heron/docs/operators/deployment/schedulers/aurora/) 调度器，Heron [Apache REEF](https://reef.apache.org/)  框架实现 YARN 调度器，以便拓扑可以在 YARN 集群上运行。

YARN 调度器的核心功能：

* **异构容器分配(heterogeneous container allocation)**：调度器会向 YARN 的资源管理器 [RM](http://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YARN.html) 申请异构容器(heterogeneous containers)。这意味着，拓扑不会过分申请资源。
* **资源重用(container reuse)**：REEF 框架尽力保证 YARN 容器的可用性，即便是在拓扑重启(restarts)时也是如此。

## Topology deployment on a YARN Cluster

和其他调度器一样，YARN 调度器目前也使用 Heron 命令行工具提交拓扑。本文档假设 Hadoop YARN 已配置正确且可正常运行。

拓扑提交时 YARN 调度器会做以下几步操作：

1. REEF 客户端会拷贝 `Heron Core package` 和 `topology package` 至分布式文件系统上
2. 为拓扑启动 YARN 的 Application Master (AM)
3. AM 随后在其进程里调用 `Heron Scheduler`
4. 为 Topology Master 和其他 Heron Instance 分配 container。最终每个拓扑会有 `N+2` 个 container

### Configuring the Heron client classpath

在 0.14.3 版本发布之后，开发者已不再需要手工拷贝任何 `Hadoop classpath Jars` 到指定目录了。Heron 开发者在命令行中添加了 `extra-launch-classpath` 参数(可参阅[PR](https://github.com/twitter/heron/issues/1245))，让拓扑的提交方式变得更加简单。用户直接指定 `Hadoop classpath Jars` 路径即可。

> **Tips**
>
>***请认真阅读以下内容，该内容适用于 Heron 0.14.2(包含)以后的任何版本***
>
>对于`本地文件系统状态管理器`(localfs-state-manager)
>
>* common-cli jar 的版本号应不低于(>=) 1.3.1
>
>对于 `Zookeeper 状态管理器`(zookeeper-state-manager)
>
>* common-cli jar 的版本号应不低于(>=) 1.3.1
>* curator-framework jar 的版本号应不低于(>=) 2.10.0
>* curator-client jar 的版本号应不低于(>=) 2.10.0

### Configure the YARN scheduler

可以在 [conf/yarn](https://github.com/twitter/heron/tree/master/heron/config/src/yaml/conf/yarn) 路径下查看默认的 YARN 调度器配置。缺省配置使用本地文件系统状态管理器 `local state manager`，它仅在单节点的 YARN 环境中使用。如果是 YARN 集群环境，请使用 Zookeeper 状态管理器

1. YARN 通用的 Heron Launcher： `YarnLauncher`
2. YARN 通用的 Heron Scheduler： `YarnScheduler`
3. YARN 集群环境的状态管理器设置：`com.twitter.heron.statemgr.zookeeper.curator.CuratorStateManager`
4. `YarnLauncher` 已经优化了 uploader 的工作，所以使用 `NullUploader` 即可

## 拓扑管理 (Topology management)

### 拓扑提交 (Topology Submission)

**命令**

`$ heron submit yarn heron-examples.jar com.twitter.heron.examples.AckingTopology AckingTopology --extra-launch-classpath <extra-classpath-value>`

>**Tips**
>
>1.关于 `--extra-launch-classpath` 参数。您可以将所有 `hadoop-lib-jars` 放置到一个路径下，也可以将 `hadoop classpath` 命令中的路径用冒号(:)连接起来一起提交。***如果有 Jar 路径缺失，提交过程将会失败***
>
>2. 如果您希望将拓扑提交至 YARN 的指定队列上运行，您可以在 `--config-property` 参数中添加 `heron.scheduler.yarn.queue` 配置，如提交拓扑至 `test` 队列：`--config-property heron.scheduler.yarn.queue=test` 。同样，这个配置也可以在 [conf/yarn/scheduler](https://github.com/twitter/heron/blob/master/heron/config/src/yaml/conf/yarn/scheduler.yaml) 中找到。默认队列 YARN 提供的 `default` 队列。

**输出样例**

```bash
INFO: Launching topology 'AckingTopology'
...
...
Powered by
     ___________  ______  ______  _______
    /  ______  / /  ___/ /  ___/ /  ____/
   /     _____/ /  /__  /  /__  /  /___
  /  /\  \     /  ___/ /  ___/ /  ____/
 /  /  \  \   /  /__  /  /__  /  /
/__/    \__\ /_____/ /_____/ /__/

...
...
com.twitter.heron.scheduler.yarn.ReefClientSideHandlers INFO:  Topology AckingTopology is running, jobId AckingTopology.
```

**验证**

Visit the YARN http console or execute command `yarn application -list` on a yarn client host.

```bash
Total number of applications (application-types: [] and states: [SUBMITTED, ACCEPTED, RUNNING]):1
                Application-Id	    Application-Name	    Application-Type	      User	     Queue	             State	       Final-State	       Progress	                       Tracking-URL
application_1466548964728_0004	      AckingTopology	                YARN	     heron	   default	           RUNNING	         UNDEFINED	             0%	                                N/A
```

### 终止拓扑 (Topology termination)

**命令**

`$ heron kill yarn AckingTopology`

### 日志文件 (Log File location)

Heron 和 REEF 的日志文件路径如下所示，注意文件路径示例皆为 HDFS 路径：

1. 拓扑 AM 日志路径 :
`<LOG_DIR>/userlogs/application_1466548964728_0004/container_1466548964728_0004_01_000001/driver.stderr`

1. 调度器的日志路径将会在第一个 container 中 :
`<NM_LOCAL_DIR>/usercache/heron/appcache/application_1466548964728_0004/container_1466548964728_0004_01_000001/log-files`

1. Topology Master(TM) 的启动日志路径将会在自己所在的 container 中:
`<LOG_DIR>/userlogs/application_1466548964728_0004/container_1466548964728_0004_01_000002/evaluator.stderr`

1. Topology Master(TM) 的日志将会在拓扑的第二个 container 中:
`<NM_LOCAL_DIR>/usercache/heron/appcache/application_1466548964728_0004/container_1466548964728_0004_01_000002/log-files`

1. Worker 日志将会在 Yarn NodeManager 的余下的 container 的本地目录中。

---

***笔者后记***

在 0.14.3 版本以前，Heron on YARN 的部署方案是比较繁琐的，之后的版本加入了 `extra-launch-classpath` 参数，使得这个操作能简单许多。如果搭配 `heronrc` 使用将会使提交过程变得更加简单。关于 YARN 的部署，笔者个人有一些个人建议，但不代表任何官方观点：

1. YARN 集群最好使用 CentOS 7 搭建，如果使用 CentOS 6 或 5，可能会有一些复杂的 C++ 链接库问题让人十分挠头，比如：libunwind。
2. 使用 Zookeeper 做状态管理器，这一点官方文档中已经着重指出了。
3. 目前，如果使用 Hadoop 2.7.0 版本，最好需要在 `yarn-site.xml` 中添加 `yarn.application.classpath` 信息。例如：

```xml
   <property>
        <name>yarn.application.classpath</name>
        <value>
            $HADOOP_HOME/etc/hadoop,
            $HADOOP_HOME/share/hadoop/common/lib/*,
            $HADOOP_HOME/share/hadoop/common/*,
            $HADOOP_HOME/share/hadoop/hdfs,
            $HADOOP_HOME/share/hadoop/hdfs/lib/*,
            $HADOOP_HOME/share/hadoop/hdfs/*,
            $HADOOP_HOME/share/hadoop/yarn/lib/*,
            $HADOOP_HOME/share/hadoop/yarn/*,
            $HADOOP_HOME/share/hadoop/mapreduce/lib/*,
            $HADOOP_HOME/share/hadoop/mapreduce/*,
            $HADOOP_HOME/contrib/capacity-scheduler/*.jar,
            $HADOOP_HOME/share/hadoop/yarn/*,
            $HADOOP_HOME/share/hadoop/yarn/lib/*
        </value>
    </property>
```
