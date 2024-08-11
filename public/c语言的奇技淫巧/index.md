# C语言的奇技淫巧


<!--more-->

# C语言中的奇技淫巧



## 01. 宏定义用`do{}while(0)`

如果定义的宏函数后面有多条语句，使用这样的方式会有问题：

```c
#define FUNC()  func1(); func2()
if(bRunF)
    FUNC();
```

展开宏定义后会变成：

```c
if(bRunF)
    func1();
    func2();
```

逻辑就不对了。可以用这一的方式解决，非常好用：

```c
#define FUNC()  do{func1(); func2();}while(0)
```

## 02. 数组的初始化

假如给arr的第2~6元素初始化为5，也许你会

```c
int arr[10] = {0, 5, 5, 5, 5, 5, 0, 0, 0, 0};
```

现在告诉你C99可以这样：

```c
int arr[10] = {[1... 5] = 5};
```

## 03. 数组的访问

你想取数组的第6个元素（下标为5），教科书教你这样做：

```c
int arr[10] = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
int n1 = arr[5];
int n2 = *(arr+5);
```

其实你可以：

```c
int arr[10] = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
int n = 5[arr];
```

也不会有错，实际上
arr[5]对应`*(arr+5)`，而5[arr]对应`*(5+arr)`，没多大区别。

## 04. 结构体的初始化

结构体的初始化，传统的做法是：

```c
typedef struct
{
    int a;
    int x;
    int y;
    int z;
    char b;
    short c;
}S;
S s = {100, 0, 0, 0, 'A', 0x12);
```

对于C99，其实你可以：

```c
typedef struct
{
    int a;
    int x;
    int y;
    int z;
    char b;
    short c;
}S;
S s = {
            .a = 100, 
            .b = 'A', 
            .c = 0x12
        };
```

## 05. 用`include`的方式初始化大数组

```c
double array[SIZE][SIZE] = {
    #include "float_values.txt"
}
```

## 06. Debug时输出文件名、函数名、行号等

```c
#define DEBUG_INFO() fprintf(stderr,"[DEBUG]%s:%d %s\n", __FILE__, __LINE__, __FUNCTION__);
```

## 07. C语言有`-->`“趋向于…”操作符？

```c
int main(void)
{
        int n = 10; 
        while(n --> 0 ) // n goes to 0
        { 
                printf("%d ", n);
        }
        printf("\n");
}
```

实际上C语言没有这个`-->`操作符，是`--`和`>`的组合而已

```c
        while( n--  >  0 )
```

## 08. 获得任意类型数组的元素数目

```
#define NUM_OF(arr) (sizeof (arr) / sizeof (*arr))
```

## 09. 判断运行环境的大小端

Linux有以下代码：

```c
    static union { 
        char c[4]; 
        unsigned long l; 
    } endian_test = { { 'l', '?', '?', 'b' } };
    #define ENDIANNESS ((char)endian_test.l)

    printf("ENDIANNESS: %c\n", ENDIANNESS);
```

## 10. 编译时做条件检查

Linux Kernel有以下代码

```c
/* Force a compilation error if condition is true */
#define BUILD_BUG_ON(condition) ((void)sizeof(char[1 - 2*!!(condition)]))
```

例如，在某些平台为了防止内存对齐问题，检查一个结构体或者一个数组的大小是否为8的倍数。

```c
BUILD_BUG_ON((sizeof(struct mystruct) % 8) != 0);
```

除了这个，还有

```c
#define BUILD_BUG_ON_ZERO(e)  (sizeof(struct{int : -!!(e);}))
#define BUILD_BUG_ON_NULL(e)  ((void*)sizeof(struct{int : -!!(e);}))
#define BUILD_BUG_ON(condition)  ((void)BUILD_BUG_ON_ZERO(condition))
#define MAYBE_BUILD_BUG_ON(condition)  ((void)sizeof(char[1 - 2 * !!(condition)]))
```

## 11. 用异或运算实现数据交换

交换俩变量数据，一般做法是：

```c
// 方法1
temp = a;
a = b;
b = temp;

// 方法2
a=a+b;
b=a-b;
a=a-b；
```

