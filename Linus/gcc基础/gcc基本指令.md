# 1. gcc基本指令

## 1.1. gcc简介
gcc全称GNU C Compiler是C的编译工具。

C文件的编译过程要经过预处理(Preprocessing)、编译(Compilation)、汇编 (Assembly)和连接(Linking)4个步骤。
每个步骤都有对应的gcc命令。

## 1.2. gcc命令格式
格式gcc [-x] test.* -o test.*
-o: ouput
-x: 处理类型说明符
一般加上-Wall选项，提示错误信息
如：gcc -Wall hello.c -o hello

## 1.3. c文件编译（简单了解）

### 1.3.1. 直接生成.exe可执行文件
**单文件**
gcc -Wall hello.c -o hello
**多文件**
gcc -Wall hello.c main.c -o hello
注：命令行无须添加.h，因为.c文件已经include,编译器会总动寻找

### 1.3.2. 生成.o目标文件
将每个.c文件都编译成.o目标文件（二进制）
gcc -Wall hello.c main.c ... 
注：无需添加-o *.o,因为编译器会自动生成 *.o文件
多文件编译连接中，若只有个别文件有所改动，无需全部重新编译。
只须编译改动的那些文件，重新生成目标文件。

### 1.3.3. 连接外部函数库
静态函数库以.a扩展名
在Linux系统中，标准函数库一般存放在/usr/lib和/lib中
如math library存放在/usr/lib/libm.a中
若程序中include了外部函数库，则需要在命令行中添加库的路径
如：gcc -Wall mian.c  /usr/lib/libm.a -o main
可以用-lm代替/usr/lib/libm.a
-l=/usr/lib/lib
