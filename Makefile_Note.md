本编文章是关于交叉编译链的学习笔记当中的核心篇目。文章围绕makefile文件的编写方式，向读者讲述如何在ubuntu平台上用交叉编译链编译出 STM32F4xx 系列 MCU 的执行文件。文章核心在于讲述 Makefile 的原理及应用方式，对比于嵌入式可视编译器 keil_v5 有什么共同点，编译思维是怎样产生的，由此来完成一个模板项目 <a href="https://github.com/laneston/STM32_RTOS_GUN"> STM32_RTOS_GUN </a> 的编译工作。


# 什么是Makefile

Makefile文件表述的是文件的依赖关系，告诉编译器怎么去编译和链接当中的文件。如果能掌握Makefile文件的编写方法，就能脱离可视化编译器，使用编译链工具编译出所需的目标文件。

## 编译和链接

编译和链接是获得目标文件的主要方式，在常见的stm32f1xx/stm32f4xx系列的编程任务中，我们可能会用到keil/MDK这个工具，当点击其中的build按键时，我们就能在输出文件夹中获得bin（二进制）或者HEX（十六进制）文件。之后，我们就可以通过MDK自带的烧录工具，使用JLink/STLink将目标文件烧录到芯片的Flash当中，抑或是通过ISP工具烧录到芯片的Flash当中。

在这个过程当中，编译器已经帮我们执行了编译和链接两个步骤。以下我会通过简单的例子说明这一个过程是如何在编译链上体现的。

创建一个名为 main.c 的文件，并写入以下代码：

```C
#include <stdio.h>

int main(int argv, char argc[])
{
    int num1=0;
    int num2=0;
    int revalue=0;

    printf("please input num 1\r\n");
    scanf("%d",&num1);

    printf("please input num 2\r\n");
    scanf("%d",&num2);

    revalue = num1 + num2;

    printf("result is ：%d\r\n", revalue);
    return 0;
}
```

我们在Ubuntu平台上用 **GCC编译器** 编译以上代码：在当前文件夹内，使用以下命令：

```
gcc main.c -o app
```

<img src="https://github.com/laneston/Pictures/blob/master/Post-Makefile/20200812144507.jpg" width="50%" height="50%">

我们可以从上图所示，得到一个名为 app 的目标文件。

输入以下命令即可执行目标文件:

```
./app
```
以上过程便是我们说的编译与链接。

### 编译

其实，一个完成的编译过程包括：预处理、编译、汇编、链接。我们通常把 预处理+编译+汇编 统称为编译。

1. **预处理：** 展开头文件/宏替换/去掉注释/条件编译；
2. **编译：**   检查语法，生成汇编；
3. **汇编：**   汇编代码转换机器码；
4. **链接：**    链接到一起生成可执行程序 。

我们先了解一下编译器常用的的命令：

|  选项 |含义|
|:-----:|:---|
|-v|查看gcc编译器的版本，显示gcc执行时的详细过程|
|-o|指定输出文件名为file，这个名称不能跟源文件名同名|
|-E|只预处理，不会编译、汇编、链接|
|-S|只编译，不会汇编、链接|
|-c|编译和汇编，不会链接|

如果我们想看编译出的中间文件，可以写入以下命令：

```
gcc -S main.c
```

<img src="https://github.com/laneston/Pictures/blob/master/Post-Makefile/20200812152754.jpg" width="50%" height="50%">

以上我们可以看到，文件夹中多了一个 .s 文件，这是汇编格式的文件。

如果我们写入以下命令，即可得到常用的中间文件（.o 文件）：

```
gcc -c main.c
```

### 链接

链接顾名思义是为某些有关联的目标建立关系。在这个过程当中，主要是链接函数和全局变量。在链接过程当中，链接器不理会源文件（.c文件），只管编译时生成的中间文件（.o文件）。我们在使用keil编译时，在输出文件夹中，也可以看到相关的 .o 文件，那就是编译器生成的中间文件。

我们可以使用以下命令，将 main.o 链接生成目标文件：

```
gcc main.o -o app
```

# 如何编写Makefile

经过以上的学习，我们已经大致了解编译和链接到底是怎么回事了。我们也清楚，即便不用 Makefile 文件也能编译出目标文件。那我们继续学习下去，了解 Makefile 在编译当中充当的角色的重要性。

