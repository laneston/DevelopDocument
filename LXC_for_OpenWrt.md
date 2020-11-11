这篇文章记录如何在 OpenWrt 镜像上搭建 LXC ，并讲述如何进行相关配置等操作。

## OPKG 软件包管理

opkg 工具 (一个 ipkg 变种) 是一个用来从本地软件仓库或互联网软件仓库上下载并安装 OpenWrt 软件包的轻量型软件包管理器。

GNU/Linux 用户可能会对 apt-get，aptitude，pacman，yum 等比较熟悉，也会看出其相似之处。它与 NSLU2 上同样用于嵌入式设备的 Optware 也有相似之处。OPKG 没有仅仅将软件安装到一个单独的路径（如：/opt），而是根文件系统上的一个完整的包管理器。它也包含了增加内核模块与驱动的可能性。OPKG 有时被称为 Entware ，但这主要是针对为嵌入式设备准备的 Entware 仓库。

opkg 试图在软件包仓库内来解决依赖关系。如果失败了，它将会报告一个错误并停止安装该软件包。

如果丢失第三方包的依赖关系，源码包依然可用的话，为了忽略依赖关系的错误可以使用 –force-depends 选项。

请注意: 如果你在使用一个 snapshot 、trunk 或 bleeding edge 版本，在仓库中的软件包适用内核版本比你的 Flash 上的内核更高，opkg install <pkg> 可能会失败。这种情况下，会报错『Cannot satisfy the following dependencies for…』。


