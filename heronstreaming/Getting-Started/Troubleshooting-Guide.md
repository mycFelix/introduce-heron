# [问题定位简易指南](http://twitter.github.io/heron/docs/getting-started-troubleshooting/)

本文档旨在列举一些部署拓扑时可能会出现的常见问题的常规解决方案。

### 1. 如何能获得更多的调试信息?

在拓扑提交命令时，开启 `--verbose` 选项。

```bash
heron submit ... ExclamationTopology --verbose        
```

### 2. 为什么拓扑启动成功，但没有正常运行呢？

有时候，即便是拓扑提交成功，某些组件也会出现一些问题，导致拓扑运行失败。比如，TMaster 可能会因为依赖不全而导致启动异常。

例如，可能会出现如下信息：

```bash
$ heron activate local ExclamationTopology

...

[2016-05-27 12:02:38 -0600] com.twitter.heron.common.basics.FileUtils SEVERE: \
Failed to read from file.
java.nio.file.NoSuchFileException: \
/home//.herondata/repository/state/local/pplans/ExclamationTopology

...

[2016-05-27 12:02:38 -0600] com.twitter.heron.spi.utils.TMasterUtils SEVERE: \
Failed to get physical plan for topology ExclamationTopology

...

ERROR: Failed to activate topology 'ExclamationTopology'
INFO: Elapsed time: 1.883s.
```

#### 要怎么做呢？

* 如下日志文件会记录任何组件启动异常信息

    ```bash
    ~/.herondata/topologies/{cluster}/{role}/{TopologyName}/heron-executor.stdout
    ```

    例如，当 Stream Manager 开始处理相关信息时，在上述文件中可能会得到如下错误日志：

    ```
    Running stmgr-1 process as ./heron-core/bin/heron-stmgr ExclamationTopology \
    ExclamationTopology0a9c6550-7f3d-44fb-97ea-5c779fac6924 ExclamationTopology.defn LOCALMODE \
    /Users/${USERNAME}/.herondata/repository/state/local stmgr-1 \
    container_1_word_2,container_1_exclaim1_1 58106 58110 58109 ./heron-conf/heron_internals.yaml
    2016-06-09 16:20:28:  stdout:
    2016-06-09 16:20:28:  stderr: error while loading shared libraries: libunwind.so.8: \
    cannot open shared object file: No such file or directory
    ```

    那么，着手去修复指定的问题即可。

* 还有可能会出现解析 `localhost` 异常。可以根据如下命令，做相关检查

        ```bash
        $ python -c "import socket; print socket.gethostbyname(socket.gethostname())"
        Traceback (most recent call last):
          File "<string>", line 1, in <module>
        socket.gaierror: [Errno 8] nodename nor servname provided, or not known
        ```

        如果输出结果是一个看起来正常的 IP 地址，如 `127.0.0.1` ，那么您不会遇到上述问题。

        如果输出结果同如上代码打印信息一样，那么您可能需要修改 `/etc/hosts` 文件来解决这个问题，您可以参考如下命令：

        1. 运行如下命令输出您本地机的 `hostname`

            ```bash
            $ python -c "import socket; print socket.gethostname()"
            ```

        2. 用超级用户权限打开 `/etc/hosts` 文件并找到如下这一行：

            ```bash
            127.0.0.1	localhost
            ```

        3. 将您的 `hostname` 追加直 "localhost" 字样的后面，以 `hostname=tw-heron`

            ```bash
            127.0.0.1   localhost   tw-heron
            ```

        4. 保存文件。修改通常会即可生效。是否重启，可依您所用的平台具体而定。

### 3. 为什么进程会在运行期报错？

        如果某个组件(如 TMaster 或 Stream Manager) 在运行期报错或因异常而终止，您可以进入如下目录，查看相关日志。

        ```bash
        ~/.herondata/topologies/{cluster}/{role}/{TopologyName}/log-files/
        ```

### 4. 如何强制杀死(kill)并清理(clean up)拓扑？

        通常可以使用如下命令终止拓扑运行:

        ```bash
        heron kill ...
        ```

        如果命令返回错误，拓扑可能依旧处于未杀死状态，此时可以运行 `kill pid` 来杀死所有相关进程，并执行 `rm -rf ~/.herondata/` 命令清理拓扑状态。

---
***笔者后记***

拓扑提交正常，而在运行时报错是很常见的，也是很让人心烦的。我们此时往往很难梳理思路去定位问题。那么，这个时候日志就是最好的帮手。Heron 为拓扑记录的日志非常全面，不论调度器是 `local` 模式还是 `YARN` 模式，任何运行时的异常，都能在日志中找到答案。

不论是状态器在 `local` 模式下的 `rm -rf ~/.herondata/` 命令或者是 `ZK` 模式下的 `rmr /heron` 命令，都是非常危险的操作，因为它会删掉当前所有拓扑的状态记录，请谨慎操作。如果可以确定拓扑名称，那么最好还是找到相关目录，依次删除，或者写个脚本也是比较好的选择。
