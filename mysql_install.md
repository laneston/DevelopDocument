# mysql交叉编译与安装

- 主机编译平台：ubuntu20.04 (Windows 子系统)
- 交叉编译链：aarch64-none-linux-gnu-
- mysql版本：mysql-5.7.32
- mysql运行环境：openwrt

mysql交叉编译的主要流程是：主机编译 mysql，交叉编译 boost 库，交叉编译 ncurse 库，交叉编译 openssl 库，最后交叉编译 mysql 库。值得注意的是，mysql 对依赖库的版本有特殊指定，请尽量按照本说明对应的版本进行操作，如果依赖库没有达到 mysql 指定的版本新度，会出现配置不通过。

## 主机编译mysql

之所以第一步要主机编译 mysql，第一是为了熟悉一遍编译的流程，但最重要的是，我们之后交叉编译需要用到主机编译生成的脚本。

编译过程在此说明。

解压 mysql-5.7.32 压缩包，进入文件夹第一层目录，输入以下命令：

```
cmake . -DDOWNLOAD_BOOST=1 -DWITH_BOOST=/home/Mysql_Complie/mysql-8.0.22/boost -DCMAKE_INSTALL_PREFIX=/home/Mysql_Complie/mysql-8.0.22/__install
```

这个命令是配置命令，目的是生成编译的 Makefile 文件。这个命令的意思是：在当前目录生成 Makefile 文件，并在 /home/Mysql_Complie/mysql-8.0.22/boost 路径下 download boost 库。因为编译 mysql 库时需要依赖 boost。除此之外，主机也需要安装 openssl-dev 库，与 ncurse 库。如果是 ubuntu 环境，可在编译之前输入以下命令进行安装：

```
apt-get install openssl
apt-get install libssl-dev
```

-DCMAKE_INSTALL_PREFIX=/home/Mysql_Complie/mysql-8.0.22/__install 这一句是指定安装路径。

如无意外我们已经生成了 Makefile 文件了，我们分别输入以下命令就可以进行编译与安装:

```
make
make install
```

这样，我们就能在之前定义的安装路径下找到编译好的 mysql 文件。

## 编译boost

源码 <a href="https://www.boost.org/users/download/">在此下载</a>

本次编译的版本为：boost_1_73_0

需要注意的是，不同版本的 mysql 需要的 boost 的版本可能会有不同，编译时需要注意错误提示，并交叉编译对应的版本。

解压之后进入文件夹内，输入以下命令进行编译配置：

```
./bootstrap.sh  --prefix=/home/mysqlCompile/boost_1_73_0/__install
```

此次配置会生成名为 b2 的执行文件，配置结束后在 project-config.jam 文件中修改交叉编译链配置：

```
if ! gcc in [ feature.values <toolset> ]
{
    using gcc : : /home/lanceli/openwrt-toolchain-mediatek-mt7622_gcc-8.3.0_musl.Linux-x86_64/toolchain-aarch64_cortex-a53_gcc-8.3.0_musl/bin/aarch64-openwrt-linux-gcc ;
}
```

注意：冒号间与分号前空格的存在。

依次输入以下命令进行代码的编译与库的安装：

```
./b2

./b2 install
```

这个也可以在配置过程中设置参数使工程自动下载：

```
-DENABLE_DOWNLOADS=1 \
-DDOWNLOAD_BOOST=1 \
-DWITH_BOOST=/home/mysqlCompile/mysql-8.0.22/boost
```


## 编译ncurse

解压之后进入文件夹内，输入以下命令配置编辑方式与安装路径：
```
./configure --prefix=/home/Mysql_Complie/ncurses-6.2/__install --host=aarch64-openwrt-linux  CC=aarch64-openwrt-linux-gcc --with-shared  --without-progs
```

编译与安装：
```
make
make install
```


## mysql编译

解压 mysql-8.0.22 压缩包，将文件 toolChain.camke 复制到 mysql-8.0.22 文件夹中。在 mysql-8.0.22 文件夹中新建目录 BUILD ，将文件 compile-arm64 复制到 BUILD 中。

执行 compile-arm64 文件：

```
./compile-arm64
```