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
- nstype：用来检查fd关联Namespace是否与nstype表明的Namespace一致，如果填0的话表示不进行该项检查。