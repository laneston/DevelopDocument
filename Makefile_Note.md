文章围绕makefile文件的编写方式，向读者讲述如何在ubuntu平台上用交叉编译链 arm-none-eabi- 编译出 STM32F4xx 系列 MCU 的执行文件。文章核心在于讲述 arm-none-eabi- 在 Makefile 中的应用方式，对比于嵌入式可视编译器 keil_v5 有什么共同点，编译思维是怎样的，并完成一个简单项目 <a href="https://github.com/laneston/STM32F4xx_LED-Makefile"> STM32F4xx_LED-Makefile </a> 的编译工作。 如果还没对 Makefile 入门的朋友可以查看我的另一篇文章 <a href="https://github.com/laneston/Note/blob/master/Hey_Makefile.md"> Hey Makefile! </a>

- 平台： Ubuntu20，STM32F407
- 工具： arm-none-eabi-

# 初见交叉编译链

为什么要使用交叉编译链工具呢？在嵌入式开发过程中有宿主机和目标机的角色之分：宿主机是执行编译、链接嵌入式软件的计算机；目标机是运行嵌入式软件的硬件平台。简单地说，就是在一个平台上生成另一个平台上的可执行代码。

有朋友可能会问了：为什么要在一个平台上编译另一个平台的执行文件呢？不能像 PC 那样在 PC 上编译 PC 能执行的文件呢？嗯……你觉得能在 STM32 上安装一个编译工具，然后接上鼠标键盘和屏幕，编译出一个LED流水灯程序来吗？你有问过它的 Flash 和 RAM 够用吗？

说到这里，我们该用哪个交叉编译链工具呢？

**<a href = "https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm/downloads">gcc-arm-none-eabi</a>** 

这个交叉编译链工具适用于 Arm Cortex-M & Cortex-R processors (Cortex-M0/M0+/M3/M4/M7/M23/M33, Cortex-R4/R5/R7/R8/R52)系列平台。如何安装这个软件我就不赘述了，因为安装一个软件比学习一个软件容易得多。

## 我们怎么开始

我们不如联想一下用 keil_v5(MDK5) 编译 STM32 代码时的操作过程，如果没有用过这个软件，也可以联想 IAR 亦或是主机编译类软件的操作方式。 keil_v5 封装了编译链接的细节，使得我们觉得编译就是点一下按键的过程，根据我们上面的学习内容，我们认识到编译一个可执行的目标文件远比我们想象的要复杂得多。

万事开头难，中间难，结尾也难。那我们就从最简单的“点灯”程序开始吧，因为越简单的程序，给我们出错的余地就越少。我们先用 keil_v5 编译一个 LED 点灯程序，烧录到开发板中进行验证，以确保这个程序是可以正常工作的。代码的链接与详细说明请看这里：<a href = "https://github.com/laneston/STM32F4xx_LED-Makefile/tree/master/STM32F4xx_LED(keil_v5)">STM32F4xx_LED(keil_v5)</a>

可以验证，上面的工程已经能成功点亮一盏LED，并且以500ms的时间间隔闪烁。

我们可以看到，这个工程中的代码根据功能不同来分类，分别存放在几个文件夹当中。

- STM32F4xx_DSP_StdPeriph_Lib_V1.8.0 [驱动内核与寄存机接口文件]
- Peripheral_BSP[外设驱动]
- User[代码入口]

主要代码分别放在以上3个文件夹当中，而其中一个文件夹 STM32F4xx_DSP_StdPeriph_Lib_V1.8.0 当中又有其他的文件夹。在当前的工程中，我们需要解决文件的依赖关系，如何让 Makefile 文件像 keil_v5 编译器那样，设置 include 路径后能自动查找.c文件所依赖的文件。我们的 Makefile 编写思想还是像<a href="https://github.com/laneston/Note/blob/master/Hey_Makefile.md"> Hey Makefile! </a>中那么简单直接。

## 代码路径

我们先看看这个工程的文件结构：

```
STM32F4XX_LED(MAKEFILE)
    |__output
    |
    |__Peripheral_BSP
    |   |__stm32f4xx_led_bsp.c
    |   |__stm32f4xx_led_bsp.h
    |
    |__STM32F4xx_DSP_StdPeriph_Lib_V1.8.0
    |   |__CMSIS
    |   |   |__Device\ST\STM32F4xx
    |   |   |__Documentation
    |   |   |__DSP_Lib
    |   |
    |   |
    |   |
    |   |__STM32F4x7_ETH_Driver
    |   |__STM32F4xx_StdPeriph_Driver
    |
    |
    |
    |
    |__User
    |
    |__Makefile
```

-Wl选项告诉编译器将后面的参数传递给链接器。