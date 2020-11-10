
硬件平台：MT7622

# 编译准备

## 软件安装

在 Ubuntu 上安装以下工具：

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

## 下载固件

在 <a href = "https://openwrt.org/downloads"> OpenWrt </a> 官网上下载你想要的版本的固件。

<img src="https://github.com/laneston/Pictures/blob/master/Post-OpenWrt/OpenWrt%20kernel.jpg" width="50%" height="50%">

如上图所示，我们选取平台所对应的固件： openwrt-sdk-19.07.4-mediatek-mt7622_gcc-7.5.0_musl.Linux-x86_64.tar

# 开始编译



## 生成配置文件

```
sudo make defconfig 
```

## 进入定制界面

```
sudo make menuconfig
```

### 定制界面选项

 