我们编写 Makefile 的规则是：

1. 如果这个工程没有编译过，那么我们的所有c文件都要编译并被链接。
2. 如果这个工程的某几个c文件被修改，那么我们只编译被修改的c文件，并链接目标程序。
3. 如果这个工程的头文件被改变了，那么我们需要编译引用了这几个头文件的c文件，并链接目标程序。

## Makefile的编写规则

```
target... : prerequisites ...

    command
```

- **target** 是一个目标文件，可以是中间文件，也可以是执行文件，还可以是一个标签（Label）。
- **prerequisites** 就是要生成那个 **target** 所需要的文件。
- **command** 是 make 需要执行的命令（任意的Shell命令）。需要注意的是，命令要以一个Tab键作为开头。

## Makefile的基本应用

以下我们将会用一个例子来展示 Makefile 的应用。

**文件1: main.c**

```
#include <stdio.h>
#include "addition.h"
#include "subtraction.h"

int main(int argv, char argc[])
{
    char symbol;
    int num1=0;
    int num2=0;
    int result;

    printf("please input the operation symbol\r\n");
    scanf("%c", &symbol);

    printf("please input num 1\r\n");
    scanf("%d",&num1);

    printf("please input num 2\r\n");
    scanf("%d",&num2);

    switch(symbol)
    {
        case '+':
            result = addition(num1, num2);
            break;
        case '-':
            result = subtraction(num1, num2);
            break;
        default:
            break;
    }
    printf("the result is:%d\r\n", result);

    return 0;
}
```

**文件2: addition.c**

```
int addition(int a, int b)
{
    return (a+b);
}
```

**文件3: subtraction.c**

```
int subtraction(int a, int b)
{
    return (a-b);
}
```

以上有3个文件，文件内容浅显易懂，所以不再说明。现在我们分析3者的依赖关系，然后写对应的 Makefile 文件。

文件1为主函数，主函数中有两个子函数，分别是加法运算函数 addition(int a, int b) 和减法运算函数 subtraction(int a, int b)。所以我们得知，要生成目标文件 app，除了需要 main.o 文件，还需要依赖 addition.o 和 subtraction.o 两个文件。

而生成 main.o 文件需要依赖 main.c文件；而生成 subtraction.o 文件需要依赖 subtraction.c文件；而生成 addition.o 文件需要依赖 addition.c文件。

由此我们可以得出 Makefile 的书写方式：

```
app: main.o addition.o subtraction.o
	gcc -o app main.o addition.o subtraction.o

main.o: main.c
	gcc -c main.c

addition.o: addition.c
	gcc -c addition.c

subtraction.o:subtraction.c
	gcc -c subtraction.c
```

<img src="https://github.com/laneston/Pictures/blob/master/Post-Makefile/20200812165429.jpg" width="50%" height="50%">

上图便是编译与运行的结果。

## 依赖关系的初步分析

根据上面的实验，我们已经成功编写了一个 Makefile 文件。下面笔者将会告诉大家这个过程是怎么进行的。

根据上图所示，第一条执行的命令是：gcc -c main.c 

执行这一条命令时，就用到了预处理。因为main.c 文件中有 #include 语法，预处理就是将要包含(include)的文件插入原文件中。 在例子中就是 addition.h subtraction.h文件。

第二第三条执行的命令都是编译加法与减法两条子函数所在的文件。

第四条执行的命令是将生成的中间文件 main.o addition.o subtraction.o 链接目标文件 app

到此为止，Makefile 文件就做了这几件事。

## Makefile文件的变换

之前说到，预处理就是将要包含(include)的文件插入原文件中，例子中就是 addition.h subtraction.h文件。其实我们可以省下这个预处理，只需要把文件依赖关系写在 Makefile 文件中就可以。

```
app: main.o addition.o subtraction.o
	gcc -o app main.o addition.o subtraction.o

main.o: main.c addition.h subtraction.h
	gcc -c main.c

addition.o: addition.c
	gcc -c addition.c

subtraction.o:subtraction.c
	gcc -c subtraction.c
```

