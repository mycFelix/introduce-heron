# 在 Mac 平台上编译 Heron (Building on Mac OS X)

可以在 Mac (versions 10.10 and 10.11) 平台上，依次执行如下命令，进行 Heron 的编译。

### 第一步 --- 安装 Homebrew (Install Homebrew)

***笔者注释：Homebrew 是 Mac 平台上非常棒的包管理工具***

如果你还没有安装 [Homebrew](http://brew.sh/)，你可以通过以下命令进行一键式安装：

```bash
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

### 第二步 --- 安装其他所需环境 (Install other required libraries)

```bash
brew install automake
brew install cmake
brew install libtool
```

### 第三步 --- 设置环境变量 (Set the following environment variables)

```bash
$ export CC=/usr/bin/clang
$ export CXX=/usr/bin/clang++
$ echo $CC $CXX
```

### 第四步 --- 安装 Bazel (Install Bazel)

***笔者注释：此处请确保您使用的网络环境可以正常安装访问示例命令中的链接***

```bash
wget -O /tmp/bazel.sh  https://github.com/bazelbuild/bazel/releases/download/0.3.1/bazel-0.3.1-installer-darwin-x86_64.sh
chmod +x /tmp/bazel.sh
/tmp/bazel.sh --user
```

### 第五步 --- 设置 Bazel 环境变量 (Make sure the Bazel executable is on your `PATH`)

```bash
$ export PATH="$PATH:$HOME/bin"
```

### 第六步 --- 获取 Heron 最新源代码 (Fetch the latest version of Heron's source code)

***笔者注释：此处请确保您使用的网络环境可以正常进行 git clone 操作***

```bash
$ git clone https://github.com/twitter/heron.git && cd heron
```

### 第七步 --- 配置 Heron 编译时环境信息 (Configure Heron for building with Bazel)

```bash
$ ./bazel_configure.py
```

如果检查报告显示缺失组件，可以使用 Homebrew 进行对应安装。

### 第八步 --- 编译工程 (Build the project)

```bash
$ bazel build --config=darwin heron/...
```

### 第九步 --- 构建可执行安装包 (Build the packages)

***笔者注释：binpkgs 生成的是 shell 可执行文件; tarpkgs 生成的是 tar 安装文件。二者选其一执行即可***

```bash
$ bazel build --config=darwin scripts/packages:binpkgs
$ bazel build --config=darwin scripts/packages:tarpkgs
```

生成的脚本目录：`bazel-bin/scripts/packages/`。
