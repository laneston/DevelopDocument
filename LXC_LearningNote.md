<a href="#Container overview">容器概述</a>
- <a href="#Container and virtual machine">容器与虚拟机</a>
- <a href="#Why use containers">为什么要用容器</a>
- <a href="#What is LXC">什么是LXC</a>

<a href="#Namespace">Namespace</a>
- <a href="#Types of namespace">Namespace的种类</a>
- <a href="#Use of namespace">Namespace的使用</a>
- <a href="#Notes on using namespace">Namespace使用注意事项</a>
- <a href="#Functions and features of namespace">Namespace的功能和特性</a>
- <a href="#features">features</a>
- <a href="#features">features</a>

# 容器概述

## 容器与虚拟机

容器是一种基于操作系统层级的虚拟化技术。“容器”是轻量级的虚拟化技术，与我们常用的 VMware 虚拟机不同，因为 LXC 不仿真硬件，且由于容器与主机共享相同的操作系统，共用相同的硬件资源，而虚拟机是寄生在宿主系统上的软件，与宿主系统或其它寄生系统抢占硬件的资源。

## 为什么要用容器

既然虚拟机与容器都能提供软件所需的执行环境，那容器的存在又有什么必要性呢？

容器是操作系统层级的虚拟化技术，与传统的硬件抽象层的虚拟化技术相比有以下优势：

1. 更小的虚拟化开销，虚拟机模拟硬件和操作系统，但是容器只模拟操作系统，因此更轻量级、速度更快。
2. 更快的部署。利用容器来隔离特定应用，只需安装容器，即可使用容器相关命令来创建并启动容器来为应用提供虚拟执行环境。传统的虚拟化技术则需要先创建虚拟机，然后安装系统，再部署应用。

## 什么是LXC

LXC 是 Linux Containers 的简称，LXC 允许你在宿主操作系统内的容器运行应用。容器在网络、行为等方面都与宿主OS都隔离。LXC 的仿真（模拟）是通过 Linux 内核的 cgroups 和 namespaces 来实现的，因此 LXC 只能模拟基于 Linux 类的操作系统。

# Namespace

namespace 是 Linux 内核用来隔离内核资源的方式。通过 namespace 可以让一些进程只能看到与自己相关的一部分资源，而另外一些进程也只能看到与它们自己相关的资源，这两拨进程根本就感觉不到对方的存在。具体的实现方式是把一个或多个进程的相关资源指定在同一个 namespace 中。Linux namespaces 是对全局系统资源的一种封装隔离，使得处于不同 namespace 的进程拥有独立的全局系统资源，改变一个 namespace 中的系统资源只会影响当前 namespace 里的进程，对其他 namespace 中的进程没有影响。

## Namespace的种类

目前Linux中提供了六类系统资源的隔离机制，分别是：

- Mount: 隔离文件系统挂载点；
- UTS:   隔离主机名和域名信息；
- IPC:   隔离进程间通信；
- PID:   隔离进程的ID；
- Network: 隔离网络资源；
- User:  隔离用户和用户组的ID。

## Namespace的使用

涉及到 Namespace 的操作口包括 clone(), setns(), unshare() 以及还有 /proc 下的部分文件。为了使用特定的 Namespace，在使用这些接口的时候需要指定以下一个或者多个参数：

- CLONE_NEWNS: 用于指定Mount Namespace
- CLONE_NEWUTS: 用于指定UTS Namespace
- CLONE_NEWIPC: 用于指定IPC Namespace
- CLONE_NEWPID: 用于指定PID Namespace
- CLONE_NEWNET: 用于指定Network Namespace
- CLONE_NEWUSER: 用于指定User Namespace

**clone函数**

可以通过 clone 系统调用来创建一个独立 Namespace 的进程，它的函数描述如下：

```
int clone(int (*child_func)(void *), void *child_stack, int flags, void *arg);
```

它通过 flags 参数来控制创建进程时的特性，比如新创建的进程是否与父进程共享虚拟内存等。比如可以传入 CLONE_NEWNS 标志使得新创建的进程拥有独立的 Mount Namespace，也可以传入多个flags使得新创建的进程拥有多种特性，比如：

```
flags = CLONE_NEWNS | CLONE_NEWUTS | CLONE_NEWIPC;
```

传入这个 flags 那么新创建的进程将同时拥有独立的 Mount Namespace、UTS Namespace 和 IPC Namespace。

**通过/proc文件查看已存在的Namespace**

输入 ps -A 即可查看所有程序的 PID

如果需要查看 PID 为 1988 的进程的文件信息，则只需输入以下 ls 命令即可查看对应的信息。

