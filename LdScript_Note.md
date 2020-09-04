本篇文章主要围绕项目 <a href = "https://github.com/laneston/STM32_RTOS_GUN">STM32_RTOS_GUN</a> 的链接脚本 STM32F417IG_FLASH.ld 进行分析，同时对编写链接脚本的方法进行相应的讲解，尽可能地做到通过阅读这篇文章后能够学会编写简单的链接脚本。

# 什么是链接脚本

我们在做 GCC 交叉编译时总会遇到一个 .ld 后缀的文件，这个就是我们常说的链接脚本。有朋友可能会疑惑这个脚本的到底有还是呢么用呢，我们在编写 GCC 命令时只需一句 gcc -o hello hello.c 就能同时实现而文件的编译和链接了，怎么在进行交叉编译时就需要这样也给文件呢？那是因为在编译主机运行的程序时，内存分配信息已经由编译器帮你写好了，而文件 STM32F417IG_FLASH.ld 恰恰是用来存放芯片内存信息的。

连接脚本的一个主要目的是描述输入文件中的节如何被映射到输出文件中，并控制输出文件的内存排布，几乎所有的连接脚本只做这两件事情。但是,在需要的时候，连接器脚本还可以指示连接器执行很多其他的操作。

连接器总是使用连接器脚本的，如果我们不提供，连接器会使用一个缺省的脚本，这个脚本是被编译进连接器可执行文件的。我们可以使用 '--verbose' 命令行选项来显示缺省的连接器脚本的内容。 某些命令行选项,比如 '-r' 或 '-N'，会影响缺省的连接脚本。

我们可以过使用 '-T' 命令行选项来提供你自己的连接脚本，就像我们 <a href = "https://github.com/laneston/STM32_RTOS_GUN">STM32_RTOS_GUN</a> 里 Makefile 文件写的那样。当这么做的时候, 连接脚本会替换缺省的连接脚本。

## 基本概念

总所周知，连接器把多个输入文件合并成单个输出文件。当中输出文件和输入文件都以一种叫做 “目标文件格式” 的可执行文件存在，譬如 .o 和 .elf 文件。每一个目标文件中都有一个节列表。我们把输入文件的节叫做输入节(input section)，相似的，输出文件中的一个节叫做输出节(output section)。目标文件的每个 section 至少包含两个信息: 名字和大小. 大部分 section 还包含与它相关联的一块数据, 称为 section contents(section内容)。 一个 section 可被标记为 “loadable(可加载的)” 或 “allocatable(可分配的)” 。这个概念笔者曾在编程笔记： <a href = "https://github.com/laneston/Note/blob/master/Makefile_Note.md">Makefile_Note</a>里 **编译优化** 一节中有提到过。

## VMA和LMA

每个“可加载的”或“可分配的”输出 section 通常包含两个地址: VMA(virtual memory address 虚拟内存地址或程序地址空间地址)和 LMA(load memory address 加载内存地址或进程地址空间地址)。通常 VMA 和 LMA 是相同的。

在目标文件中, loadable 或 allocatable 的输出 section 有两种地址: VMA(virtual Memory Address)和LMA(Load Memory Address)。 VMA是执行输出文件时section所在的地址，而LMA是加载输出文件时 section 所在的地址。一般而言，某 section 的VMA 与 LMA 相同。但在嵌入式系统中，经常存在加载地址和执行地址不同的情况: 比如将输出文件加载到开发板的 flash 中(由LMA指定)，而在运行时将位于 flash 中的输出文件复制到 RAM 中(由VMA指定)。我们把一些初始化的值先烧到 ROM 中，我们设定 LMA 为ROM 的地址，VMA 为 RAM 的地址。程序在内存中执行时，是根据 VMA 地址来执行的，也就是PC指针使用的是 VMA 地址空间。Loader下载程序时，Loader先将某些块程序拷贝到 ROM，但由于 ROM 只读，所以 ROM 的值不会变。接着比较 LMA 和 VMA，如果不相等，则把 LMA 处的内容拷贝到 VMA 所指向的位置，这样就把 ROM 处的值拷贝到了 RAM。

