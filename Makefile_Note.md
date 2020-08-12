本编文章是关于交叉编译链的学习笔记当中的核心篇目。文章围绕makefile文件的编写方式，向读者讲述如何在ubuntu平台上用GCC交叉编译链编译出STM32F4xx系列的执行文件（bin)。

# 什么是Makefile

Makefile文件表述的是文件的依赖关系，告诉编译器怎么去编译和链接当中的文件。如果能掌握Makefile文件的编写方法，就能脱离可视化编译器，使用编译链工具编译出所需的目标文件。

## 编译和链接

编译和链接是获得目标文件的主要方式，在常见的stm32f1xx/stm32f4xx系列的编程任务中，我们可能会用到keil/MDK这个工具，当点击其中的build按键时，我们就能在输出文件夹中获得bin（二进制）或者HEX（十六进制）文件。之后，我们就可以通过MDK自带的烧录工具，使用JLink/STLink将目标文件烧录到芯片的Flash当中，抑或是通过ISP工具烧录到芯片的Flash当中。

在这个过程当中，编译器已经帮我们执行了编译和链接两个步骤。以下我会通过简单的例子说明这一个过程是如何在编译链上体现的。

**编译**

我们在Ubuntu平台上用 **GCC编译器** 编译以下代码：

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

在当前文件夹内，使用以下命令：

```
gcc main.c -o app
```

<img src="https://github.com/laneston/Pictures/blob/master/Post-Makefile/20200812144507.jpg" width="50%" height="50%">

我们可以从上图所示，得到一个名为 app 的目标文件。

输入以下命令即可执行目标文件:

```
./app
```
