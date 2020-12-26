- 主机平台：ubuntu20.04
- 交叉编译链：aarch64-openwrt-linux-
- mysql运行环境：openwrt

## 编译boost

源码 <a href="https://www.boost.org/users/download/">在此下载</a>

本次编译的版本为：boost_1_75_0

输入以下命令进行编译配置：

```
./bootstrap.sh  --prefix=/home/boost_1_57_0/__install
```

此次配置会生成名为 b2 的执行文件，配置结束后在 project-config.jam 文件中修改交叉编译链配置：

```
if ! gcc in [ feature.values <toolset> ]
{
    using gcc : : /root/openwrt-toolchain-mediatek-mt7622_gcc-8.3.0_musl.Linux-x86_64/toolchain-aarch64_cortex-a53_gcc-8.3.0_musl/bin/aarch64-openwrt-linux-gcc ;
}
```

注意：冒号间与分号前空格的存在。

依次输入以下命令进行代码的编译与库的安装：

```
./b2

./b2 install
```