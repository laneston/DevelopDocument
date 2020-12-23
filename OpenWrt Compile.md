# 编译环境

- 硬件平台：MT7622
- 主机环境：Ubuntu18 Server
- openwrt 版本：18.06.9

## 软件安装

在正式编译前，我们需要在 Ubuntu 上安装以下工具，以保证编译能够正常执行：

```
/*获取服务器端更新的清单*/
sudo apt-get update

/*安装 git (非必要)*/
sudo apt-get install git-core

/*安装 g++ 编译工具*/
sudo apt-get install g++

/*GNU C Library: Development Libraries and Header Files (6.0+20160213-1ubuntu1)*/
sudo apt-get install libncurses5-dev

/*GNU C Library: Development Libraries and Header Files (1:1.2.8.dfsg-2ubuntu4.3)*/
sudo apt-get install zlib1g-dev

/*YACC-compatible parser generator*/
sudo apt-get install bison

/*fast lexical analyzer generator*/
sudo apt-get install flex

/*解压工具*/
sudo apt-get install unzip

/*automatic configure script builder*/
sudo apt-get install autoconf

/*GNU awk, a pattern scanning and processing language*/
sudo apt-get install gawk

sudo apt-get install libssl-dev

/*makefile 脚本执行工具*/
sudo apt-get install make

/*GNU Internationalization utilities*/
sudo apt-get install gettext

sudo apt-get install gcc

/*GNU assembler, linker and binary utilities*/
sudo apt-get install binutils

/*Apply a diff file to an original*/
sudo apt-get install patch

/*high-quality block-sorting file compressor - utilities*/
sudo apt-get install bzip2

/*compression library - development*/
sudo apt-get install libz-dev

/* Highly configurable text format for writing documentation [universe]*/
sudo apt-get install asciidoc

/*Advanced version control system*/
sudo apt-get install subversion

/*Informational list of build-essential packages*/
sudo apt-get install build-essential

/*easy-to-use, scalable distributed version control system [universe]*/
sudo apt-get install mercurial
```

## 下载源码

在 GitHub 上找到 <a href="https://github.com/openwrt/openwrt"> openwrt </a> 源码，选择自己需要的版本进行下载。

# 开始编译

## 更新软件包

```
sudo ./scripts/feeds update -a
sudo ./scripts/feeds install -a
```

这一步不是必须的，但是做这一步处理可以避免出现一些错误。

## 进入定制界面

```
sudo make menuconfig
```

### 定制界面选项


## 问题1

```
WARNING: Makefile 'package/utils/busybox/Makefile' has a dependency on 'libpam', which does not exist
WARNING: Makefile 'package/utils/busybox/Makefile' has a dependency on 'libpam', which does not exist
WARNING: Makefile 'package/utils/busybox/Makefile' has a build dependency on 'libpam', which does not exist
WARNING: Makefile 'package/boot/kexec-tools/Makefile' has a dependency on 'liblzma', which does not exist
WARNING: Makefile 'package/network/services/lldpd/Makefile' has a dependency on 'libnetsnmp', which does not exist
WARNING: Makefile 'package/utils/policycoreutils/Makefile' has a dependency on 'libpam', which does not exist
WARNING: Makefile 'package/utils/policycoreutils/Makefile' has a dependency on 'libpam', which does not exist
WARNING: Makefile 'package/utils/policycoreutils/Makefile' has a build dependency on 'libpam', which does not exist
 ```

这个警告提示语表示 libpam 这个库没有存在，需在 <a href="https://github.com/openwrt/packages/tree/master/libs"> packages </a> 中找到 libpam 文件夹，然后将文件夹放入本地编译路径 /package/libs 中。

值得注意的是，有些库在 packages 中并非如警告提示中显示的名称，如果找不到，可以在 google 搜索中输入关键词 "openwrt liblzma" 。

<img src="https://github.com/laneston/Pictures/blob/master/Post-OpenWrt/libs-view.jpg" width="50%" height="50%">

打开搜索到的 openwrt 网站页面，就能看到如上图所示的画面。点击最底端的 source code 选项，页面就会跳转到 github 的 packages 源码页面，所显示的部分就是对应的库的路径了。将 packages 中对应路径的部分移动到本地路径，并修改成所需的库的名字。如上图中的例子，在 packages 中找到的路径为：packages/utils/xz

所以接下来把 xz 文件夹移动至本地端的 /package/libs 中即可。

## 问题2

```
make[3]: Leaving directory '/home/openwrt/tools/cmake'
time: tools/cmake/compile#113.44#11.93#612.80
tools/Makefile:152: recipe for target 'tools/cmake/compile' failed
make[2]: *** [tools/cmake/compile] Error 2
make[2]: Leaving directory '/home/openwrt'
tools/Makefile:150: recipe for target '/home/openwrt/staging_dir/target-aarch64_cortex-a53_musl/stamp/.tools_compile_yynyyyyynyyyyynyynnyyyynyyyyyyyyyyyyyyynyynynnyyynnyy' failed
make[1]: *** [/home/openwrt/staging_dir/target-aarch64_cortex-a53_musl/stamp/.tools_compile_yynyyyyynyyyyynyynnyyyynyyyyyyyyyyyyyyynyynynnyyynnyy] Error 2
make[1]: Leaving directory '/home/openwrt'
/home/openwrt/include/toplevel.mk:216: recipe for target 'world' failed
make: *** [world] Error 2
```

