# 迁移 Storm 拓扑至 Heron

***笔者注释：Heron 目前支持了 Storm 的 API，可以做到近乎无痛的 Storm 拓扑迁移。不过近乎无痛并不等于真无痛，偶尔还是有一些小瑕疵，关于迁移的更深入的问题，笔者会在后续的章节中单独讨论***

后项兼容 [Apache Storm](http://storm.apache.org/index.html) 是 Heron 的设计理念之一。开发者仅仅需要调整 `pom.xml` [Maven configuration file](https://maven.apache.org/pom.html) 中的若干配置便可以无痛的将拓扑从 Storm 中迁移至 Heron。

***笔者注释：Heron 既支持 `backtype.storm` 也支持 `org.apache.storm`，如果可以的话还是尽量使用 `org.apache.storm` API，这仅仅个人的建议***

## 第一步 --- 下载 Heron API 安装脚本 (Download Heron API binaries with an installation script)

点击 [Releases 页面](https://github.com/twitter/heron/releases)，下载对应版本的 API 安装脚本，脚本符合如下命名规则：

```
heron-api-install-${heronVersion}-PLATFORM.sh
```

以 Mac OS X (`darwin`) 为例，API 安装脚本名为：`heron-api-install-${heronVersion}-darwin.sh`.

脚本下载完成后，可以使用 `--user` 和 `--maven` 参数来执行，示例如下

```
$ chmod +x heron-api-install-${heronVersion}-PLATFORM.sh
$ ./heron-api-install-${heronVersion}-PLATFORM.sh --user --maven
Heron API installer
-------------------

Installing jars to local maven repo.
tar xfz /var/folders/8r/x6dwcnkn4p9_rgwvq_3jg6y00000gn/T/heron.XXXX.EnJDpZNb/heron-api.tar.gz
-C /var/folders/8r/x6dwcnkn4p9_rgwvq_3jg6y00000gn/T/heron.XXXX.EnJDpZNb

Heron API is now installed!

See http://heronstreaming.io/docs/getting-started for how to use Heron.

heron.build.version : '${heronVersion}'
heron.build.time : ...
heron.build.timestamp : ...
heron.build.host : ${HOSTNAME}
heron.build.user : ${USERNAME}
heron.build.git.revision : ...
heron.build.git.status : Clean
```

此时，Heron API 将会被安装到本地的 Maven 仓库中

```
$ ls ~/.m2/repository/com/twitter/heron
heron-api
heron-spi
heron-storm
```

## 第二步 --- 在 `pom.xml` 中添加 Heron 依赖 (Add Heron dependencies to `pom.xml`)

将如下代码拷贝至已有的 `pom.xml` [依赖](https://maven.apache.org/pom.html#Dependencies)文件中。

```
<dependency>
  <groupId>com.twitter.heron</groupId>
  <artifactId>heron-api</artifactId>
  <version>SNAPSHOT</version>
  <scope>compile</scope>
</dependency>
<dependency>
  <groupId>com.twitter.heron</groupId>
  <artifactId>heron-storm</artifactId>
  <version>SNAPSHOT</version>
  <scope>compile</scope>
</dependency>
```

## 第三步 --- 在 `pom.xml` 文件中去除 Storm 依赖 (Remove Storm dependencies from `pom.xml`)

删除 Storm 的相关依赖代码

```
<dependency>
  <groupId>org.apache.storm</groupId>
  <artifactId>storm-core</artifactId>
  <version>storm-VERSION</version>
  <scope>provided</scope>
</dependency>
```

## 第四步 (可选) --- 在 `pom.xml` 文件中去除 Clojure 插件 (Remove the Clojure plugin from `pom.xml`)

如果 `pom.xml` 文件中有形如如下代码块的 [Clojure plugin](https://maven.apache.org/pom.html#Plugins) 的相关依赖，请删除。

```
<plugin>
  <groupId>com.theoryinpractise</groupId>
  <artifactId>clojure-maven-plugin</artifactId>
  <version>1.3.12</version>
  <extensions>true</extensions>
  <configuration>
    <sourceDirectories>
      <sourceDirectory>src/clj</sourceDirectory>
    </sourceDirectories>
  </configuration>
</plugin>
```


## 第五步 --- 执行 Maven 命令 (Run Maven commands)

运行 [Maven lifecycle](https://maven.apache.org/run.html) 命令：

```
$ mvn clean
$ mvn compile
$ mvn package
```

**注意**: [Storm Distribute RPC](http://storm.apache.org/releases/0.10.0/Distributed-RPC.html) 在 Heron 中以被废弃

## 第五步(可选) --- 运行已升级后的拓扑 (Launch your upgraded Heron topology)

此时，你可以根据具体需要使用 `heron submit` 运行升级后的拓扑，可以参考[快速上手指南](../Getting-Started/Local(Single-Node).md)，如下命令展示了将拓扑提交到 `local` 环境的命令示例。

```
$ heron submit local \
  ${basedir}/target/PATH-TO-PROJECT.jar \
  TOPOLOGY-FILE-NAME \
  TOPOLOGY-CLASS-NAME
```

使用 [快速上手指南](../Getting-Started/Local(Single-Node).md) 中的示例拓扑加以说明：

```
$ heron submit local \
  ~/.heron/examples/heron-examples.jar \ # 拓扑 Jar 文件所在路径
  com.twitter.heron.examples.ExclamationTopology \ # 拓扑中的 Java Class
  ExclamationTopology # The name of the topology
```

---

***笔者后记***

总体来讲，笔者个人的经验是：**迁移基本上是无痛的。** 当然，痛点也还是有的。如果你使用的是 `backtype.storm` API ，那么最真诚的建议是借此机会修改代码，将拓扑迁移到 `org.apache.storm`。毕竟 Apache Storm 也已经完成了类似的 API 转换，同时后续版本 (1.0.x 以后) 将会只提供 `org.apache.storm` 的更新。同时，也请注意一下 `Kryo` 序列化问题，如有一些类没有完成注册，也请注册一下。某些针对性的问题会在后续建立专题讨论。
