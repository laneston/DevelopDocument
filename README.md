**1. <a href="https://github.com/laneston/Note/blob/master/mangopi_developLog.md">mangopi 开发日志</a>**

此篇文章为个人路由开发板制作日志，记录了硬件开发以及系统移植的详细过程，可以作为嵌入式学习参考。

**2. <a href="https://github.com/laneston/Note/blob/master/Hey_Makefile.md"> Hey Makefile! </a>**

文章通过两个例子来说明 Makefile 是如何运行的，文章用较小的篇幅来概括初学者在编写 Makefile 时会遇到的问题，目的是用最直接简单有效的方式教会大家编写 Makefile。

**3. <a href="https://github.com/laneston/Note/blob/master/Makefile_Note.md"> Makefile_Note </a>**

文章围绕 makefile 文件的编写方式，向读者讲述如何在ubuntu平台上用交叉编译链 arm-none-eabi- 编译出 STM32F4xx 系列 MCU 的执行文件。文章核心在于讲述 arm-none-eabi- 在 Makefile 中的应用方式，对比于嵌入式可视编译器 keil_v5 有什么共同点，编译思维是怎样产生的，由此来完成一个简单项目 <a href="https://github.com/laneston/STM32F4xx_LED-Makefile"> STM32F4xx_LED-Makefile </a> 的编译工作。

**4. <a href="https://github.com/laneston/Note/blob/master/LdScript_Note.md"> LdScript_Note </a>**

文章围绕项目 <a href = "https://github.com/laneston/STM32_RTOS_GUN">STM32_RTOS_GUN</a> 的链接脚本 STM32F417IG_FLASH.ld 进行分析，同时对编写链接脚本的方法进行相应的讲解，尽可能地做到通过阅读这篇文章后能够学会编写简单的链接脚本。

**5. <a href="https://github.com/laneston/Note/blob/master/STM32F4xx-NAND.md"> STM32F4xx-NAND </a>**

文章主要翻译ST手册 RM0090 FSMC NAND应用部分，围绕当前项目：<a href="https://github.com/laneston/SRAM_NAND-STM32F4xx"> SRAM_NAND-STM32F4xx </a> 整理相关内容。同时针对项目应用 NAND Flash MX30LF1G18AC 给出时序分析。

**6. <a href="https://github.com/laneston/Note/blob/master/STM32F4xx-Ether.md"> STM32F4xx-Ether </a>**

文章主要翻译ST手册 RM0090 以太网硬件部分，笔者在其中添加自己的见解，由于篇幅较长，翻译需要分多次完成，主要会围绕当前开源项目的内容进行学习与翻译，其余非主要内容会在文章收尾之后补上。

**7. <a href="https://github.com/laneston/Note/blob/master/quicksort_Note.md"> Hey 快速排序! </a>**

通过一趟排序将要排序的数据分割成独立的两部分，其中一部分的所有数据都比另一部分的所有数据都要小，然后再按此方法对这两部分数据分别进行快速排序，整个排序过程可以递归进行，以此达到整个数据标称有序序列。