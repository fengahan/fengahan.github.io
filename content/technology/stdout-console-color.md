---
title: "如何在控制台 输出带有颜色 的字体（转载）"
date: 2019-06-18T12:51:18+02:11
description: "之前这个地方看到过但是没在意,今天打算把自己写的框架 在控制台命令下输出不同的颜色所以就找到了这篇帖子 主要怕 到时候原贴不在了,所以转到博客上来"
categories: [go, php,操作系统]
---

先在终端运行下看下效果
```shell
echo  -e "\033[1;31mI ♡  You \e[0m"
```
请注意，引号内的\e等同于\033；\033、\x1b和\e效果是一样，对应键盘左上角Esc键对应的ASCII码（8进制）；
通用的控制文本颜色的转义序列格式如下：

CSI n1 [;n2 [;…]] m
其中CSI全称为“控制序列引导器”（Control Sequence Introducer/Initiator），也就是上述示例中的"\033["（其中\033是你键盘左上角Esc键对应的ascii码（八进制））；

n1、n2等表示SGR参数（下面会列出一些常用的SGR参数），用于控制颜色、粗体、斜体、闪烁等文本输出格式；m表示转义序列结束。

颜色参数
常用颜色
格式：\033[显示方式;前景色;背景色m

说明：
前景色            背景色           颜色

| 前景色     | 背景色       |  颜色|
|----------|-------------|-------------|
30 |                40    |           黑色 | 
31  |                41    |           红色 | 
32 |                42    |           绿色 | 
33 |                43    |           黃色 | 
34  |               44    |           蓝色 | 
35 |                45    |           紫红色 | 
36  |               46    |           青蓝色 | 
37  |               47    |           白色 | 


| 序号    | 释义       |
|----------|-------------|
0 |                终端默认设置  | 
1 |                高亮显示      | 
4 |               使用下划线    | 
5 |                闪烁        |  
7 |                反白显示     | 
8 |                不可见       | 

例子：
\033[1;31;40m    <!--1-高亮显示 31-前景色红色  40-背景色黑色-->
\033[0m          <!--采用终端默认设置，即取消颜色设置-->
输入图片说明

RGB颜色
输入图片说明

使用方式：

ESC[ … 38;2;<r>;<g>;<b> … m Select RGB foreground color
ESC[ … 48;2;<r>;<g>;<b> … m Select RGB background color
特别注意：此处用法需从38开始设置, 代表前景色，48 代表后景色；2 代表RGB模式，后面三位为RGB数值。前后仍可使用SRG参数

例子：

```sh
echo  -e "\033[1;38;2;255;85;85mI ♡  You \e[0m"
```
附：经测试，在不是用SRG参数的情况下，可将;替换为:但在使用SRG参数时无效。 即：echo -e "\033[38:2:255:85:85mI ♡ You \e[0m"

256色模式
输入图片说明

使用方式：

ESC[ … 38;5;<n> … m Select foreground color
ESC[ … 48;5;<n> … m Select background color

例子：

```sh
echo  -e "\033[1;38;5;9mI ♡  You \e[0m"
```

SRG控制参数

| 序号    | 释义       |
|----------|-------------|
| 0  |   关闭所有格式，还原为初始状态 | 
| 1  |    粗体/高亮显示            | 
| 2  |     模糊（※）              | 
| 3  |     斜体（※）              | 
| 4  |     下划线（单线）          | 
| 5  |     闪烁（慢）             | 
| 6  |     闪烁（快）（※）           
| 7  |     交换背景色与前景色              | 
| 8  |     隐藏（伸手不见五指，啥也看不见）（※） | 

（1）其中含有（※）标注的编码表示不是所有的终端仿真器都支持，只有少数仿真器支持。

（2）多个SGR参数可以组合使用，例如：echo -e "\x1b[31;4mRed Underline Text\e[0m"输出红色下划线字体“Red Underline Text”。

（3）更多参数信息请参考“ANSI escape code”。

各语言下的控制台颜色输出
PHP
// hello.php
```php
<?php
echo "\033[1;38;5;9mI ♡  You \e[0m\n";
```
请注意此处使用的是"而不是单引号，因为颜色代码需要转义，正如echo的-e一样。

C
// hello.c
```c
#include <stdio.h>
int main() {
    printf("\033[1;31mI ♡  You \e[0m\n");
}
```
编译：gcc hello.c

运行：./a.out

C++
// hello.cpp
```c
#include <iostream>
int main() {
    std::cout << "\033[1;31mI ♡  You \e[0m" << std::endl;
}
```
编译：g++ hello.cpp

运行：./a.out

Java
// hello.java
```java
class hello {
    public static void main(String[] args) {
       System.out.println("\033[1;31mI ♡  You \033[0m");
    }
}
```

注：Java中不能识别\e和\0x1b，仅可使用\033。

编译：javac hello.java

运行：java hello

Python
hello.py
```python
print "\033[1;31mI ♡  You \x1b[0m"

```

注：python（v2.6.5）中不能识别\e，可以使用\033和\x1b。

非常地道的贴上原文地址：https://phperzh.com/wr?u=https%3a%2f%2fmy.oschina.net%2fdingdayu%2fblog%2f1537064