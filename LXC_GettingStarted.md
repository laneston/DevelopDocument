本篇文章记录 LXC 的初步安装与使用的相关内容，目的是为了使读者能够根据本篇内容实现单次安装成功，并能够进行基础的操作内容。

运行平台：
- ubuntu 20.04

## 依赖要求

要使 LXC 成功运行，则需要一个 C 库来支持，支持的 C 库包括: glibc, musl libc, uclib, bionic 

除此之外，还要保证内核的版本大于或等于 2.6.32

查看内核指令：

```
cat /proc/version
```

如果使用 lxc-attach 内核版本要大于等于3.8

如果要使用 unprivileged containers 则：

- 为了 unprivileged CGroups 操作使用 libpam-cgfs 配置你的系统；
- 最新版本的新 uidmap 和新版本的 gidamap；
- Linux 内核大于等于 3.12

推荐的库：

- libcap (to allow for capability drops)
- libapparmor (to set a different apparmor profile for the container)
- libselinux (to set a different selinux context for the container)
- libseccomp (to set a seccomp policy for the container)
- libgnutls (for various checksumming)
- liblua (for the LUA binding)
- python3-dev (for the python3 binding)

## glibc

### 什么是glibc

glibc 的全称是 GUN C Library，这个库为 GUN 系统和 GUN/Linux 系统，以及许多其他使用Linux作为内核的系统提供核心库文件。这些库提供了关键的api，包括isoc11、POSIX.1-2008、BSD、特定于操作系统的api等等。这些api包括诸如open、read、write、malloc、printf、getaddrinfo、dlopen、pthread_create、crypt、login、exit等基础设施。

### 查看glibc

一般情况下，ubuntu 桌面版会自带 glibc 文件。如果要查看版本信息，只需在命令窗口输入：

```
ldd --version
```

## 安装 LXC

使用 ubuntu 10.04 LTS 以上版本，输入以下命令：

```
sudo apt-get install lxc
```

你的系统将有所有的LXC命令可用，它的所有模板以及python3绑定都需要编写LXC脚本。

## 作为用户创建一个非特殊容器

非特殊容器是最安全的容器，它们使用 uid 和 gid 的映射为容器分配一系列 uid 和 gid

**注：**

**用户标识号(UID):** 是一个整数，系统内部用它来标识用户。一般情况下它与用户名是一一对应的。如果几个用户名对应的用户标识号是一样的，系统内部将把它们视为同一个用户，但是它们可以有不同的口令、不同的主目录以及不同的登录Shell等。取值范围是0-65535。0是超级用户root的标识号，1-99由系统保留，作为管理账号，普通用户的标识号从100开始。在Linux系统中，这个界限是500。

**组标识号(GID):** 字段记录的是用户所属的用户组。它对应着/etc/group文件中的一条记录。

这意味着容器中的 uid 0(root) 实质是容器外部的 uid x 类似。因此如果出现严重错误，攻击者设法逃离容器，它们将会发现自己拥有的权限与 nobody 用户一样少。

然而，这也意味着不允许以下常见操作：

1. 大多数文件系统的挂载；
2. 创建设备节点；
3. 针对映射集之外的 uid/gid 的任何操作。

因此，大多数分发模板根本无法使用这些模板。相反，你应该使用“下载”模板，该模板将为您提供已知在这种环境中工作的发行版的预构建映像。

下面说明的前提都是使用最新的 Ubuntu 系统或提供类似体验的替代 Linux 发行版，即最新的内核和 <a href="#shadow">shadow</a> 的最新版本，以及 <a href="#libpam-cgfs">libpam-cgfs</a> 和默认 uid/gid 分配。

首先，需要确保用户在 /etc/subuid 和 /etc/subgid 中定义了 uid 和 gid 映射。在 Ubuntu 系统上，默认分配给系统中的每个新用户 65536 个 uid 和 gid，所以您应该已经有了一个。如果没有，你可以使用用户模式给自己新建一个。

接下来是 /etc/lxc/lxc-usernet，用于为非特权用户设置网络设备配额。默认情况下，不允许用户在主机上创建任何网络设备，若要更改此设置，请添加：

```
your-username veth lxcbr0 10
```

<h3 id="shadow">shadow解析</h3>

shadow 是 passwd 的影子文件，与/etc/passwd文件不同，/etc/shadow文件是只有系统管理员才有权利进行查看和修改的文件。

shadow 是一个文件，它包含系统账户的密码信息和可选的年龄信息。如果没有维护好密码安全，此文件绝对不能让普通用户可读。此文件的每行包括 9 个字段，使用半角冒号 (“:”) 分隔，顺序如下：

```
       登录名
           必须是有效的账户名，且已经存在于系统中。

       加密了的密码
           如果密码字段包含一些不是合法结果的字符，比如 ! 或 *，用户将无法使用 unix
           密码登录(但是可以通过其它方法登录系统)。

           此字段可以为空，此时认证为特定的登录名时，不要求密码。然而，一些读取 /etc/shadow
           文件的应用程序，在密码字段为空时，可能决定禁止任何访问。

           A password field which starts with an exclamation mark means that the password is
           locked. The remaining characters on the line represent the password field before the
           password was locked.

       最后一次更改密码的日期
           最近一次更改密码的时间，表示从1970年1月1日开始的天数。

           The value 0 has a special meaning, which is that the user should change her password
           the next time she will log in the system.

           空字段表示密码年龄功能被禁用。

       密码的最小年龄
           最小密码年龄是指，用户一次更改密码之后，要等多长时间才再次被允许更改密码。

           空字段或 0 表示没有最小密码年龄。

       最大密码年龄
           最大密码年龄是指，这写天之后，用户必须更改密码。

           这写天之后，密码仍然可用。用户将会在下次登录的时候被要求更改密码。

           空字段表示没有最大密码年龄，没有密码警告时间段，没有密码禁用时间段(请看下边)。

           如果最大密码年龄小于最小密码年龄，用户将会不能更改密码。

       密码警告时间段
           密码过期之前，提前警告用户的的天数(请参考上边的密码的最大年龄)。

           空字段或者 0 表示没有密码警告期。

       密码禁用期
           密码过期(查看上边的密码最大年龄)后，仍然接受此密码的天数(在此期间，用户应该在下次登录时修改密码)。

           密码到期并且过了这个宽限期之后，使用用户的当前的密码将会不能登录。用户需要联系系统管理员。

           空字段表示没有强制密码过期。

       账户过期日期
           账户过期的日期，表示从1970年1月1日开始的天数。

           Note that an account expiration differs from a password expiration. In case of an
           account expiration, the user shall not be allowed to login. In case of a password
           expiration, the user is not allowed to login using her password.

           空字段表示账户永不过期。

           应该避免使用 0，因为它既能理解成永不过期也能理解成在1970年1月1日过期。

       保留字段
           此字段保留作将来使用。
```

<h3 id="libpam-cgfs">libpam-cgfs</h3>
