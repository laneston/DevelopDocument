# 编译环境配置

- 硬件平台：MT7622
- 主机环境：Ubuntu20(windows 子系统)
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


# 编译过程

在编译前，最好将编译文件夹读写权限设置成最低：

```
sudo chmod 777 -R openwrt-18.06.9
```

## 解压源码包

```
sudo unzip openwrt-18.06.9.zip
```

## 更新软件包

```
sudo ./scripts/feeds update -a
sudo ./scripts/feeds install -a
```

## 进入定制界面

```
make menuconfig
```

### 下载所需工具包

```
sudo make download
```
### 开始编译


```
sudo make -j1 V=s
```

为了增加成功率，编译所设置的线程数最好为1： -j1

如果能够链接 VPN，编译过程中最好全程链接 VPN


# 编译问题解决方案

