这篇文章主要用于讲述 arm for Linux 交叉编译链安装与使用，目的是使读者通过看完这篇博客后能够对交叉编译链有相关的了解，达到能够使用交叉编译链编译一个简单例程的效果，并在 S5P6818 平台上正常工作。

笔者之前在文章**<a href = "https://github.com/laneston/Note/blob/master/Makefile_Note.md">交叉编译链下的Makefile</a>** 中讲述了 Cortex-M4 平台上的交叉编译方式，也大致说明了 arm-none-eabi-gcc 这款编译器的使用要点，现在我们继续从 Cortex-A53 平台的一系列编译工具出发，带大家深入了解交叉编译链是怎么在 x86-64 平台上编译出 arm 平台上的执行文件。

## 交叉编译器的命名规则及详细解释




