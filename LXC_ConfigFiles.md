- <a href="#DESCRIPTION ">概述</a>
- <a href="#CONFIGURATION ">配置事项</a>
- <a href="#ARCHITECTURE ">结构</a>
- <a href="#HOSTNAME ">主机名称</a>
- <a href="#HALT SIGNAL ">停止信号</a>
- <a href="#REBOOT SIGNAL ">重启信号</a>
- <a href="#INIT COMMAND ">初始化命令</a>
- <a href="#INIT WORKING DIRECTORY ">初始化工作路径</a>
- <a href="#PROC ">PROC</a>
- <a href="#EPHEMERAL ">EPHEMERAL</a>
- <a href="#NETWORK ">网络</a>
- <a href="# "></a>
- <a href="# "></a>
- <a href="# "></a>
- <a href="# "></a>
- <a href="# "></a>
- <a href="# "></a>
- <a href="# "></a>

<h2 id="DESCRIPTION">概述</h2>

LXC 支持非特权容器。非特权容器是指在没有任何特权的情况下运行的容器。这需要在运行容器的内核中支持 namespaces 。使用 namespaces 合并到主线内核后 LXC 第一时间支持非特权容器的运行。

本质上，用户 namespaces  隔离给定的 uid 和 gid 集。这是通过主机上一系列 uid 和 gid 与容器（非特权）中不同的uid 和 gid 之间建立映射来实现的。内核将以这样一种方式转换这个映射：在容器中，所有 uid 和 gid 都会出现在主机上，而在主机上，这些 uid 和 gid 实际上没有特权。例如，容器中以 UID 和 GID 为 0 的身份运行在主机上的进程可能显示为 UID 和 GID 为 100000。实际操作的细节可以从相应的 namespaces 手册页收集。UID 和 GID 映射可以用关键词 lxc.idmap 来定义。

Linux 容器是用一个简单的配置文件定义的。配置文件中的每个选项在一行中都有固定格式： key=value。 “#” 字符表示该行是注释。列表选项（如 capabilities 和 cgroups 选项）可以在没有值的情况下使用，以该字符清除该选项以前定义的任何值。

LXC namespaces 配置键使用单点。这意味着复杂的配置键，例如 lxc.net.0 展开各种子项：例如 lxc.net.0.type, lxc.net.0.link, lxc.net.0.ipv6.address ，以及其他更细参数的配置。


<h2 id="CONFIGURATION">配置事项</h2>

为了简化对多个相关容器的管理，可以使用一个容器配置文件来加载另一个文件。例如，网络配置可以在由多个容器包含的一个公共文件中定义。然后，如果将容器移动到另一个主机，则可能只需要更新一个文件。

```
lxc.include = /file path
```

指定要包含的文件。包含的文件必须采用相同的有效 lxc 配置文件格式。


<h2 id="ARCHITECTURE">结构</h2>

允许设置容器的体系结构。例如，为在 64 位主机上运行 32 位二进制文件的容器设置 32 位体系结构。这修复了容器脚本，这些脚本依赖于体系结构来完成一些工作，比如下载包。

```
lxc.arch = parameter
```

指定容器的体系结构，一些有效的选项：x86, i686, x86_64, amd64


<h2 id="HOSTNAME">主机名称</h2>

utsname 部分定义为容器所设置的主机名。这意味着容器可以设置自己的主机名，而无需更改系统中的主机名。这使得容器的主机名是私有的。

```
lxc.uts.name = parameter
```

<h2 id="HALT SIGNAL">停止信号</h2>

为了彻底关闭容器，允许指定信号名称或编号发送到容器的 init 进程。不同的 init 系统可以使用不同的信号来执行干净的关闭顺序。此选项允许以 kill（1）方式指定信号，例如 SIGPWR、SIGRTMIN+14、SIGRTMAX-10 或普通数字。默认信号是SIGPWR。

```
lxc.signal.halt = parameter
```

<h2 id="REBOOT SIGNAL">重启信号</h2>

允许指定信号名称或编号用作重新启动容器。此选项允许以kill（1）方式指定信号，例如 SIGTERM、SIGRTMIN+14、SIGRTMAX-10 或普通数字。默认信号是 SIGINT。

```
lxc.signal.reboot = parameter
```

<h2 id="STOP  SIGNAL">停止信号</h2>

允许指定信号名称或编号用作强制关闭容器。此选项允许以kill（1）方式指定信号，例如 SIGKILL、SIGRTMIN+14、SIGRTMAX-10 或普通数字。默认信号是 SIGKILL。

```
lxc.signal.stop = parameter
```

<h2 id="INIT COMMAND">初始化命令</h2>

设置此命令用作初始化容器的操作。

```
lxc.execute.cmd
```

这是从容器 rootfs 的绝对路径到二进制文件的默认执行方式，这与 lxc-execute 是相等的。

```
lxc.init.cmd = /sbin/init.
```

这是从容器 rootfs 的绝对路径到二进制文件的初始化方式。这与 lxc-start 是等同的。默认值是 /sbin/init.


<h2 id="INIT WORKING DIRECTORY">初始化工作路径</h2>

将容器内的绝对路径设置为容器的工作目录。在执行 init 之前，LXC 将切换到这个目录。

```
lxc.init.cwd 
```

容器中用作工作目录的绝对路径。


<h2 id="INIT ID">初始化ID</h2>

设置用于 init 系统和后续命令的 UID/GID。请注意，由于缺少权限，在引导系统容器时使用非根 UID 可能无法工作。在运行应用程序容器时，设置 UID/GID 非常有用。默认为：UID（0），GID（0）

```
lxc.init.uid = parameter
lxc.init.gid = parameter
```


<h2 id="PROC">PROC</h2>

配置容器 pro 文件系统。

```
lxc.proc.[proc file name]
```

指定要设置的 proc 文件名。可用的文件名是 /proc/PID/ 列出的文件名。例如：

```
lxc.proc.oom_score_adj = 10
```


<h2 id="EPHEMERAL">EPHEMERAL</h2>

允许指定容器是否在关闭时销毁。

```
lxc.ephemeral = parameter
```

唯一允许的值是 0 和 1。将此设置为 1 可在关闭时销毁容器。


<h2 id="NETWORK">网络</h2>

network 部分定义如何在容器中虚拟化网络。网络虚拟化作用于第二层。为了使用网络虚拟化，必须指定参数来定义容器的网络接口。即使系统只有一个物理网络接口，也可以在容器中分配和使用多个虚拟接口。

```
lxc.net
```

可在没有值的情况下使用，以清除所有以前的网络选项。

```
lxc.net.[i].type
```

指定要用于容器的网络虚拟化类型。


通过在所有键 lxc\.net.* 之后使用附加索引i可以指定多个网络。









<h2 id=""></h2>

<h2 id=""></h2>

<h2 id=""></h2>

<h2 id=""></h2>