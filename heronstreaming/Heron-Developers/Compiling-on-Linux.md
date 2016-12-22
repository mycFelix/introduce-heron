# 在 Linux 平台上编译 Heron (Building on Linux Platforms)

Heron 现在支持在以下两个 Linux 平台上编译

* Ubuntu 14.04
* CentOS 7

## 在 Ubuntu 14.04 编译 (Building on Ubuntu 14.04)

在 Ubuntu 上，可以依次执行以下命令编译 Heron：

### 第一步 --- 更新 Ubuntu (Update Ubuntu)

```bash
$ sudo apt-get update -y
$ sudo apt-get upgrade -y
```

### 第二步 --- 安装依赖库 (Install required libraries)

```bash
$ sudo apt-get install git build-essential automake cmake libtool zip \
  libunwind-setjmp0-dev zlib1g-dev unzip pkg-config -y
```

### 第三步 --- 设置环境变量 (Set the following environment variables)

```bash
export CC=/usr/bin/gcc-4.8
export CCX=/usr/bin/g++-4.8
```

### 第四步 --- 安装 JDK 8 并设置 JAVA_HOME (Install JDK 8 and set JAVA_HOME)

```bash
$ sudo add-apt-repository ppa:webupd8team/java
$ sudo apt-get update -y
$ sudo apt-get install oracle-java8-installer -y
$ export JAVA_HOME="/usr/lib/jvm/java-8-oracle"
```

### 第五步 --- 安装 Bazel (Install Bazel)

***笔者注释：此处请确保您使用的网络环境可以正常安装访问示例命令中的链接***

```bash
wget -O /tmp/bazel.sh https://github.com/bazelbuild/bazel/releases/download/0.3.1/bazel-0.3.1-installer-linux-x86_64.sh
chmod +x /tmp/bazel.sh
/tmp/bazel.sh --user
```

请确认 Bazel 版本正确

### 第六步 --- 安装 Python 开发工具 (Install python development tools)

```bash
$ sudo apt-get install  python-dev python-pip
```

### 第七步 --- 设置 Bazel 环境变量 (Make sure the Bazel executable is in your `PATH`)

```bash
$ export PATH="$PATH:$HOME/bin"
```

### 第八步 --- 获取 Heron 最新源代码 (Fetch the latest version of Heron's source code)

```bash
$ git clone https://github.com/twitter/heron.git && cd heron
```

### 第九步 --- 设置 Heron 编译期环境 (Configure Heron for building with Bazel)

```bash
$ ./bazel_configure.py
```

### 第十步 --- 编译工程 (Build the project)

```bash
$ bazel build --config=ubuntu heron/...  
```

### 第十一步 --- 构建可执行安装包 (Build the packages)

***笔者注释：binpkgs 生成的是 shell 可执行文件; tarpkgs 生成的是 tar 安装文件。二者选其一执行即可***

```bash
$ bazel build --config=darwin scripts/packages:binpkgs
$ bazel build --config=darwin scripts/packages:tarpkgs
```

生成的脚本目录：`bazel-bin/scripts/packages/`

### 编译并安装 libtool (Compling and installing libtool)

```bash
$ wget http://ftpmirror.gnu.org/libtool/libtool-2.4.6.tar.gz
$ tar -xvf libtool-2.4.6.tar.gz
$ cd libtool-2.4.6
$ ./configure
$ make
$ sudo make install
```

### 编译并安装 libunwind (Compiling and installing libunwind)

```bash
$ wget http://download.savannah.gnu.org/releases/libunwind/libunwind-1.1.tar.gz
$ tar -xvf libunwind-1.1.tar.gz
$ cd libunwind-1.1
$ ./configure
$ make
$ sudo make install
```

### 编译并安装 gperftools (Compiling and installing gperftools)

```bash
$ wget https://github.com/gperftools/gperftools/releases/download/gperftools-2.5/gperftools-2.5.tar.gz
$ tar -xvf gperftools-2.5.tar.gz
$ cd gperftools-2.5
$ ./configure
$ make
$ sudo make install
```

## 在 CentOS 7 编译 (Building on CentOS 7)

在 CentOS 7 上,可以依次执行以下命令编译 Heron：

### 第一步 --- 安装依赖工具 (Install the required dependencies)

```bash
$ sudo yum install gcc gcc-c++ kernel-devel wget unzip zlib-devel zip git automake cmake patch libtool -y
```

### 第二步 --- 安装 libunwind (Install libunwind from source)

```bash
$ wget http://download.savannah.gnu.org/releases/libunwind/libunwind-1.1.tar.gz
$ tar xvf libunwind-1.1.tar.gz
$ cd libunwind-1.1
$ ./configure
$ make
$ sudo make install
```

### 第三步 --- 设置环境变量 (Set the following environment variables)

```bash
$ export CC=/usr/bin/gcc
$ export CCX=/usr/bin/g++
```

### 第四步 --- 安装 JDK 8 (Install JDK 8)

```bash
$ cd /opt/
$ sudo wget --no-cookies --no-check-certificate \
  --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" \
  "http://download.oracle.com/otn-pub/java/jdk/8u91-b14/jdk-8u91-linux-x64.tar.gz"
$ sudo tar xzf jdk-8u91-linux-x64.tar.gz
```

可以使用 `alternatives` 来设置 Java 版本

```bash
$ sudo cd /opt/jdk1.8.0_91/
$ sudo alternatives --install /usr/bin/java java /opt/jdk1.8.0_91/bin/java 2
$ sudo alternatives --config java
```

设置 `javac` 和 `jar` 命令

```bash
$ sudo alternatives --install /usr/bin/jar jar /opt/jdk1.8.0_91/bin/jar 2
$ sudo alternatives --install /usr/bin/javac javac /opt/jdk1.8.0_91/bin/javac 2
$ sudo alternatives --set jar /opt/jdk1.8.0_91/bin/jar
$ sudo alternatives --set javac /opt/jdk1.8.0_91/bin/javac
```

关联 Java 运行时环境

```bash
export JAVA_HOME=/opt/jdk1.8.0_91
export JRE_HOME=/opt/jdk1.8.0_91/jre
export PATH=$PATH:/opt/jdk1.8.0_91/bin:/opt/jdk1.8.0_91/jre/bin
```

### 第五步 --- 安装 Bazel (Install Bazel)

***笔者注释：此处请确保您使用的网络环境可以正常安装访问示例命令中的链接***

```bash
wget -O /tmp/bazel.sh https://github.com/bazelbuild/bazel/releases/download/0.3.1/bazel-0.3.1-installer-linux-x86_64.sh
chmod +x /tmp/bazel.sh
/tmp/bazel.sh --user
```

请确认 Bazel 版本正确

### 第六步 --- 获取 Heron 源码并编译 (Download Heron and compile it)

***笔者注释：此处请确保您使用的网络环境可以正常进行 git clone 操作***

```bash
$ git clone https://github.com/twitter/heron.git && cd heron
$ ./bazel_configure.py
$ bazel build --config=centos heron/...
```

### 第七步 --- 构建可执行安装包 (Build the packages)

***笔者注释：binpkgs 生成的是 shell 可执行文件; tarpkgs 生成的是 tar 安装文件。二者选其一执行即可***

```bash
$ bazel build --config=darwin scripts/packages:binpkgs
$ bazel build --config=darwin scripts/packages:tarpkgs
```

生成的脚本目录：`bazel-bin/scripts/packages/`。