## 链接脚本

我们可以开始了解一个完整的脚本里应该包含什么样的内容，现在我们就来看一下 ST 的链接脚本文件 STM32F417IG_FLASH.ld

```
ENTRY(Reset_Handler)

_estack = 0x2001FFFF;        /* end of RAM */

_Min_Heap_Size = 0x200;      /* required amount of heap  */
_Min_Stack_Size = 0x400;     /* required amount of stack */

MEMORY
{
FLASH (rx)       : ORIGIN = 0x8000000, LENGTH = 1024K
RAM (xrw)        : ORIGIN = 0x20000000, LENGTH = 128K
CCMRAM (rw)      : ORIGIN = 0x10000000, LENGTH = 64K
}

SECTIONS
{
  .isr_vector :
  {
    . = ALIGN(4);
    KEEP(*(.isr_vector))
    . = ALIGN(4);
  } >FLASH

  .text :
  {
    . = ALIGN(4);
    *(.text)           /* .text sections (code) */
    *(.text*)          /* .text* sections (code) */
    *(.glue_7)         /* glue arm to thumb code */
    *(.glue_7t)        /* glue thumb to arm code */
    *(.eh_frame)
    
    KEEP (*(.init))
    KEEP (*(.fini))

    . = ALIGN(4);
    _etext = .;        /* define a global symbols at end of code */
  } >FLASH

  /* Constant data goes into FLASH */
  .rodata :
  {
    . = ALIGN(4);
    *(.rodata)
    *(.rodata*)
    . = ALIGN(4);
  } >FLASH

  .ARM.extab   : { *(.ARM.extab* .gnu.linkonce.armextab.*) } >FLASH
  .ARM : {
    __exidx_start = .;
    *(.ARM.exidx*)
    __exidx_end = .;
  } >FLASH

  .preinit_array     :
  {
    PROVIDE_HIDDEN (__preinit_array_start = .);
    KEEP (*(.preinit_array*))
    PROVIDE_HIDDEN (__preinit_array_end = .);
  } >FLASH

  .init_array :
  {
    PROVIDE_HIDDEN (__init_array_start = .);
    KEEP (*(SORT(.init_array.*)))
    KEEP (*(.init_array*))
    PROVIDE_HIDDEN (__init_array_end = .);
  } >FLASH

  .fini_array :
  {
    PROVIDE_HIDDEN (__fini_array_start = .);
    KEEP (*(SORT(.fini_array.*)))
    KEEP (*(.fini_array*))
    PROVIDE_HIDDEN (__fini_array_end = .);
  } >FLASH

  _sidata = LOADADDR(.data);

  .data : 
  {
    . = ALIGN(4);
    _sdata = .;
    *(.data)
    *(.data*)

    . = ALIGN(4);
    _edata = .;
  } >RAM AT> FLASH

  _siccmram = LOADADDR(.ccmram);

  .ccmram :
  {
    . = ALIGN(4);
    _sccmram = .;
    *(.ccmram)
    *(.ccmram*)
    
    . = ALIGN(4);
    _eccmram = .;
  } >CCMRAM AT> FLASH

  . = ALIGN(4);
  .bss :
  {
    _sbss = .;
    __bss_start__ = _sbss;
    *(.bss)
    *(.bss*)
    *(COMMON)

    . = ALIGN(4);
    _ebss = .;
    __bss_end__ = _ebss;
  } >RAM

  ._user_heap_stack :
  {
    . = ALIGN(4);
    PROVIDE ( end = . );
    PROVIDE ( _end = . );
    . = . + _Min_Heap_Size;
    . = . + _Min_Stack_Size;
    . = ALIGN(4);
  } >RAM  

  /DISCARD/ :
  {
    libc.a ( * )
    libm.a ( * )
    libgcc.a ( * )
  }

  .ARM.attributes 0 : { *(.ARM.attributes) }
}

```

这是 <a href = "https://github.com/laneston/STM32_RTOS_GUN">STM32_RTOS_GUN</a> 的链接脚本，项目基于 STM32F407 硬件平台进行开发，笔者把脚本中的注释去除掉了，让它看起来尽可能地简短，但我们看起来仍旧觉得凌乱不堪。其实链接脚本可以做到很简单，之所以举这个例子，是因为笔者经历过后思维比较清晰，而其中涉及的内容也较为普遍常用，没什么花里胡哨的技巧。

