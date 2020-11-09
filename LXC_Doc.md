
虚拟系统由早期完全透过软件来模拟硬件的全虚拟化（Full virtualization），演进到以修改系统（Guest OS）核心的方式，来简化 CPU 与外围设备操作指令转译的复杂度，以提升虚拟系统效能的半虚拟化（Paravirtualization），一直到目前 Linux 核心支持的原生虚拟化技术（Linux Kernel-base Virtual Machine，简称Linux KVM）。

即使如 Linux KVM 利用支持虚拟化技术的实体 CPU 与系统核心，强化了 CPU 指令运算的效能，但大部分的周边设备，如网络卡等，却还是要倚赖软件来（例如QEMU）将其虚拟化之后使用，在实际的应用上也会明显拖累整体系统的效能，尤其是网卡更是明显，所有的虚拟网卡都是以同一张实体网卡来当做对外联系的窗口（如果所有的虚拟网络卡都有对外联机的需求时），这势必会影响原先实体系统的网络运作，虚拟计算机的数量越多，影响的程度就越大。


LXC附带了一个稳定的 C API 和一系列绑定。我们可以在 LXC 发行版中添加 liblxc1 API，但不会在没有调用 liblxc2 的情况下删除或更改现有符号。随稳定 API 一起发布的第一个 LXC 版本是lxc1.0.0。只有文件 lxccontainer.h 中列出的符号才是 API 的一部分，其他所有符号都是 LXC 内部的，可以随时更改。

如上所述，文件 lxccontainer.h 是我们的公共C API。API 使用的一些最好的例子是绑定和  LXC 工具本身。我们也有一个最新的 API 文档，用于当前的 git 主机。下面是一个如何使用 API 创建、启动、停止和销毁容器的简单示例：

```
#include <stdio.h>

#include <lxc/lxccontainer.h>

int main() {
    struct lxc_container *c;
    int ret = 1;

    /* Setup container struct */
    c = lxc_container_new("apicontainer", NULL);
    if (!c) {
        fprintf(stderr, "Failed to setup lxc_container struct\n");
        goto out;
    }

    if (c->is_defined(c)) {
        fprintf(stderr, "Container already exists\n");
        goto out;
    }

    /* Create the container */
    if (!c->createl(c, "download", NULL, NULL, LXC_CREATE_QUIET,
                    "-d", "ubuntu", "-r", "trusty", "-a", "i386", NULL)) {
        fprintf(stderr, "Failed to create container rootfs\n");
        goto out;
    }

    /* Start the container */
    if (!c->start(c, 0, NULL)) {
        fprintf(stderr, "Failed to start the container\n");
        goto out;
    }

    /* Query some information */
    printf("Container state: %s\n", c->state(c));
    printf("Container PID: %d\n", c->init_pid(c));

    /* Stop the container */
    if (!c->shutdown(c, 30)) {
        printf("Failed to cleanly shutdown the container, forcing.\n");
        if (!c->stop(c)) {
            fprintf(stderr, "Failed to kill the container.\n");
            goto out;
        }
    }

    /* Destroy the container */
    if (!c->destroy(c)) {
        fprintf(stderr, "Failed to destroy the container.\n");
        goto out;
    }

    ret = 0;
out:
    lxc_container_put(c);
    return ret;
}
```