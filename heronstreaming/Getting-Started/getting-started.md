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

```bash
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

```bash
$ export PATH=$PATH:~/bin
```

现在，我们可以继续安装 Heron tools，具体执行命令可以参考

```bash
$ chmod +x heron-tools-install-VERSION-PLATFORM.sh
$ ./heron-tools-install-VERSION-PLATFORM.sh --user
Heron tools installer
---------------------

Uncompressing......
Heron Tools is now installed!
...
```

这时，我们可以执行以下命令来验证 Heron 是否安装成功：

```bash
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

***笔者注释：Heron 的开发组为我们提供了丰富的样例拓扑，以便我们学习 Heron 的运行机制。同时运行样例拓扑也是一个不错的学习 Heron CLI 命令的过程***