```
sudo ls -al /proc/1988/ns/

lrwxrwxrwx 1 lance lance 0 11月  3 16:39 cgroup -> 'cgroup:[4026531835]'
lrwxrwxrwx 1 lance lance 0 11月  3 16:39 ipc -> 'ipc:[4026531839]'
lrwxrwxrwx 1 lance lance 0 11月  3 16:39 mnt -> 'mnt:[4026531840]'
lrwxrwxrwx 1 lance lance 0 11月  3 16:39 net -> 'net:[4026532000]'
lrwxrwxrwx 1 lance lance 0 11月  3 16:39 pid -> 'pid:[4026531836]'
lrwxrwxrwx 1 lance lance 0 11月  3 16:39 pid_for_children -> 'pid:[4026531836]'
lrwxrwxrwx 1 lance lance 0 11月  3 16:39 user -> 'user:[4026531837]'
lrwxrwxrwx 1 lance lance 0 11月  3 16:39 uts -> 'uts:[4026531838]'
```

**setns函数**

setns() 函数可以把进程加入到指定的 Namespace 中，它的函数描述如下：

```
int setns(int fd, int nstype);
```

它的参数描述如下：

- fd：表示文件描述符，前面提到可以通过打开 /proc/$pid/ns/ 的方式将指定的 Namespace 保留下来，也就是说可以通过文件描述符的方式来索引到某个 Namespace。
- nstype：用来检查 fd 关联 Namespace 是否与 nstype 表明的 Namespace 一致，如果填 0 的话表示不进行该项检查。

**unshare函数**

unshare() 系统调用函数用于将当前进程和所在的 Namespace 分离，并加入到一个新的 Namespace 中，相对于 setns() 系统调用来说，unshare() 不用关联之前存在的 Namespace，只需要指定需要分离的 Namespace 就行，该调用会自动创建一个新的 Namespace。

unshare()的函数描述如下：

```
int unshare(int flags);
```

其中 flags 用于指明要分离的资源类别，它支持的 flag s与 clone 系统调用支持的 flags 类似，这里简要的叙述一下几种标志：

- CLONE_FILES: 子进程一般会共享父进程的文件描述符，如果子进程不想共享父进程的文件描述符了，可以通过这个flag来取消共享;
- CLONE_FS: 使当前进程不再与其他进程共享文件系统信息;
- CLONE_SYSVSEM: 取消与其他进程共享SYS V信号量;
- CLONE_NEWIPC: 创建新的IPC Namespace，并将该进程加入进来。

## Namespace使用注意事项

unshare() 和 setns() 系统调用对 PID Namespace 的处理不太相同，当 unshare PID namespace 时，调用进程会为它的子进程分配一个新的 PID Namespace，但是调用进程本身不会被移到新的 Namespace 中。而且调用进程第一个创建的子进程在新 Namespace 中的PID 为1，并成为新 Namespace 中的 init 进程。

setns()系统调用也是类似的，调用者进程并不会进入新的 PID Namespace，而是随后创建的子进程会进入。

为什么创建其他的 Namespace 时 unshare() 和 setns() 会直接进入新的 Namespace，而唯独 PID Namespace 不是如此呢？

因为调用 getpid() 函数得到的 PID 是根据调用者所在的 PID Namespace 而决定返回哪个 PID，进入新的 PID namespace 会导致 PID 产生变化。而对用户态的程序和库函数来说，他们都认为进程的 PID 是一个常量，PID 的变化会引起这些进程奔溃。

换句话说，一旦程序进程创建以后，那么它的 PID namespace 的关系就确定下来了，进程不会变更他们对应的 PID namespace。

## Namespace的功能和特性

**Mount Namespace**

Mount Namespace 用来隔离文件系统的挂载点，不同 Mount Namespace 的进程拥有不同的挂载点，同时也拥有了不同的文件系统视图。Mount Namespace 是历史上第一个支持的 Namespace，它通过 CLONE_NEWNS 来标识的。

**挂载的概念**

在Windows下，mount 挂载，就是给磁盘分区提供一个盘符。比如插入U盘后系统自动分配给了它 I:盘符 。这其实就是挂载，退U盘的时候进行安全弹出，其实就是卸载 unmount。

mount 所达到的效果是：像访问一个普通的文件一样访问位于其他设备上文件系统的根目录，也就是将该设备上目录的根节点挂到了另外一个文件系统的页节点上，达到给这个文件系统扩充容量的目的。

可以通过/proc文件系统查看一个进程的挂载信息，具体做法如下：

```
cat /proc/$pid/mountinfo
```

输出格式如下：

|36|35|98:0|/mnt1|/mnt2|rw,noatime|master:1|-|ext3|/dev/root|rw,error=continue|
|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
|1|2|3|4|5|6|7|8|9|10|11|


1. mount ID: 对于 mount 操作一个唯一的ID；
2. parent ID: 父挂载的 mount ID 或者本身的mount ID(本身是挂载树的顶点)；
3. major:minor: 文件关联额设备的主次设备号；
4. root: 文件系统的路径名，这个路径名是挂载点的根；
5. mount point: 挂载点的文件路径名(相对于这个进程的跟目录)；
6. mount options: 挂载选项
7. optional fields: 可选项，格式 tag:value；
8. separator: 分隔符，可选字段由这个单个字符标示结束的；
9. filesystem type: 文件系统类型 type[.subtype]；
10. mount source: 文件系统相关的信息，或者none；
11. super options: 一些高级选项(文件系统超级块的选项)。