这样也能成功编译，但可能会有警告，因为 addition.h subtraction.h 只声明了函数，编译器因为找不到函数的定义而产生隐形声明警告。

到此为止，我们已经基本学会些 Makefile 了，接下来我们将会进一步学习如何在复杂的情况中应用 Makefile 。

# Makefile还可以做什么

当一个工程项目十分庞大，所涉及的文件数目也会因此递增，并且伴随着频繁的文件增减，Makefile文件也需要跟着频繁修改，这会令写 Makefile 变得十分困难，而且容易出错。有没有什么办法可以写一个“万能”的 Makefile 文件，从而一劳永逸呢？答案是肯定的！在一个工程模板不变的情况下，是可以做到编写一个通用 Makefile 的。

## Makefile中使用变量

我们可以看到 main.o addition.o subtraction.o 这几个语句出现了两次，可以用一个变量来表示这几个语句，以防在复杂的 Makefile 文件中漏了增加某个语句。

```
objects = main.o addition.o subtraction.o

app:  $(objects)
	gcc -o app $(objects)

main.o: main.c
	gcc -c main.c

addition.o: addition.c
	gcc -c addition.c

subtraction.o:subtraction.c
	gcc -c subtraction.c
```

以上例子当中，我们看到用 objects 这个变量来表示 main.o addition.o subtraction.o 这几个文件。在调用的时候需要用 $() 来包括住变量。

## 让make自动推导

GNU的 make 很强大，它可以自动推导文件以及文件依赖关系后面的命令，于是我们就没必要去在每一个[.o]文件后都写上类似的命令，因为，我们的 make 会自动识别，并自己推导命令。

```
objects = main.o addition.o subtraction.o

app:  $(objects)
	gcc -o app $(objects)

main.o: main.c

addition.o: addition.c

subtraction.o:subtraction.c
```

由上面的例子可见，我们少写了几句编译命令，但 Make 还是能帮我们去推导出 .o 的依赖文件。这就是 make 的隐形规则。

## 清空编译文件

我们发现每次编译的过程中都会产生一大堆中间文件，这令人感到杂乱，每个 Makefile 中都应该写一个清空目标文件（.o和执行文件）的规则，这不仅便于重编译，也很利于保持文件的清洁。

```
objects = main.o addition.o subtraction.o

app:  $(objects)
	gcc -o app $(objects)

main.o: main.c

addition.o: addition.c

subtraction.o:subtraction.c

.PHONY : clean
clean :
    -rm  $(objects)
```

输入命令行：

```
make clean
```

就可以清除对应的文件了。

.PHONY 意思表示 clean 是一个“伪目标”，在rm命令前面加了一个小减号的意思就是，也许某些文件出现问题，但不要管，继续做后面的事。clean 的规则不要放在文件的开头，不然，这就会变成make 的默认目标，一般我们都会放在文件结尾处。

# Makefile的进阶应用

虽然可以用一个变量来减少 Makefile 的复杂度，也可以用一些隐式规则来简化 Makefile 的编写，但在一些大型的工程当中，这个技巧的使用带来的收益仍旧微乎其微。我们可以联想到提起过的 keil_v5 编译器，是否可以做到像 keil_v5 编译器那样，可以指定文件路径，让工具自动去查找和关系文件？我们先不要着急寻找这个答案，在此之前，还需要进一步了解相关的知识。

## 使用通配符

通配符是一种特殊语句，主要有星号 (*) 和问号 (?) ，用来模糊搜索文件。

make 支持三各通配符：*，? 和 [...]

**波浪号(~)**字符在文件名中也有比较特殊的用途。如果是 ~/test ，这就表示当前用户的$HOME目录下的test目录。而 ~lance/test 则表示用户 lance 的宿主目录下的test目录。这个知识点与 Linux 下命令符旁通。

通配符代替了一系列文件，如 *.c 表示所以后缀为c的文件。

一个需要我们注意的是，如果我们的文件名中有通配符，如：* 那么可以用转义字符“\”，如“\*”来表示真实的 * 字符，而不是任意长度的字符串。

按这个原理来，是否可以作以下转变呢？

objects = main.o addition.o subtraction.o

objects = *.o