方法1需要第三个变量，方法二存在数据溢出可能，可以尝试下以下方法：

```c
a = a ^ b;
b = a ^ b;
a = a ^ b;
```

## 12. 判断语句中把`const`数值放在前面

通常条件语句写成

```c
if(n == 0){ /*...*/ }
```

但是，有可能手误写成

```c
if(n = 0){ /*...*/ }
```

这种错误只有机器在运行时候知道，而人不一定能发现这种bug。
把数值放在前面就不怕了，`==`写成`=`，编译器就知道

```c
if(0 == n){ /*...*/ }
```

## 13. 用冒号表达式替代`if...else...`语句

这个用法应该很普遍了，不算什么特别的技巧了。

```c
if(y < 0)
{
    x = 10;
}
else
{
    x = 20;
}
```

可以改成以下一行代码即可

```c
x = (y < 0) ? 10 : 20;
```

## 14. 判断一个整数是否为2的幂

也许你会不断地将这个数除以2，除到底，然而Linux kernel有个巧妙的办法：

```c
#define is_power_of_2(n) ((n) != 0 && ((n) & ((n) - 1)) == 0)
```

`((n) & ((n) - 1)) == 0`这个不理解？那先想想2的X次方的值的二进制是怎样的。

## 15. 静态链表

直接看代码

```c
    struct mylist {
        int a;
        struct mylist* next;
    };
    #define cons(x, y) (struct mylist[]){{x, y}}
    struct mylist *list = cons(1, cons(2, cons(3, NULL)));
    struct mylist *p = list;
    while(p != 0) {
        printf("%d\n", p->a);
        p = p -> next;
    };
```

## 16. 柔性数组

```c
#include <stdlib.h>
#include <string.h>

struct line
{
    int length;
    char contents[0];
};
struct line *thisline = (struct line *) malloc (sizeof (struct line) + this_length);
thisline->length = this_length;

struct f1 { int x; int y[]; } f1 = { 1, { 2, 3, 4 } }; 
struct f2 { struct f1 f1; int data[3]; } f2 = { { 1 }, { 2, 3, 4 } };
```

