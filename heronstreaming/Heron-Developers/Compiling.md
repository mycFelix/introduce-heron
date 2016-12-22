# 编译 Heron (Compiling Heron)

***笔者注释：大家拿到开源软件的第一步就是进行编译，Heron 的编译有一些小技巧。本文档旨会先对官方文档进行翻译，而后再介绍笔者的一些经验***

现今，Heron 支持 Mac OS X 10.10 、Ubuntu 14.04 以及 CentOS7 三个平台，本文档旨在介绍 Heron 的构建体系，如需进一步了解相关平台的编译细节，请参考如下文档。

* [Building on Linux Platforms](../Heron-Developers/Compiling-on-Linux.md)
* [Building on Mac OS X](../Heron-Developers/Compiling-on-MacOSX.md)

Heron 支持开发者进行全编译、独立组件编译和发布可执行版本等操作。

可以在 [Testing Heron](http://twitter.github.io/heron/docs/contributors/testing/) 文档中找到运行 Heron 单元测试用例的相关指南。

## 编译前置条件 (Requirements)

如需编译 Heron，请先确认已安装以下组件：

* [Bazel](http://bazel.io/docs/install.html) = 0.3.1 不确定更高级的版本是否可以正常编译
* [Java
  8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) Ja是 Bazel 的运行时必要环境。Heron 的拓扑可使用 Java 7 或者以上版本开发，同时 Heron 所有运行时的 Jar 也兼容 Java 7.
* [Autoconf](http://www.gnu.org/software/autoconf/autoconf.html) >=
  2.6.3
* [Automake](https://www.gnu.org/software/automake/) >= 1.11.1
* [GNU Make](https://www.gnu.org/software/make/) >= 3.81
* [GNU Libtool](http://www.gnu.org/software/libtool/) >= 2.4.6
* [gcc/g++](https://gcc.gnu.org/) >= 4.8.1 (Linux platforms) ***笔者注释：Heron 使用 C++、Java、Python 三种语言开发，所以需要 C++ 的编译环境***
* [CMake](https://cmake.org/) >= 2.6.4
* [Python](https://www.python.org/) >= 2.7 (not including Python 3.x)
* [Perl](https://www.perl.org/) >= 5.8.8

定义 `CC` 和 `CXX` 环境变量

```bash
$ export CC=/your-path-to/bin/c_compiler
$ export CXX=/your-path-to/bin/c++_compiler
$ echo $CC $CXX
```

## 安装 Bazel (Installing Bazel)

Heron 使用 [Bazel](http://bazel.io) 做为其构建工具。Bazel 的发布版本可点击如下[链接](https://github.com/bazelbuild/bazel/releases)，可在[安装指南](http://bazel.io/docs/install.html) 中找到相关的安装必要信息。

可以运行 `bazel version` 命令来校验 Bazel 是否安装成功并能正常运行。

## 设置 Bazel (Configuring Bazel)

***笔者注释：这一步非常关键，执行本环节命令，并确认所有组件已经安装成功后，Heron 会生成必要的安装文件，该文件是后续编译中不可缺少的一环***

可以执行 Python 脚本来自动配置 Bazel 的编译运行时信息。

```bash
$ cd /path/to/heron
$ ./bazel_configure.py
```

## 编译 (Building)

### Bazel 的系统环境 (Bazel OS Environments)

利用 Bazel 构建时可以通过 `--config` 参数来指定编译平台。目前 Heron 支持的编译平台如下：

* `darwin` (Mac OS X)
* `ubuntu` (Ubuntu 14.04)
* `centos5` (CentOS 5) ***笔者注释：目前 Heron 已支持 centos7 编译，笔者亲测可用***

以 Mac OS X (`darwin`) 平台举例，可以使用如下命令进行全编译：

```bash
$ bazel build --config=darwin heron/...
```

在发布生产环境版本时，可以使用 `opt` 参数开启编译期优化选项。该选项开启后，编译时间也会有所增加。

```bash
$ bazel build -c opt --config=PLATFORM heron/...
```

### 构建全部组件 (Building All Components)

可以利用 Bazel 构建可执行的 shell 安装脚本或者 tar 安装包。如想一次性构建 Heron 的所有组件，可以使用如下命令：

```bash
$ bazel build --config=PLATFORM scripts/packages:binpkgs
$ bazel build --config=PLATFORM scripts/packages:tarpkgs
```

命令执行结束后，结果文件会放在 `bazel-bin` 文件夹中。例如，`heron-tracker` 的可执行文件目录为 `bazel-bin/heron/tools/tracker/src/python/heron-tracker`.

### 编译指定的组件 (Building Specific Components)

如果并不想进行耗时的全编译，你可以选择编译某个独立的组件。如只编译 `Heron Tracker`，可以执行以下命令

```bash
$ bazel build --config=darwin heron/tools/tracker/src/python:heron-tracker
```

## 执行单元测试样例 (Testing Heron)

执行单元测试样例的具体细节可以参考 [Testing Heron](http://twitter.github.io/heron/docs/contributors/testing/)

---
***笔者后记***

本文档仅仅从介绍了一下 Heron 的编译概要，具体到某个指定平台的编译细节，可以在相关文档中找到。Heron 使用 C++、Java、Python 三种语言开发，所以以上三种编译环境必须按照 Heron 所需准备妥当，不然是无法进行正常编译的。如果说一定有什么特别需要注意的，那就是尽量要有一个畅通无阻的网络环境，以便编译时可以正常下载依赖 Jar。

- **在 Linux 平台上运行 Heron 是否可以使用 darwin 版本的？** *答案是不行的，Heron 有部分代码使用 C++ 编写，平台不同决定了 C++ 的运行时环境也不同*

- **如果我的生产环境是 Linux 平台，那我如何在 Mac 上进行开发呢？** *这是一个非常好的、也是非常常见的问题。同时也是笔者最初接触 Heron 困扰的问题之一。手边没有 Linux 机器，但还要编译对应的版本。好不容易找到一台 Linux 机器，配置一系列运行时依赖库又非常麻烦，怎么办呢？Heron 是为我们提供对应解决方案的。Heron 提供了一种利用 Docker 的编译方式。我们可以在 Docker 中编译任何已经提供的平台的 Heron 版本。当然，前提条件是，你有一个非常不错的网络环境，可以正常下载相关依赖。*

- **编译环境报错怎么办？** *据笔者自身经验，网络环境多半会是真凶*

- **编译时有没有什么需要注意的？** *确保你的运行时环境已经安装了所有 Heron 所需要的依赖库，确保你已经按照文档介绍的版本进行 Bazel 的安装，确保你有一个干净的网络环境，然后就可以放心大胆的执行命令了！*
