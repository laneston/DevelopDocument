本编文章是关于交叉编译链的学习笔记当中的核心篇目。文章围绕makefile文件的编写方式，向读者讲述如何在ubuntu平台上用GCC交叉编译链编译出STM32F4xx系列的执行文件（bin)。

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