其实以上两条式子并不是等价的，因为在 Makefile 规则中，通配符会被自动展开。但在变量的定义和函数引用时，通配符将失效。这种情况下如果需要通配符有效，就需要使用函数“wildcard”，它的用法是：$(wildcard PATTERN...) 。

在当前工程中，我们可以这样使用：

```
objects := $(patsubst %.c,%.o,$(wildcard *.c))

app:  $(objects)
	gcc -o app $(objects)

.PHONY : clean
clean :
	-rm  $(objects)
```

首先使用“wildcard”函数获取工作目录下的.c文件列表；之后将列表中所有文件名的后缀.c替换为.o。这样我们就可以得到在当前目录可生成的.o文件列表。

这里我们使用了 make 的隐含规则来编译.c的源文件。对变量的赋值也用到了一个特殊的符号（:=）。

到这一步，我们已经大大地简化了Makefile文件的编写，现在如果要继续为这个程序增添功能，只需要在当前文件夹内增加相关的 .c 或 .h 文件，而无需修改 Makefile 文件了。

## 通配符的灵活用法

在进一步解析其中的使用方式之前，我们先来了解以下几个常用概念：

**常用函数：**
 
- $(subst from, to, text)  把字串 text 中的 from 字符串替换成 to。
- $(patsubst pattern, replacement, text)  查找 text 中的单词（单词以“空格”、“Tab”或“回车”“换行”分隔）是否符合模式 pattern ，如果匹配的话，则以 replacement 替换。这里， pattern 可以包括通配符“%”，表示任意长度的字串。如果  replacement 中也包含“%”，那么， replacement 中的这个“%”将是 pattern 中的那个“%”所代表的字串。（可以用“\”来转义，以“\%”来表示真实含义的“%”字符）
- $(notdir names)  从文件名序列 names 中取出非目录部分。非目录部分是指最后一个反斜杠（“/”）之后的部分。
- $(wildcard PATTERN) 获取匹配 PATTERN 的所有对象。

**常用通配符：**

- $@：目标的名字
- $^：构造所需文件列表所有所有文件的名字
- $<：构造所需文件列表的第一个文件的名字
- $?：构造所需文件列表中更新过的文件

- = 是最基本的赋值
- := 是覆盖之前的值
- ?= 是如果没有被赋值过就赋予等号后面的值
- += 是添加等号后面的值

**%和*的区别：**

%为Makefile规则通配符，一般用于规则描述；通配符*则不具备上述功能，尤其是在Makefile，当变量定义或者函数调用时，该通配符的展开功能就失效了，即不能正常使用了，此时需要借助 wildcard 函数。二者应用范围不同。

## VPATH 与 vpath

在一些大的工程中，有大量的源文件，我们通常的做法是把这许多的源文件分类，并存放在不同的目录中。所以，当make需要去找寻文件的依赖关系时，你可以在文件前加上路径，但最好的方法是把一个路径告诉make，让make在自动去找。

**VPATH**

Makefile文件中的特殊变量“VPATH”就是完成这个功能的，如果没有指明这个变量，make只会在当前的目录中去找寻依赖文件和目标文件。如果定义了这个变量，那么，make就会在当前目录找不到的情况下，到所指定的目录中去找寻文件 。

VPATH = src:../headers

上面的的定义指定两个目录，“src”和“../headers”，make会按照这个顺序进行搜索。目录由“冒号”分隔。当然，当前目录永远是最高优先搜索的地方。

**vpath**

另一个设置文件搜索路径的方法是使用make的“vpath”关键字（注意，它是全小写的）， 这不是变量，这是一个make的关键字，这和上面提到的那个VPATH变量很类似，但是它更为灵活。它可以指定不同的文件在不同的搜索目录中。这是一个很 灵活的功能。它的使用方法有三种：

1. vpath [pattern] [directories]   为符合模式 pattern 的文件指定搜索目录 directories 。
2. vpath [pattern]                 清除符合模式 pattern 的文件的搜索目录。
3. vpath                           清除所有已被设置好了的文件搜索目录。

vapth使用方法中的 pattern 需要包含“%”字符。“%”的意思是匹配零或若干字符，例如，“%.h”表示所有以“.h”结尾的文件。 pattern 指定了要搜索的文件集， 而 directories 则指定了 pattern 的文件集的搜索的目录。例如：

