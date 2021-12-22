有些信息在存储时，并不需要占用一个完整的字节，而只需占几个或一个二进制位。例如在存放一个开关量时，只有0和1两种状态，用一位二进位即可。

为了节省存储空间，并使处理简便，C语言又提供了一种数据结构，称为“位域”或“位段”。所谓“位域”是把一个字节中的二进位划分为几个不同的区域，并说明每个区域的位数。每个域有一个域名，允许在程序中按域名进行操作。这样就可以把几个不同的对象用一个字节的二进制位域来表示。



### 定义

```c++
struct  位域结构名 
{ 位域列表 };
```

位域列表的形式：类型说明符位域名：位域长度



例如：

```c++
structbs 
{ 
    int a:8; 
    int b:2; 
    int c:6; 
}data;
```

说明：data为bs变量，其中位域a占8位，位域b占2位，位域c占6位。(一个字节8位)



### 位域可以无位域名

这时它只用来作填充或调整位置，无名的位域是不能使用的。



例如：

```c++
typedef   structk 
{ 
    int  a:1 
    int  :2    
    int  b:3 
    int  c:2 
};
```

从以上分析可以看出，位域在本质上就是一种结构类型，不过其成员是按二进位分配的。



### 指针类型变量不能指定所占的位数

这点很好理解，在c语言中，所有的指针类型统一占4字节，不能更改。



### struct变量二进制位数简要说明

例如：定义结构体如下：

```c++
typedefstruct  test
{
    int           a:2;
    unsigned int  b:2;
};
```



对于结构体test来说，a与b成员都是占用两位二进制，但存储的最大值是不一样的。其中：a是有符号型，所以第一位用来存储符号，代表的最大值为二进制“+1”，即1；b为无符号型，代表的最大值为二进制“11”，即3。此结构体占用的大小为4字节,而不是4位额！

> 记住：位域成员不能单独被取sizeof值，且给位域变量成员赋值时，当数值超过变量范围，自动截取不会报错！



使用位域的主要目的是压缩存储，其大致规则为：
1) 如果相邻位域字段的类型相同，且其位宽之和小于类型的sizeof大小，则后面的字段将紧邻前一个字段存储，直到不能容纳为止；
2) 如果相邻位域字段的类型相同，但其位宽之和大于类型的sizeof大小，则后面的字段将从新的存储单元开始，其偏移量为其类型大小的整数倍；

示例1：

```c++
structBF1
{
    char f1 : 3;
    char f2 : 4;
    char f3 : 5;
};
```


其内存布局为：

```c++
|__f1___|____f2___ |__|____f3______|______|
|__|__|__|__|__|__|__|__|__|__|__|__|__|__|__|__|
```

位域类型为char，第1个字节仅能容纳下f1和f2，所以f2被压缩到第1个字节中，而f3只
能从下一个字节开始。因此sizeof(BF1)的结果为2。




3) 如果相邻的位域字段的类型不同，则各编译器的具体实现有差异，VC6采取不压缩方
式，Dev-C++采取压缩方式；

示例2：

```c++
structBF2
{	
    char f1 :  3;
    short f2 : 4;
    char f3 :  5;
};
```

由于相邻位域类型不同，在VC6中其sizeof为6，在Dev-C++中为2。




4) 如果位域字段之间穿插着非位域字段，则不进行压缩；

示例3：

```c++
structBF3
{
    char f1 : 3;
    char f2;
    char f3 : 5;
};
```

非位域字段穿插在其中，不会产生压缩，在VC6和Dev-C++中得到的大小均为3。



举这些例子是为了说明一下，定义位域的话，最好是把所以有位域放在一起，这样可以节省空间，另外也是为了强调一下位结构体的内存分配方式，按定义的先后顺序来分配！




5) 整个结构体的总大小为最宽基本类型成员大小的整数倍！。——永远成立！

还是让我们来看看例子，你会感到不可思议： 

```c++
struct  mybitfields
{
    unsigned short a : 4;
    unsigned short b : 5;
    unsigned short c : 7;
} test;
=> sizeof(test) ==2;

struct mybitfields
{
    unsigned char a : 4;
    unsigned char b : 5;
    unsigned char c : 7;
} test;
=> sizeof(test) ==3;

struct mybitfields
{
    unsigned char a : 4;
    unsigned short b : 5;
    unsigned char c : 7;
} test;
=> sizeof(test) ==6;

struct mybitfields
{
    unsigned short a : 4;
    unsigned char b : 5;
    unsigned char c : 7;
} test;
=> sizeof(test) ==4;

struct mybitfields
{
    unsigned char a : 4;
    unsigned char b : 5;
    unsigned short c : 7;
} test;
=> sizeof(test) ==4;

struct mybitfields
{
    unsigned char a : 4;
    unsigned int b : 5;
    unsigned short c : 7;
} test;
=> sizeof(test) ==12;
```



### 常用内置类型的字节数

对于32位编译器来说：

char：   1个字节

指针变量:  4个字节（32位的寻址空间是2^32,即32个bit，也就是4个字节。同理64位编译器）

short int : 2个字节

int：    4个字节

unsigned int :4个字节

float:    4个字节

double:   8个字节

long:    4个字节

long long:  8个字节

unsigned long:4个字节