详见[6.18 Arrays of Length Zero](https://gcc.gnu.org/onlinedocs/gcc/Zero-Length.html)

## 17. 数组之间直接赋值

```c
int a[10] = {0,1,2,3,4,5,6,7,8,9};
int b[10] = {0};
b = a;
```

这样是非法的，但是你可以放数组穿个马甲：

```c
typedef struct
{
    int n[10];
}S;
S a = {{0,1,2,3,4,5,6,7,8,9}};
S b = {0};
b = a;
```

## 18. `#include`的不一定是要`.h`文件

`#include`后面跟的可以是任意后缀的，但文件内容一定要是合法的。例如

```c
#include "test.fxxk"
```

## 19. 自动获取变量类型

```c
#define var(left, right) __typeof__(right) left = (right)

var(s, 1LL); // 相当于 long long s = 1LL;
```

是不是有点像C++ 11的auto类型？

## 20. 宏定义函数`MIN(x,y)`的终极做法

```c
#define MIN(x, y)   x < y? x : y    // 这样给0分

#define MIN(x, y)   (x < y? x : y)  // 这样给50分
// 不信你试试这个
int n = 3 * MIN(3, 4 < 5 ? 4 : 5);

#define MIN(x, y)   ((x) < (y)? (x) : (y))  // 这个给90分
// 不信你试试这个
double xx = 1.0;
double yy = MIN(xx++, 1.5);
printf("xx=%f, yy=%f\n",xx,yy);

// 以下放大招了，看看GNU的
#define MIN(A,B)	({ __typeof__(A) __a = (A); __typeof__(B) __b = (B); __a < __b ? __a : __b; })
double xx = 1.0;
double yy = MIN(xx++, 1.5);
printf("xx=%f, yy=%f\n",xx,yy);
```

## 21. 行控制`#line`

也许你知道用`__LINE__`可以输出行号，然而你试下这个：

```c
 #line 12345 "abcdefg.xxxxx"    
 printf("%s line: %d\n", __FILE__, __LINE__);    printf("%s line: %d\n", __FILE__, __LINE__);
```

不单止行号被改了，文件名也被改了，是不是我们可以用这个干点啥……想想？

## 22. C和C++代码混合编译

在C的头文件上面

```c
#ifdef __cplusplus
extern "C" {
#endif
```

然后再头文件下面

```c
#ifdef __cplusplus
}
#endif
```

## 23. 用查表法实现hex2str

直接上代码

```c
void hex2str(const unsigned char* hex, int size, char* str)
{
    char char_arr[17] = "0123456789ABCDEF";
    for(int i = 0; i < size; i++)
    {
        str[3*i] = char_arr[hex[i]>>4];
        str[3*i+1] = char_arr[hex[i]&0x0F];
        str[3*i+2] = ' ';
    }
}
```

## 24. 用sprintf实现hex2str

直接上代码

```c
void hex2str(const unsigned char* hex, int size, char* str)
{
    for(int i = 0; i < size; i++)
    {
        sprintf(&str[3*i], "%02X ", hex[i]);
    }    
}
```

## 25. 将变量名变字符串

如果想打印一个变量名和它的值，也许会这样：

```c
unsigned int program_flag = 0xAABBCCDD;
printf("program_flag: 0x%08X\n", program_flag);
```

对于你有很多这样的变量要打印，建议你做个宏函数：

```c
#define PRINT_HEX_VAR(var)  printf("%s: 0x%08X\n", #var, var);
unsigned int program_flag = 0xAABBCCDD;
PRINT_HEX_VAR(program_flag);
```

## 26. 获取结构体元素的偏移

```c
#define offsetof(type, member) ( (size_t)&((type*)0->menber) )

typedef struct
{
    char a;
    int b;
 }S;
 offsetof(S, b);
```

## 27. 根据结构体成员获取结构体变量指针

```c
#define offsetof(TYPE, MEMBER) ((size_t) &((TYPE *)0)->MEMBER)
/**
 * container_of - cast a member of a structure out to the containing structure
 * @ptr:        the pointer to the member.
 * @type:       the type of the container struct this is embedded in.
 * @member:     the name of the member within the struct.
 *
 */
#define container_of(ptr, type, member) ({                      \
    const typeof( ((type *)0)->member ) *__mptr = (ptr);    \ 
    (type *)( (char *)__mptr - offsetof(type,member) );}) 
```

这个怎么玩？看看链表

```c
struct list_head {
    struct list_head *next；
    struct list_head  *prev;
};

struct ipstore{
    unsigned long time;
    __u32 addr[4];
    struct list_head list;
};

container_of(ist1->list, struct ipstore, list)
```

## 28. `scanf`高级玩法

```c
scanf(“%[^,]”, a); // This doesn’t scrap the comma
scanf(“%[^,],”,a); // This one scraps the comma
scanf(“%[^\n]\n”, a); // It will read until you meet  '\n', then trashes the '\n'
scanf(“%*s %s”, last_name); // last_name is a variable
```

这是啥意思，正则表达式先了解下？然后自己试试，理解会更深入。

## 29. 两个数相加可以不用`+`号？

```c
int Add(int x, int y)
{
      if (y == 0)
            return x;
      else
            return Add( x ^ y, (x & y) << 1);
}
```

## 30. 调试的时候打印数组

你是不是曾经为打印数组而烦恼，每次都要将元素一个个取出来？

```c
#define ARR_SIZE(arr)               (sizeof(arr)/sizeof(*arr))
#define PRINT_DIGIT_ARR(arr)    do{\
                                               printf("%s: ", #arr); \
                                               for(int i=0; i < ARR_SIZE(arr); i++) \
                                                   printf("%d ", arr[i]);\
                                               printf("\n");\
                                            }while(0)
                                
int arr[10] = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
PRINT_DIGIT_ARR(arr);
```

## 31. 感受下这个`0x5F3759DF`

```c
float Q_rsqrt( float number )
{
    long i;
    float x2, y;
    const float threehalfs = 1.5F;

    x2 = number * 0.5F;
    y  = number;
    i  = * ( long * ) &y;                       // evil floating point bit level hacking
    i  = 0x5f3759df - ( i >> 1 );               // what the fuck?
    y  = * ( float * ) &i;
    y  = y * ( threehalfs - ( x2 * y * y ) );   // 1st iteration
    //      y  = y * ( threehalfs - ( x2 * y * y ) );   // 2nd iteration, this can be removed
 
    return y;
}
```

## 32. `switch-case`的特殊玩法

直接看代码

```c
void send(uint8* to, uint8 from, uint16 count)
{
    uint16 n = (count + 7) / 8; 
    switch (count % 8) 
    { 
        case 0: do { *to = *from++; 
        case 7: *to = *from++; 
        case 6: *to = *from++; 
        case 5: *to = *from++; 
        case 4: *to = *from++; 
        case 3: *to = *from++; 
        case 2: *to = *from++; 
        case 1: *to = *from++; 
    } while (--n > 0); 
}
```

实际上它是

```c
void send(uint8* to, uint8 from, uint16 count)
{
    do 
    { 
        *to = *from++; 
    } while (--count > 0); 
}
```

使用最上面的`switch-case`的形式大大提高了运行效率。
理解不了？汇编看看。
还是理解不了？那就网上自行搜索“Duff’s Device”

## 33. 防止头文件重复包含导致问题

这个用法很常见了，而且非常有用

```c
// xxx.h
#ifndef _XXX_H_
#define _XXX_H_

//  Header file contents...

#endif
```

当然，如果你的编译器支持的话，也可以

```c
// xxx.h
#pragma once

//  Header file contents...
```

不过为了更好的兼容性，我建议你用第一种方法。

## 34. 2的N次幂ROUNDUP

```c
#define ROUNDUP(a, size) (((a) & ((size)-1)) ? (1+((a) | ((size)-1))) : (a))
```

其中，size是2的整数次幂，而

1. `a & (2^n-1)`检查a的低位是否有值
2. `a | (2^n - 1)` 将a的低n位赋值为1
3. `1 + a | (2^n -1)` 为a最近的下一个2^n倍值

```c
ROUNDUP(10, 8);     // 结果为16
ROUNDUP(10, 16);   // 结果为16
ROUNDUP(10, 32);   // 结果为32
ROUNDUP(16, 16);   // 结果为16
```

有什么用？申请内存的时候可以按某字节对齐，减少内存碎片啊。

## 35. 某整数的ROUNDUP

```c
#define VAL_ROUNDUP(size, val_size)     (((size)+val_size-1)/val_size*val_size)
```

这个不是按2的次幂ROUNDUP的，而是按某个整数的倍数ROUNDUP，例如

```c
VAL_ROUNDUP(10, 8);     // 结果为16
VAL_ROUNDUP(16, 8);   // 结果为16
VAL_ROUNDUP(8, 10);   // 结果为10
VAL_ROUNDUP(16, 16);   // 结果为16
```

这个又有什么用？EEPROM或者Flash的page大小对齐的时候就非常有意义。

## 36. 万能的`void*`

想想，`memcpy`函数为什么要用`void*`？

```c
void *memcpy(void *dest, const void *src, size_t n);
```

因为，它不关心你传什么类型的指针过来，我`void`统统都接纳。

> 无为而无不为。

再看看这个：

```c
void *p1;
int *p2;

p1 = p2; // 这个没问题
p2 = p1; // 这个是错的
```

> 因为“空类型”可以包容“有类型”，而“有类型”则不能包容“空类型”。

所以，适可而止，不要滥用哦。

## 37. `sizeof(空)`

```c
    typedef struct 
    {
    }StructNull;
    
    sizeof(void);
    sizeof(StructNull);
```

正所谓

> 空即是色，色即是空。

在C语言上，`sizeof(void);` 的值为1；而`sizeof(StructNull);`为0。
C++的情况请自行验证，别瞎猜，哈哈哈。

## 38. 布尔变量的判断

正确的做法：

```c
if(bValue)
```

以下是瞎搞

```c
if(bValue == TRUE)
```

为啥？布尔类型中只有两个值：假和真。
请问：假是什么，真又是什么？

假是0，而真是非0。那么非0是什么？`-1,1,2，……`除了0的一切。

所以再想想以下代码中的两个叹号是否可以去掉？

```c
#define BUILD_BUG_ON(condition) ((void)sizeof(char[1 - 2*!!(condition)]))
```

## 39. 感受下`##`的用法

我们知道`##`是用来连接字符的，看看RTX怎么用，感受一下：

```c
/// Create a Thread Definition with function, priority, and stack requirements.
/// \param         name         name of the thread function.
/// \param         priority     initial priority of the thread function.
/// \param         instances    number of possible thread instances.
/// \param         stacksz      stack size (in bytes) requirements for the thread function.
///       macro body is implementation specific in every CMSIS-RTOS.
                         // define the object
#define osThreadDef(name, priority, instances, stacksz)  \
const osThreadDef_t os_thread_def_##name = \
{ (name), (priority), (instances), (stacksz)  }

/// Access a Thread definition.
/// \param         name          name of the thread definition object.
///       macro body is implementation specific in every CMSIS-RTOS.
#define osThread(name)  \
&os_thread_def_##name
```

代码比较简单，我就不解释了，自行思考下。

## 40. 传值和传址

看两个例子：

```c
void swap(int a; int b)
{
    int t = 0;
    t = a;
    a = b;
    b = t;
}

int x = 100;
int y = 200;
swap(x, y);
```

请问，这个`swap`可以交换`x, y`的值吗？

再看看一个常见的面试题：

```c
void GetMemory(char *p)
{
    p = (char *)malloc(100);
}

void Test(void)
{
    char *str = NULL;
    GetMemory(str);
    strcpy(str, "hello world");
    printf(str);
}
```

这个程序能输出“hello world”吗？

此处没有答案，为了加深理解，建议感兴趣的朋友请自行动手验证和思考。

## 41. 形参到底传值好还是传址好

接着上一条，我们从另一个角度看。函数定义一个结构体类型形参，传值好还是传址好？

```c
void func1(struct tStructType param)
{
}
void func2(struct tStructType* param)
{
}
```

实际上两种方式都行，但你要明白形参实际上就是一个临时变量，不管传值还是传址都有一个复制给临时变量的过程。这个仿真汇编看看就知道了。
很明显，如果tStructType这个类型占用空间很大，那么肯定用`tStructType*`比较合算。

## 42. bit翻转的几个方法

bit翻转是从MSB->LSB到LSB->MSB, 所有的Bit都必须反转。例如：

```c
1010 0001 => 1000 0101
```

**1. 运算实现32位bit翻转**

```c
unsigned int
reverse(register unsigned int x)
{
    x = (((x & 0xaaaaaaaa) >> 1) | ((x & 0x55555555) << 1));
    x = (((x & 0xcccccccc) >> 2) | ((x & 0x33333333) << 2));
    x = (((x & 0xf0f0f0f0) >> 4) | ((x & 0x0f0f0f0f) << 4));
    x = (((x & 0xff00ff00) >> 8) | ((x & 0x00ff00ff) << 8));
    return((x >> 16) | (x << 16));
}
```

**2. 查表法bit翻转**

```c
static const unsigned char BitReverseTable256[] = 
{
  0x00, 0x80, 0x40, 0xC0, 0x20, 0xA0, 0x60, 0xE0, 0x10, 0x90, 0x50, 0xD0, 0x30, 0xB0, 0x70, 0xF0, 
  0x08, 0x88, 0x48, 0xC8, 0x28, 0xA8, 0x68, 0xE8, 0x18, 0x98, 0x58, 0xD8, 0x38, 0xB8, 0x78, 0xF8, 
  0x04, 0x84, 0x44, 0xC4, 0x24, 0xA4, 0x64, 0xE4, 0x14, 0x94, 0x54, 0xD4, 0x34, 0xB4, 0x74, 0xF4, 
  0x0C, 0x8C, 0x4C, 0xCC, 0x2C, 0xAC, 0x6C, 0xEC, 0x1C, 0x9C, 0x5C, 0xDC, 0x3C, 0xBC, 0x7C, 0xFC, 
  0x02, 0x82, 0x42, 0xC2, 0x22, 0xA2, 0x62, 0xE2, 0x12, 0x92, 0x52, 0xD2, 0x32, 0xB2, 0x72, 0xF2, 
  0x0A, 0x8A, 0x4A, 0xCA, 0x2A, 0xAA, 0x6A, 0xEA, 0x1A, 0x9A, 0x5A, 0xDA, 0x3A, 0xBA, 0x7A, 0xFA,
  0x06, 0x86, 0x46, 0xC6, 0x26, 0xA6, 0x66, 0xE6, 0x16, 0x96, 0x56, 0xD6, 0x36, 0xB6, 0x76, 0xF6, 
  0x0E, 0x8E, 0x4E, 0xCE, 0x2E, 0xAE, 0x6E, 0xEE, 0x1E, 0x9E, 0x5E, 0xDE, 0x3E, 0xBE, 0x7E, 0xFE,
  0x01, 0x81, 0x41, 0xC1, 0x21, 0xA1, 0x61, 0xE1, 0x11, 0x91, 0x51, 0xD1, 0x31, 0xB1, 0x71, 0xF1,
  0x09, 0x89, 0x49, 0xC9, 0x29, 0xA9, 0x69, 0xE9, 0x19, 0x99, 0x59, 0xD9, 0x39, 0xB9, 0x79, 0xF9, 
  0x05, 0x85, 0x45, 0xC5, 0x25, 0xA5, 0x65, 0xE5, 0x15, 0x95, 0x55, 0xD5, 0x35, 0xB5, 0x75, 0xF5,
  0x0D, 0x8D, 0x4D, 0xCD, 0x2D, 0xAD, 0x6D, 0xED, 0x1D, 0x9D, 0x5D, 0xDD, 0x3D, 0xBD, 0x7D, 0xFD,
  0x03, 0x83, 0x43, 0xC3, 0x23, 0xA3, 0x63, 0xE3, 0x13, 0x93, 0x53, 0xD3, 0x33, 0xB3, 0x73, 0xF3, 
  0x0B, 0x8B, 0x4B, 0xCB, 0x2B, 0xAB, 0x6B, 0xEB, 0x1B, 0x9B, 0x5B, 0xDB, 0x3B, 0xBB, 0x7B, 0xFB,
  0x07, 0x87, 0x47, 0xC7, 0x27, 0xA7, 0x67, 0xE7, 0x17, 0x97, 0x57, 0xD7, 0x37, 0xB7, 0x77, 0xF7, 
  0x0F, 0x8F, 0x4F, 0xCF, 0x2F, 0xAF, 0x6F, 0xEF, 0x1F, 0x9F, 0x5F, 0xDF, 0x3F, 0xBF, 0x7F, 0xFF
};

unsigned int v; // reverse 32-bit value, 8 bits at time
unsigned int c; // c will get v reversed

// Option 1:
c = (BitReverseTable256[v & 0xff] << 24) | 
    (BitReverseTable256[(v >> 8) & 0xff] << 16) | 
    (BitReverseTable256[(v >> 16) & 0xff] << 8) |
    (BitReverseTable256[(v >> 24) & 0xff]);

// Option 2:
unsigned char * p = (unsigned char *) &v;
unsigned char * q = (unsigned char *) &c;
q[3] = BitReverseTable256[p[0]]; 
q[2] = BitReverseTable256[p[1]]; 
q[1] = BitReverseTable256[p[2]]; 
q[0] = BitReverseTable256[p[3]];
```

## 43. BOOL判断

面试题：如何判断一个bool变量
如果你写成`if(b_flag == TRUE)`可能会给你0分。为什么？
因为非假即真，假是0，即非0即真。
可以写成`if(b_flag)`或者`if(!b_flag)`
如果是函数呢？
例如：

```c
int flag = 0;

BOOL is_flag_true(void)
{
    return flag;
}
```

这样可以吗？也许可以，也许不可以。
如果`BOOL`是一个`unsigned char`怎么办？
那就这样咯：

```c
BOOL is_flag_true(void)
{
    return !!flag;
}
```

## 44. goto的使用

看到goto，先别慌，也忍着别吵。我们不推荐使用goto，但goto确实有它的妙用。
不详细解释了，看看以下例子代码感受下吧：

```c
int func(void)
{
    int* p1 = (int*) malloc (100*4);
    if(NULL == p1) goto free1;

    int* p2 = (int*) malloc (200*4);
    if(NULL == p2) goto free2;
 
    int* p3 = (int*) malloc (300*4);
    if(NULL == p3) goto free3;

    return 0; // return to system

    free3:
        free (p2);
    free2:
        free (p1);
    free1:
        return -1 ;
}
```

## 45. 类型定义

这个有点老套了，但是很有用。

```c
typedef uint8 unsigned char;
typedef uint16 unsigned short;
typedef uint32 unsigned int;
```

用`uint8`等来定义变量比`unsigned char`这样的好多了，方便平台移植，特别是不用位数的单片机，这个很重要。

## 46. 输出预编译信息

当你想输出预编译过程中的某些信息，可以用`#pragma message("message contents")`

## 47. 输出指针地址

当你想输出指针地址的时候，往往想到的是

```c
int *p = &n;
printf("p: %X"，(unsigned int)p);
```

但你还可以

```c
int *p = &n;
printf("p: %p"，p);
```

## 48. 数组元素赋值

```c
uint8 data1[8] = {1,2,3,4,5,6,7,8};
uint8 data2[8];

memcpy(data2, data1, 8);
 // or
 data2[0] = data1[0];
 data2[1] = data1[1];
 data2[2] = data1[2];
 data2[3] = data1[3];
 data2[4] = data1[4];
 data2[5] = data1[5];
 data2[6] = data1[6];
 data2[7] = data1[7];
```

你也可以

```c
typedef union
{
    uint64 nData;
    uint8 cData[8];
}uData;
uData data1, data2;
data2.nData = data1.nData;
```

还有第三种办法，详见“17. 数组之间直接赋值”
顺便思考下并验证以下方式是否可行？（想知道答案一定要亲自试试啊，别说我坑你。）

```c
uint8 data1[8] = {1,2,3,4,5,6,7,8};
uint8 data2[8];
*(uint64*) data2 = *(uint64*) data1;
```

## 49. `likely()`与`unlikely`

Linux内核中有两个这样的东西：`likely()`与`unlikely`

```c
#define likely(x) __builtin_expect(!!(x), 1)
#define unlikely(x) __builtin_expect(!!(x), 0)
```

我们可以根据高概率发生的情况放在`if`分支，低概率的放在`else`分支，以提高程序运行效率。

```c
if(likely(XXX))
{/*...*/}
else
{/*...*/}
```

或者

```c
if(unlikely(XXX))
{/*...*/}
else
{/*...*/}
```

## 50. 解放`if/else`和`switch/case`

对于多分支的程序设计，很多人通常会这样做：

```c
if(XXX_VAL_1 == nVal)
{/*...*/}
else if(XXX_VAL_2 == nVal)
{/*...*/}
else if(XXX_VAL_3 == nVal)
{/*...*/}
// ...
```

或者

```c
switch(nVal)
{
    case XXX_VAL_1:
	/*...*/
	break;
    case XXX_VAL_2:
	/*...*/
	break;
    case XXX_VAL_3:
	/*...*/
	break;
	// ...
}
```

几个分支还好，如果有几十个呢？尝试下这个：

```c
typedef void (pFunc*)(void);
typedef struct
{
    uint32 val;
    pFunc func;
}tValFunc;

tValFunc valfunc_tb[] = 
{
    {XXX_VAL_1, func1},
    {XXX_VAL_2, func2},
    {XXX_VAL_3, func3},
    // ...
};
```

## 参考连接

- [C语言的奇技淫巧（1-50）_技下万1dd47c-CSDN博客](https://blog.csdn.net/lianyunyouyou/article/details/117534039)


---

> 作者: map[avatar:<nil> email:<nil> link:<nil> name:<nil>]  
> URL: https://blog-12x.pages.dev/c%E8%AF%AD%E8%A8%80%E7%9A%84%E5%A5%87%E6%8A%80%E6%B7%AB%E5%B7%A7/  

