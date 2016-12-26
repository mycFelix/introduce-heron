# [用 Java 语言编写并运行拓扑 (Writing and Launching a Topology in Java)](http://twitter.github.io/heron/docs/developers/java/topologies/)

***笔者注释：严格意义上讲，本小节的题目应该是：构建 Heron 拓扑指南***

一个可运行的拓扑不仅仅需要像 spout 或 bolt 这样的组件，还需要正确定义他们之间关系以及赋予其合适的配置。

### 安装 Heron 开发所用的 API (Install Heron APIs for development)

在我们开始着手编写拓扑之前，我们需要安装 Heron API 并将其导入至拓扑工程的依赖库中。

* 点击 [releases page](https://github.com/twitter/heron/releases) 下载对应平台的 API 安装脚本。比如 Mac OS X (`darwin`) 对应的脚本名称为 `heron-api-install-${heronVersion}-darwin.sh`

* 脚本下载完成后，请在安装时使用 `--user` 参数

* 安装完成后，可以将 `~/.heronapi/heron-storm.jar` 导入到项目依赖库中。这样，就能开始具体的开发工作了。 ***笔者注释：也可以使用 maven***

### Maven 整合 (Maven Integration)

当然，您也可以将 API 添加至 Maven 项目的 `pom.xml` 文件中。可参照如下配置：

```xml
<dependency>
  <groupId>com.twitter.heron</groupId>
  <artifactId>heron-storm</artifactId>
  <version>${heronVersion}</version>
</dependency>
```

### 编写拓扑 (Writing your own topology)

我们会在 [Spouts](http://twitter.github.io/heron/docs/developers/java/spouts/) 和 [Bolts](http://twitter.github.io/heron/docs/developers/java/bolts/) 具体讨论如何实现它们。

在定义好 spout 和 bolt 之后，我们可以用 [`TopologyBuilder`](http://twitter.github.io/heron/api/com/twitter/heron/api/topology/TopologyBuilder.html) 类来组装拓扑。这个类主要有两个方法：

* `setBolt(String id, IRichBolt bolt, Number parallelismHint)`: `id` 是当前 bolt 的唯一标识名称, `bolt` 是这个 bolt 类的实例,  `parallelismHint` 是设置这个 bolt 运行时并行度

* `setSpout(String id, IRichSpout spout, Number parallelismHint)`: `id` 是当前 bolt 的唯一标识名称, `spout` 是这个 spout 类的实例,  `parallelismHint` 是设置这个 spout 运行时并行度

代码示例如下：

```java
TopologyBuilder builder = new TopologyBuilder();
builder.setSpout("word", new TestWordSpout(), 5);
builder.setBolt("exclaim", new ExclamationBolt(), 4);
```

如何定义 spout 和 bolt 的关系是非常重要的，否则拓扑将无法正常传递数据。目前有支持以下几种 tuple 分组策略：

* Fields Grouping: tuple 按照指定的字段(field)分组，相同字段值(field's value)的 tuple 会发给同一个 bolt 实例处理。
* Global Grouping: 把所有数据源创建的元组发送给单一目标实例（即拥有最低ID的任务）
* Shuffle Grouping: tuple 会随机发送到不同的 bolt 实例处理。
* None Grouping: 目前，与 Shuffler Grouping 效果相同。
* All Grouping: 所有 tuple 都被发送到每一个 bolt 实例上。
* Custom Grouping: 用户自定义分组策略。

如下代码片段便是使用 shuffle grouping 的方式来连接 `word` spout 和 `exclaim` bolt

```java
builder.setBolt("exclaim", new ExclamationBolt(), 4)
  .shuffleGrouping("word");
```

在 spout 和 bolt 的串连方式确定后，我们就可以构建拓扑了。

```java
HeronTopology topology = builder.createTopology();
```

点击如下链接查看 [`ExclamationTopology`](https://github.com/twitter/heron/blob/master/heron/examples/src/java/com/twitter/heron/examples/ExclamationTopology.java) 拓扑。如想参看更多的示例，请点击 [`examples package`](https://github.com/twitter/heron/tree/master/heron/examples/src/java/com/twitter/heron/examples)。

---

***笔者后记***

熟悉 Storm 的开发者会发现：Heron 的拓扑构建方式几乎和 Storm 是一样的。这也体现了 Heron 的设计理念：全面兼容 Storm API。看起来一切完美，但这里需要注意的是：文档中仅仅给出了类名，而没有告知包名，也就是 `package name` ，而这是非常关键的。如果你引用 com.twitter.heron:heron-storm ，你会发现三种 `namespace`：

* com.twitter.heron.api.* : 其包含的是 Heron 的 API ，文档中提到的 `HeronTopology` 便来自其中，同理 `TopologyBuilder` 也是。它只能解析 `com.twitter.heron.api.*` 名下面的类。其实不难理解，因为总不能指望 `org.apache.storm.TopologyBuilder` 类返回一个 `HeronTopology` 对象吧？

* org.apache.storm.* : 其包含 Apache Storm 1.0.x 后的 API，支持拓扑从 Apache Storm 迁移到 Heron 兼容性。

* backType.storm.* : 其包含 Apache Storm 0.x.x 后的 API，支持拓扑从 Apache Storm 迁移到 Heron 兼容性。

综上所述，在 Heron 拓扑的相关开发中，开发者自身应非常明确自己所使用的 Heron API 或 Storm API 版本。笔者使用 `org.apache.storm.*` ，基本不用更改拓扑代码，便能直接部署。如使用 `com.twitter.heron.api.*` ，至少 `TopologyBuilder` 处需要做响应的修改。根据 Heron 官网的建议，Heron API 可能还会在后续 Release 版本中发生改动，目前使用 `org.apache.storm.*` 是比较稳妥的方案。不过有些功能也仅支持 Heron API，比如仿真器模式(simulate mode)。

关于编写拓扑的更详细内容，我们后续的专题中深入讨论。