vpath %.h ../headers

该语句表示，要求make在“../headers”目录下搜索所有以“.h”结尾的文件。

如果某文件在当前目录没有找到的话,我们可以连续地使用vpath语句，以指定不同搜索策略。如果连续的vpath语句中出现了相同的 pattern ，或是被重复了的 pattern，那么，make 会按照 vpath 语句的先后顺序来执行搜索。如：

```
vpath %.c foo
vpath %   blish
vpath %.c bar
```

其表示“.c”结尾的文件，先在“foo”目录，然后是“blish”，最后是“bar”目录。

```
vpath %.c foo:bar
vpath %   blish
```

而上面的语句则表示“.c”结尾的文件，先在“foo”目录，然后是“bar”目录，最后才是“blish”目录。

到此为止，我们学习了几个常用的基本概念，之后我们就开始在较大型的嵌入式工程上实现交叉编译。

# Makefile的在 STM32 中的应用

这次编写 Makefile 的目的是使用交叉编译链编译 STM32F4xx 的代码，使之能在 cortex-M4 上运行。以下我们就围绕这个内容来一步步实现这个过程吧。如果直接跳到这一步的读者，或者还不了解 Makefile 与编译之间的关系，不妨回过头来去看前面两章。

## 交叉编译链

为什么要使用交叉编译链工具呢？在嵌入式开发过程中有宿主机和目标机的角色之分：宿主机是执行编译、链接嵌入式软件的计算机；目标机是运行嵌入式软件的硬件平台。简单地说，就是在一个平台上生成另一个平台上的可执行代码。

为什么要在一个平台上编译另一个平台的执行文件呢？不能像 PC 那样在 PC 上编译 PC 能执行的文件呢？嗯……你觉得能在 STM32 上安装一个编译工具然后编译出一个LED流水灯程序来吗？你有问过他的 Flash 和 RAM 够用吗？

废话少说，那我们用哪个交叉编译链工具呢？

**<a href = "https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm/downloads">gcc-arm-none-eabi</a>** 适用于 Arm Cortex-M & Cortex-R processors (Cortex-M0/M0+/M3/M4/M7/M23/M33, Cortex-R4/R5/R7/R8/R52)系列平台。

如何安装这个软件我就不赘述了，因为安装一个软件比学习一个软件容易得多。

## 我们怎么开始

我们不如联想一下用 keil_v5(MDK5) 编译 STM32 代码时的操作过程，如果没有用过这个软件，也可以联想 IAR 亦或是主机编译类软件的操作方式。 keil_v5 封装了编译链接的细节，使得我们觉得编译就是点一下按键的过程，根据我们上面的学习内容，我们认识到编译一个可执行的目标文件远比我们想象的要复杂得多。

万事开头难，中间难，结尾也难。那我们就从最简单的“点灯”程序开始吧，因为越简单的程序，给我们出错的余地就越少。

我会先用 keil_v5 编译一个 LED 点灯程序，烧录到开发板中进行验证，以确保这个程序是可以正常工作的。

代码的链接与详细说明请看这里：<a href = "https://github.com/laneston/STM32F4xx_LED-Makefile/tree/master/STM32F4xx_LED(keil_v5)">STM32F4xx_LED(keil_v5)</a>

上面的工程已经成功点亮一盏LED，并且以500ms的时间间隔闪烁。

我们可以看到，这个工程中的代码根据功能不同来分类，分别存放在几个文件夹当中。

- STM32F4xx_DSP_StdPeriph_Lib_V1.8.0 [驱动内核与寄存机接口文件]
- Peripheral_BSP[外设驱动]
- User[代码入口]

主要代码分别放在以上3个文件夹当中，而其中一个文件夹 STM32F4xx_DSP_StdPeriph_Lib_V1.8.0 当中又有其他的文件夹。在之前的例子当中，我们的所有文件都是放在同一个文件夹里的， Makefile 文件中的指令也是默认在当前文件夹中寻找相关的依赖文件。所以在当前的工程中，我们需要解决文件的依赖关系，如何让 Makefile 文件像 keil_v5 编译器那样，设置 include 路径后能自动查找.c文件所依赖的文件。

