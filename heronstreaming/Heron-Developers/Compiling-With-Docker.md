# 利用 Docker 编译 Heron (Compiling With Docker)

您可能需要本地环境中编译不同平台的 Heron 可执行文件。Heron 提供了相关脚本文件，满足您利用 Docker 编译不同平台可执行文件的需求。

目前，仅支持编译 Ubuntu 14.04、Ubuntu 15.10 和 CentOS7 平台。如果您需要新的平台，您可以根据[编写新平台的 `Dockerfile` 小节](#编写新平台的-dockerfile-contributing-new-environments)的指引添加新文件。

## 前置编译条件 (Requirements)

* [Docker](https://docs.docker.com)

## 在虚拟进中运行 Docker (Running Docker in a Virtual Machine)

如果您在虚拟机(virtual machine,VM)中运行 Docker ，我们建议您对虚拟机进行一些设置，以便加速编译过程。打开 [VirtualBox](https://www.virtualbox.org/wiki/Downloads) 或者其他容器的**设置页面**，您可能需要调整处理器个数以及内存大小。

**注意**：在设置之前，您需要停止正在运行的虚拟机

![VirtualBox Processors](http://twitter.github.io/heron/img/virtual-box-processors.png)
![VirtualBox Memory](http://twitter.github.io/heron/img/virtual-box-memory.png)

## 编译 Heron (Building Heron)

Heron 在 `docker` 文件中提供了 `build-arfifacts.sh` 脚本，可以执行如下命令执行脚本：

```bash
$ cd /path/to/heron/repo
$ docker/build-artifacts.sh
```

单独执行这个脚本会显示脚本语法说明：

```bash
Usage: docker/build-artifacts.sh <platform> <version_string> [source-tarball] <output-directory>

Platforms Supported: darwin, ubuntu14.04, ubuntu15.10, centos7

Example:
  ./build-artifacts.sh ubuntu14.04 0.12.0 .

NOTE: If running on OSX, the output directory will need to
      be under /Users so virtualbox has access to.
```

以下为必选参数：

* `platform` --- 平台名称，目前支持的平台为 `ubuntu14.04`、`ubuntu15.10` 和 `centos7`。您可以根据[编写 Docker 文件小节](#contributing-new-environments)的指引添加新脚本。
* `version-string` --- 您期望发布的版本号。
* `output-directory` --- 编译后可执行文件的输出目录。

下面是命令样例：

```bash
$ docker/build-artifacts.sh ubuntu14.04 0.12.0 ~/heron-release
```

如上命令会在 Docker 中，根据 Ubuntu 14.04 平台的要求全编译 Heron 、生成可执行文件，并复制编译结果到 `~/heron-release` 目录中。

您也可以用在 Heron Source 中包含一个 tarball。默认配置中，脚本会针对当前 Heron 源码版本创建一个 tarball，并用它来构建 Heron。

**注意**：如果您使用 Mac OS X 平台，那么 Docker 必须在虚拟中运行。因此，你必须确认源 tarball 和目的路径是在您 `${Home}` 下的。例如，您不能将输出目录设置为 `/tmp`，因为 `/tmp` 会被引导至虚拟机的目录，而不是您宿主机目录。通常，虚拟机会自动加载宿主机的 `${Home}` 目录，且对其有访问权限。

构建完成后，您可以到指定的输出目录中查看 Heron 可执行文件，如：

```bash
$ ls ~/heron-release
heron-api-0.12.0-ubuntu14.04.tar.gz
heron-client-0.12.0-ubuntu14.04.tar.gz
heron-tools-0.12.0-ubuntu14.04.tar.gz
heron-client-install-0.12.0-ubuntu.sh  
heron-tools-install-0.12.0-ubuntu.sh
heron-api-install-0.12.0-ubuntu.sh     
heron-core-0.12.0-ubuntu.tar.gz
```

## 编写新平台的 `Dockerfile` (Contributing New Environments)

您可能已经注意到，Heron 源码的 `docker` 目录下有多个 [Dockerfiles](https://docs.docker.com/engine/reference/builder/) ，每一个都对应着指定平台。

如想支持新的平台，可以添加新的 `Dockerfile`，并用相关联的平台命名。举个例子，如果您想添加支持 Debian 8 的脚本，您可以将其命名为 `Dockerfile.debian8` 。您可以根据 [Docker 指南](https://docs.docker.com/engine/articles/dockerfile_best-practices/)所描述的内容，编辑新的文件。

在新的 `Dockerfile` 中，请确保至少包含以下信息：

***笔者注释：如下内容涉及脚本的具体编写情况，为避免歧义，基本保持原文***

### Step 1 --- The OS being used in a [`FROM`](https://docs.docker.com/engine/reference/builder/#from) statement.

Here's an example:

```dockerfile
FROM centos:centos7
 ```

### Step 2 --- A `TARGET_PLATFORM` environment variable using the [`ENV`](https://docs.docker.com/engine/reference/builder/#env) instruction.

Here's an example:

```dockerfile
ENV TARGET_PLATFORM centos
```

### Step 3 --- A general dependency installation script using a [`RUN`](https://docs.docker.com/engine/reference/builder/#run) instruction.

Here's an example:

```dockerfile
RUN apt-get update && apt-get -y install \
         automake \
         build-essential \
         cmake \
         curl \
         libssl-dev \
         git \
         libtool \
         libunwind8 \
         libunwind-setjmp0-dev \
         python \
         python2.7-dev \
         python-software-properties \
         software-properties-common \
         python-setuptools \
         unzip \
         wget
```

### Step 4 --- An installation script for Java 8 and a `JAVA_HOME` environment variable

Here's an example:

```dockerfile
RUN \
     echo oracle-java8-installer shared/accepted-oracle-license-v1-1 select true | debconf-set-selections && \
     add-apt-repository -y ppa:webupd8team/java && \
     apt-get update && \
     apt-get install -y oracle-java8-installer && \
     rm -rf /var/lib/apt/lists/* && \
     rm -rf /var/cache/oracle-jdk8-installer

ENV JAVA_HOME /usr/lib/jvm/java-8-oracle
```

#### Step 5 - An installation script for [Bazel](http://bazel.io/) version {{% bazelVersion %}} or above.
Here's an example:

```dockerfile
RUN wget -O /tmp/bazel.sh https://github.com/bazelbuild/bazel/releases/download/0.3.1/bazel-0.3.1-installer-linux-x86_64.sh \
         && chmod +x /tmp/bazel.sh \
         && /tmp/bazel.sh
```

### Step 6 --- Add the `bazelrc` configuration file for Bazel and the `compile.sh` script (from the `docker` folder) that compiles Heron

```dockerfile
ADD bazelrc /root/.bazelrc
ADD compile.sh /compile.sh
```

---

***笔者后记***

在[编译 Heron](../Heron-Developers/Compiling-on-MacOSX.md)的最后，我们提出这样的问题：“如果我用 Mac 开发 Heron ，但是我要 Release Linux 版本，因为服务器是 CentOS 平台。” 我们当时给出的答案便是**使用 Docker 方式编译**。

那么本小节内容就详细讲述了如何用 Docker 编译 Heron。Heron 开发组的同事们非常贴心的为大家写好了 Dockerfile，理论上 Mac 正常运行起 Docker 后，直接傻瓜式执行 `build-artifacts.sh` 就万事大吉了。这里有一个点要注意：请确保 Docker 的网络环境畅通，可能在下载 `bazel` 时遇到困难。

如果实在不行，这点上，我个人的经验是可以把 `bazel` 安装文件先下载到可访问的服务器上，然后修改 Dockerfile 的 `bazel` 下载路径，然后再执行编译。