# 脚本分析与学习

笔者想先在这里说明一下，以便减轻一些初接触链接文件的朋友的疑惑。在链接文件中定义的变量是会其他文件(譬如 startup.s)中声明的，而这里只是做一个引用操作，所以在这里所用的变量需要与其他文件进行一个联系。举个例子：在脚本第一行中的 Reset_Handler 的本体要追溯到文件：STM32F4xx_DSP_StdPeriph_Lib_V1.8.0\CMSIS\Device\ST\STM32F4xx\Source\Templates\TrueSTUDIO\startup_stm32f40_41xxx.s 中的 69 行，在那里定义了 Reset_Handler 是一个 .text section。

## 简单脚本命令

好了，话不多说我们就看脚本的第一行：

```
ENTRY(Reset_Handler)
```

从字面看，我们可以猜出这是一个 Reset 的中断句柄。指令被称为入口点 entry point，可以使用 ENTRY 链接脚本命令设置 entry point，参数是一个符号名。有几种方法可以设置 entry point,链接器会按照如下的顺序来尝试各种方法，只要任何一种方法成功就会停止：

1. ld命令行的-e选项
2. 连接脚本的ENTRY(SYMBOL)命令
3. 如果定义了start符号, 使用start符号值
4. 如果存在.text section, 使用.text section的第一字节的位置值
5. 使用值0

```
_estack = 0x2001FFFF;
```

这段声明内存末尾地址。

```
_Min_Heap_Size = 0x200;
_Min_Stack_Size = 0x400;
```

这段定义了堆和栈的最小空间大小。如果定义的数值不符合内存的规格，在编译时会产生链接错误。

## MEMORY命令

```
MEMORY
{
FLASH (rx)       : ORIGIN = 0x8000000, LENGTH = 1024K
RAM (xrw)        : ORIGIN = 0x20000000, LENGTH = 128K
CCMRAM (rw)      : ORIGIN = 0x10000000, LENGTH = 64K
}
```

这段定义了 FLASH RAM 和 CCMRAM 的起始地址和长度，(xrw)表明了权限，r是读、w是写、x是执行，这个和 Linux 中的 shell 命令一样。

连接器在缺省状态下被配置为允许分配所有可用的内存块，所以我们可以使用 ‘MEMORY’ 命令重新配置这个设置。‘MEMORY’ 命令描述目标平台上内存块的位置与长度。我们可以用它来描述哪些内存区域可以被连接器使用，哪些内存区域是要避免使用的，然后我们就可以把节(section)分配到特定的内存区域中。连接器会基于内存区域设置节的地址，对于太满的区域，会提示警告信息。连接器不会为了适应可用的区域而搅乱节。一个连接脚本最多可以包含一次MEMORY命令。但可以在命令中定义任意的内存块。

一旦你定义了一个内存区域，可以指示连接器把指定的输出段放入到这个内存区域中，这可以通过使用 ‘>REGION’ 输出段属性，这种操作可以在之后的内容中看到。

## SECTIONS

这个部分是 .ld 文件的核心部分，将会用较大篇幅去讲述，请各位看官耐心。

### SECTIONS 命令

**SECTIONS 命令** 是脚本文件中最重要的元素，不可缺省。它的作用就是用来描述输出文件的布局。

```
SECTIONS
{
    ...
    secname:{
        contents
    }
    ...
}
```

secname 和 contents 是必须的，其他都是可选的参数。SECTIONS 命令告诉 .ld 文件如何把输入文件的 sections 映射到输出文件的各个 section；如何将输入 section 合为输出 section；如何把输出 section 放入程序地址空间 (VMA) 和进程地址空间 (LMA) 。如果整个连接脚本内没有 SECTIONS 命令, 那么 .ld 将所有同名输入 section 合成为一个输出 section 内, 各输入 section 的顺序为它们被连接器发现的顺序。如果某输入 section 没有在 SECTIONS 命令中提到，那么该 section 将被直接拷贝成输出 section。

