- 主机平台：ubuntu20.04 (Windows 子系统)
- 交叉编译链：aarch64-none-linux-gnu-
- mysql运行环境：openwrt
- mysql版本：mysql-8.0.22

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