
```
objects := $(patsubst %.c,%.o,$(wildcard *.c))

app:  $(objects)
	gcc -o $@ $^
```

值得注意的是，上面的命令行中，我们用到了的是：

```
    gcc -o $@ $^
```
而不是：
```
    gcc -o $@ $<
```
虽然按常规的理解，$< 代表的是第一个依赖的文件，而目标文件 app 后只接了一个变量 $(objects)，按照某些人的理解,$(objects) 需看作是一个整体，这个“依赖文件”理应是变量 $(objects)，遗憾的是，里面包含了：main.o addition.o subtraction.o division.o multiplication.o 几个文件。如果用符号 $< 指代的是第一个依赖文件 main.o 所以不能用 $< 



