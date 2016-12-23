# Markdown Guide

目前 markdown 已经一个程序界主流的文字排版语言。关于 markdown 语法本身，我就不多赘述了，网络上有非常多的参考资料，如：[Github Markdown Guide](https://guides.github.com/features/mastering-markdown/) 或 [MarkDown 中文网](www.markdown.cn/)。

我在这里想说明一些文字排版规范：

* 中英文混排，请用空格区分，举例如下

```
目前 markdown 已经一个程序界主流的文字排版语言。

# 注意在 Apache Storm 超链接前后都有空格
Heron 是继 [Apache Storm](http://storm.apache.org) 之后的下一代实时计算框架

# 注意在 `` 之中的英文字符两侧也有空格区分
当 Heron Instance 运行后，`StreamManagerClient` 会向 stream manager 注册并与其建立 1 个稳定的链接。
```

* 中文与数字混排，请用空格区分，举例如下

```
每个拓扑还会运行 1 个 Metrics Manager 实例
```

* 英文特殊专有名词或需要在译文中保留原文的词汇，请用英文括号示例，不用添加空格区分，举例如下

```
它可以理解为集群范畴(cluster-wide)的拓扑信息获知器
```

* “笔者注释”用加粗斜体字表示，特殊场景可用文本框标识，举例如下：

```
***笔者注释：关于双线程处理模型，论文中有非常详尽的描述。***
```

* 中文加粗字体与正文之间不用加空格，举例如下：

```
如果想更深入的了解相关信息，强烈**建议**阅读这篇论文
```
