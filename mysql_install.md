# mysql交叉编译与安装

- 主机编译平台：ubuntu20.04 (Windows 子系统)
- 交叉编译链：aarch64-none-linux-gnu-
- mysql版本：mysql-5.7.32
- mysql运行环境：openwrt

mysql交叉编译的主要流程是：主机编译 mysql，交叉编译 boost 库，交叉编译 ncurse 库，交叉编译 openssl 库，最后交叉编译 mysql 库。值得注意的是，mysql 对依赖库的版本有特殊指定，请尽量按照本说明对应的版本进行操作，如果依赖库没有达到 mysql 指定的版本新度，会出现配置不通过。

以下我们就开始按编译顺序一步步操作吧。

## mysql的主机编译

之所以第一步要主机编译 mysql，第一是为了熟悉编译的流程，但最重要的是，我们之后交叉编译需要用到主机编译时生成的脚本。

编译过程在此说明。

解压 mysql-5.7.32 压缩包，进入文件夹第一层目录，输入以下命令：

```
cmake . -DDOWNLOAD_BOOST=1 -DWITH_BOOST=/home/mysqlCompile/mysql-5.7.32/boost -DCMAKE_INSTALL_PREFIX=/home/mysqlCompile/mysql-5.7.32/__install
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

## 编译boost库

源码 <a href="https://www.boost.org/users/download/">在此下载</a>

本次编译的版本为：boost_1_59_0

需要注意的是，不同版本的 mysql 需要的 boost 的版本可能会有不同，编译时需要注意错误提示，并交叉编译对应的版本。

解压之后进入文件夹内，输入以下命令进行编译配置：

```
./bootstrap.sh  --prefix=/home/mysqlCompile/boost_1_59_0/__install
```

相应路径可以根据自己需要进行修改。

此次配置会生成名为 b2 的执行文件，配置结束后在 project-config.jam 文件中修改交叉编译链配置：

```
if ! gcc in [ feature.values <toolset> ]
{
    using gcc : : /home/lance/gcc-arm-10.2-2020.11-x86_64-aarch64-none-linux-gnu/bin/aarch64-none-linux-gnu-gcc ;
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

## 编译ncurse库

解压之后进入文件夹内，输入以下命令配置编辑方式与安装路径：
```
./configure --prefix=/home/mysqlCompile/ncurses-6.2/__install --host=aarch64-none-linux-gnu  CC=aarch64-none-linux-gnu-gcc --with-shared  --without-progs
```

编译与安装：

```
make
make install
```

分别输入以上命令即可对 ncurse 库进行编译与安装。

## 编译openssl库

执行以下命令，分别对 openssl 库进行配置编译与安装。

```
./config --prefix=/home/mysqlCompile/openssl-OpenSSL_1_1_1g/__install --cross-compile-prefix=aarch64-none-linux-gnu- no-asm shared
make
make install
```

以上相对路径可根据自己的本地路径进行修改。

## 编译tirpc库

## mysql的交叉编译

mysql 交叉编译的过程主要是通过 cmake 生成相应的配置文件与 Makefile，然后再执行 Makefile 脚本文件生成相应的目标文件。在用 cmake 生成 Makefile 文件之间，我们需要对 mysql 工程进行一些修改。

### CMakeLists.txt文件的修改

打开一级目录下的 CMakeLists.txt 文件，将以下配置信息复制到文件首部，并保存文件。

```
# this is required
SET(CMAKE_SYSTEM_NAME Linux)
SET(CMAKE_CROSSCOMPILING TRUE)

# specify the cross compiler
SET(CMAKE_C_COMPILER /home/lanceli/gcc-arm-10.2-2020.11-x86_64-aarch64-none-linux-gnu/bin/aarch64-none-linux-gnu-gcc)
SET(CMAKE_CXX_COMPILER /home/lanceli/gcc-arm-10.2-2020.11-x86_64-aarch64-none-linux-gnu/bin/aarch64-none-linux-gnu-g++)

# where is the target environment 
SET(CMAKE_FIND_ROOT_PATH  /home/lanceli/gcc-arm-10.2-2020.11-x86_64-aarch64-none-linux-gnu)

# search for programs in the build host directories (not necessary)
SET(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
# for libraries and headers in the target directories
SET(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
SET(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
SET(CMAKE_FIND_ROOT_PATH_MODE_PACKAGE ONLY)

# configure Boost
SET(BOOST_ROOT /home/mysqlCompile/boost_1_59_0/__install)
SET(BOOST_INCLUDE_DIR /home/mysqlCompile/boost_1_59_0/__install/include)
SET(BOOST_LIBRARY_DIR /home/mysqlCompile/boost_1_59_0/__install/lib)

# openssl configuration
SET(OPENSSL_INCLUDE_DIR /usr/local/opt/openssl/include)
SET(OPENSSL_LIBRARY /usr/local/opt/openssl/lib/libssl.so)
SET(CRYPTO_LIBRARY /usr/local/opt/openssl/lib/libcrypto.so)

SET(CMAKE_CXX_LINK_FLAGS "-L/usr/local/opt/openssl/lib -lssl -lcrypto")
```

这里分别指定了：

1. 编译的目标环境；
2. 交叉编译链工具的路径信息；
3. 交叉编译的环境信息；
4. cmake 寻找链接文件的规则；
5. boost 库的路径信息；
6. openssl库路径信息。

修改完 CMakeLists.txt 文件后，我们还需要修改 libevent.cmake 文件。

## 修改 libevent.cmake 文件

生成配置文件的过程中会有找不到 libevent-2.1.11-stable 版本信息的警告。

这个问题是由于生成的目标文件是运行在目标平台上的ARM文件，由于平台的不同，基本TRY_RUN() 函数执行这个目标文件，文件也是无法在主机平台上运行的。而这和目标文件的动作是获取 libevent 库的版本信息。为了使配置成功执行，我们需要做的是，让 libevent 库的版本信息成功写入到配置文件当中。以下是需要修改的部分：

```
MACRO(FIND_LIBEVENT_VERSION)

  SET(LIBEVENT_VERSION "2.1.11-stable")
  SET(COMPILE_TEST_RESULT TRUE)
  SET(RUN_OUTPUT "2.1.11-stable")

  # MESSAGE(STATUS "TRY_EVENT TEST_RUN_RESULT is ${TEST_RUN_RESULT}")
  # MESSAGE(STATUS "TRY_EVENT COMPILE_TEST_RESULT is ${COMPILE_TEST_RESULT}")
  # MESSAGE(STATUS "TRY_EVENT COMPILE_OUTPUT_VARIABLE is ${OUTPUT}")
  # MESSAGE(STATUS "TRY_EVENT RUN_OUTPUT_VARIABLE is ${RUN_OUTPUT}")

  IF(COMPILE_TEST_RESULT)
    SET(LIBEVENT_VERSION_STRING "${RUN_OUTPUT}")
    STRING(REGEX REPLACE
      "([.-0-9]+).*" "\\1" LIBEVENT_VERSION "${LIBEVENT_VERSION_STRING}")
    MESSAGE(STATUS "LIBEVENT_VERSION_STRING ${LIBEVENT_VERSION_STRING}")
    MESSAGE(STATUS "LIBEVENT_VERSION (${WITH_LIBEVENT}) ${LIBEVENT_VERSION}")
  ELSE()
    MESSAGE(WARNING "Could not determine LIBEVENT_VERSION")
  ENDIF()
ENDMACRO()
```

上面 cmake 命令修改的部分是将版本信息直接定义到 LIBEVENT_VERSION 变量中，跳过了代码编译与运行的步骤。

修改完毕后执行一次 cmake 命令：

```
cmake . -DENABLE_DOWNLOADS=1 -DWITH_BOOST= /home/mysqlCompile/boost_1_59_0/__install -DCMAKE_INSTALL_PREFIX=/home/mysqlCompile/mysql-5.7.32/__install -DCURSES_INCLUDE_PATH=/home/mysqlCompile/ncurses-6.2/__install/include -DCURSES_LIBRARY=/home/mysqlCompile/ncurses-6.2/__install/lib/libncurses.so -DSTACK_DIRECTION=1 -DWITH_LIBEVENT="bundled"
```

如果不能正常生成 Makefile 文件并伴有以下错误信息 ：

```
CMake Error: TRY_RUN() invoked in cross-compiling mode, please set the following cache variables appropriately: 
```

可再执行一次上一次的配置命令。