说到这里，很多朋友可能有点懵了，不过不要紧，毕竟掌握的信息还很少，尚不足以产生理解的质变，我们接着分析 SECTIONS 命令里面的内容。

### 输出 section 的描述

```
  .isr_vector:
  {
    . = ALIGN(4);
    KEEP(*(.isr_vector)) /* Startup code */
    . = ALIGN(4);
  } >FLASH
```

这几行脚本就是一个输出 section 的描述，我们可以看看其中的格式：

```
SECTION-NAME [ADDRESS] [(TYPE)] : [AT(LMA)]
{
  OUTPUT-SECTION-COMMAND
  OUTPUT-SECTION-COMMAND
  …
} [>REGION] [AT>LMA_REGION] [:PHDR HDR ...] [=FILLEXP]
```

我们看到上面的输出 section 命令与讲述的格式不太一样，少了许多东西。是的，中括号[]里的东西是可以省略的。

.SECTION-NAME 左右的空白与冒号是必须的。所以上面的一段命令中 .SECTION-NAME 是 .isr_vector 。作为启动代码在 startup_stm32f40_41xxx.s 中定义： .section  .isr_vector,"a",%progbits

输出 section 名字 SECTION-NAME 必须符合输出文件格式要求，比如：.o 格式的文件只允许存在.text .data 和.bss 的section 名。而有的格式只允许存在数字名字，那么此时应该用引号将所有名字内的数字组合在一起；另外，还有一些格式允许任何序列的字符存在于 section 名字内，此时如果名字内包含特殊字符(比如空格、逗号等)，那么需要用引号将其组合在一起。

输出 section 地址[ADDRESS]是一个表达式，它的值用于设置VMA。如果没有该选项且有 REGION 选项，那么连接器将根据 REGION 设置 VMA；如果也没有 REGION 选项，那么连接器将根据定位符号 ‘.’ 的值设置该 section 的 VMA，将定位符号的值调整到满足输出section 对齐要求后的值，这时输出 section 的对齐要求为：该输出 section 描述内用到的所有输入 section 的对齐要求中最严格的对齐要求。

. = ALIGN(4); 是对齐格式。

>FLASH 意思是该 section 加载在 Flash 当中。

至于 KEEP(*(.isr_vector)) 这一句命令，我们需要分两部分来看。

*(.isr_vector) 是通配符格式，表明的是所有输入文件的 .isr_vector section

在连接命令行内使用了选项 –gc-sections 后，连接器可能将某些它认为没用的 section 过滤掉，此时就有必要强制连接器保留一些特定的 section，可用 KEEP() 关键字达此目的。

整段的意思是：把所有文件 .isr_vector section 段命名为 .isr_vector section，并链接到 MEMORY 定义的 FLASH 中。

## 输出 section 的进阶描述

我们看以下这段命令：

```
  .text :
  {
    . = ALIGN(4);
    *(.text)           /* .text sections (code) */
    *(.text*)          /* .text* sections (code) */
    *(.glue_7)         /* glue arm to thumb code */
    *(.glue_7t)        /* glue thumb to arm code */
    *(.eh_frame)
    
    KEEP (*(.init))
    KEEP (*(.fini))

    . = ALIGN(4);
    _etext = .;        /* define a global symbols at end of code */
  } >FLASH
```

在进行分析之前先普及点基础知识：

1. .text   段是用来存放程序执行代码的区；
2. .data   段通常是用来存放程序中已初始化的全局变量的一块内存区域，属于静态内存分配。
3. .bss    段通常是指用来存放程序中未初始化的全局变量的一块内存区域。属于静态内存分配。
4. .rodata 段通常是指用来存放程序中常量的一块内存区域。属于静态内存分配。

*(.text) 指示将工程中所有输入文件 .o 的 代码段(.text) 链接到 MEMORY 定义的 FLASH 中。

\*(.text*) 指示将工程中所有目标文件的 .text 段链接到 FLASH 中。

链接过程是按顺序执行的，即先链接 .o 文件，再链接其他目标文件。





