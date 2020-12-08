# 笔记与文档集

**1. <a href="https://github.com/laneston/Note/blob/master/Programming_Pearls_Note.md"> Programming Pearls Note </a>**

本篇文章是《编程珠玑》第二版的学习笔记，学习的过程主要以摘抄为主，并且加入自己的见解，目的是做到简浅易懂，方便记忆与理解。书本一共有15章，因此笔记也会分为15章记录。其中知识点以个人理解后的表述方式记录在笔记当中，其中若有错误，请各位读者勘正。

**2. <a href="https://github.com/laneston/Note/blob/master/Ethernet_EVB_Hardware_Note.md"> Ethernet EVB Hardware Note </a>**

文章记录以太网硬件接口的相关资料，包括但不限于MCU、MAC PHY、PCB Layout等。

**3. <a href="https://github.com/laneston/Note/blob/master/STM32F4xx-NAND.md"> STM32F4xx-NAND </a>**

本篇文章主要是翻译ST手册 RM0090 FSMC NAND应用部分，围绕当前项目：<a href="https://github.com/laneston/SRAM_NAND-STM32F4xx"> SRAM_NAND-STM32F4xx </a> 整理相关内容。同时针对项目应用 NAND Flash MX30LF1G18AC 给出时序分析。

**4. <a href="https://github.com/laneston/Note/blob/master/STM32F4xx-Ether.md"> STM32F4xx-Ether </a>**

本篇文章是翻译ST手册 RM0090 以太网硬件部分，本着学习的目的，笔者会在翻译的过程添加自己的见解。由于篇幅较长，翻译需要分多次完成，主要会围绕当前开源项目的内容进行学习与翻译，其余非主要内容会在文章收尾之后补上。

**5. <a href="https://github.com/laneston/Note/blob/master/Hey_Makefile.md"> Hey Makefile! </a>**

本篇文章将会用两个例子来说明 Makefile 是如何运行的，笔者用较小的篇幅来概括初学者在编写 Makefile 时会遇到的问题，这篇文章的目的就是用最直接简单有效的方式教会大家编写 Makefile。在看完这篇文章后，也能开始写自己的 Makefile 。

**6. <a href="https://github.com/laneston/Note/blob/master/Makefile_Note.md"> Makefile_Note </a>**

文章围绕makefile文件的编写方式，向读者讲述如何在ubuntu平台上用交叉编译链 arm-none-eabi- 编译出 STM32F4xx 系列 MCU 的执行文件。文章核心在于讲述 arm-none-eabi- 在 Makefile 中的应用方式，对比于嵌入式可视编译器 keil_v5 有什么共同点，编译思维是怎样产生的，由此来完成一个简单项目 <a href="https://github.com/laneston/STM32F4xx_LED-Makefile"> STM32F4xx_LED-Makefile </a> 的编译工作。

**7. <a href="https://github.com/laneston/Note/blob/master/LdScript_Note.md"> LdScript_Note </a>**

本篇文章主要围绕项目 <a href = "https://github.com/laneston/STM32_RTOS_GUN">STM32_RTOS_GUN</a> 的链接脚本 STM32F417IG_FLASH.ld 进行分析，同时对编写链接脚本的方法进行相应的讲解，尽可能地做到通过阅读这篇文章后能够学会编写简单的链接脚本。

**8. <a href="https://github.com/laneston/Note/blob/master/quicksort_Note.md"> Hey 快速排序! </a>**

通过一趟排序将要排序的数据分割成独立的两部分，其中一部分的所有数据都比另一部分的所有数据都要小，然后再按此方法对这两部分数据分别进行快速排序，整个排序过程可以递归进行，以此达到整个数据标称有序序列。

**9. <a href="https://github.com/laneston/Note/blob/master/LXC_ConfigFiles.md">LXC配置要点</a>**

本篇文章主要讲述 LXC 的使用与配置相关。翻译自 <a href="https://linuxcontainers.org"> Linux containers </a> 网站手册。
