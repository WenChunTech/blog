# C语言常用函数


<!--more-->

# 一、string.h中字符串处理函数

在头文件<string.h>中定义了两组字符串函数。第一组函数的名字以str开头；第二组函数的名字以mem开头。
只有函数memmove对重叠对象间的拷贝进行了定义，而其他函数都未定义。比较类函数将其变量视为unsigned char类型的数组。

#### 1 strcpy

`#include <string.h>`
char *strcpy(char *str1, const char *str2);
把字符串str2(包括'\0')拷贝到字符串str1当中，并返回str1。

#### 2 strncpy

`#include <string.h>`
char *strncpy(char *str1, const char *str2, size_t count);
把字符串str2中最多count个字符拷贝到字符串str1中，并返回str1。如果str2中少于count个字符，那么就用'\0'来填充，直到满足count个字符为止。

#### 3 strcat

`#include <string.h>`
char *strcat(char *str1, const char *str2);
把str2(包括'\0')拷贝到str1的尾部(连接)，并返回str1。其中终止原str1的'\0'被str2的第一个字符覆盖。

#### 4 strncat

`#include <string.h>`
char *strncat(char *str1, const char *str2, size_t count);
把str2中最多count个字符连接到str1的尾部，并以'\0'终止str1，返回str1。其中终止原str1的'\0'被str2的第一个字符覆盖。
注意，最大拷贝字符数是count+1。

#### 5 strcmp

`#include <string.h>`
int strcmp(const char *str1, const char *str2);
按字典顺序比较两个字符串，返回整数值的意义如下：
小于0，str1小于str2；
等于0，str1等于str2；
大于0，str1大于str2；

#### 6 strncmp

`#include <string.h>`
int strncmp(const char *str1, const char *str2, size_t count);
同strcmp，除了最多比较count个字符。根据比较结果返回的整数值如下：
小于0，str1小于str2；
等于0，str1等于str2；
大于0，str1大于str2；

#### 7 strchr

`#include <string.h>`
char *strchr(const char *str, int ch);
返回指向字符串str中字符ch第一次出现的位置的指针，如果str中不包含ch，则返回NULL。

#### 8 strrchr

`#include <string.h>`
char *strrchr(const char *str, int ch);
返回指向字符串str中字符ch最后一次出现的位置的指针，如果str中不包含ch，则返回NULL。

#### 9 strspn

`#include <string.h>`
size_t strspn(const char *str1, const char *str2);
返回字符串str1中由字符串str2中字符构成的第一个子串的长度。

#### 10 strcspn

`#include <string.h>`
size_t strcspn(const char *str1, const char *str2);
返回字符串str1中由不在字符串str2中字符构成的第一个子串的长度。

#### 11 strpbrk

`#include <string.h>`
char *strpbrk(const char *str1, const char *str2);
返回指向字符串str2中的任意字符第一次出现在字符串str1中的位置的指针；如果str1中没有与str2相同的字符，那么返回NULL。

#### 12 strstr

`#include <string.h>`
char *strstr(const char *str1, const char *str2);
返回指向字符串str2第一次出现在字符串str1中的位置的指针；如果str1中不包含str2，则返回NULL。

#### 13 strlen

`#include <string.h>`
size_t strlen(const char *str);
返回字符串str的长度，'\0'不算在内。

#### 14 strerror

`#include <string.h>`
char *strerror(int errnum);
返回指向与错误序号errnum对应的错误信息字符串的指针(错误信息的具体内容依赖于实现)。

#### 15 strtok

`#include <string.h>`
char *strtok(char *str1, const char *str2);
在str1中搜索由str2中的分界符界定的单词。
对strtok()的一系列调用将把字符串str1分成许多单词，这些单词以str2中的字符为分界符。第一次调用时str1非空，它搜索str1，找出由非str2中的字符组成的第一个单词，将str1中的下一个字符替换为'\0'，并返回指向单词的指针。随后的每次strtok()调用(参数str1用NULL代替)，均从前一次结束的位置之后开始，返回下一个由非str2中的字符组成的单词。当str1中没有这样的单词时返回NULL。每次调用时字符串str2可以不同。
如：



```cpp
char *p;
p = strtok("The summer soldier,the sunshine patriot", " ");
printf("%s", p);
do {
    p = strtok("\0", ", "); /* 此处str2是逗号和空格 */
    if (p)
        printf("|%s", p);
} while (p);
```

显示结果是：`The | summer | soldier | the | sunshine | patriot`

#### 16 memcpy

`#include <string.h>`
void *memcpy(void *to, const void *from, size_t count);
把from中的count个字符拷贝到to中。并返回to。

#### 17 memmove

`#include <string.h>`
void *memmove(void *to, const void *from, size_t count);
功能与memcpy类似，不同之处在于，当发生对象重叠时，函数仍能正确执行。

#### 18 memcmp

`#include <string.h>`
int memcmp(const void *buf1, const void *buf2, size_t count);
比较buf1和buf2的前count个字符，返回值与strcmp的返回值相同。

#### 19 memchr

`#include <string.h>`
void *memchr(const void *buffer, int ch, size_t count);
返回指向ch在buffer中第一次出现的位置指针，如果在buffer的前count个字符当中找不到匹配，则返回NULL。

#### 20 memset

`#include <string.h>`
void *memset(void *buf, int ch, size_t count);
把buf中的前count个字符替换为ch，并返回buf

## example
```c
#include "malloc.h"
#include "stdint.h"
#include "stdio.h"
#include "stdlib.h"
#include "string.h"
#include <stdlib.h>
#include <string.h>

int main() {
  // malloc and memset and strlen usage
  char *s = (char *)malloc(sizeof(char) * 10);
  printf("origin strlen(s): %llu\n", strlen(s));

  memset(s, 'i', 12);
  printf("after memset, strlen(s): %llu\n", strlen(s));

  int *i = (int *)malloc(sizeof(int) * 10);
  printf("sizeof(i): %llu\n", sizeof(i));

  // strcpy usage
  char *s1 = (char *)malloc(sizeof(char) * 10);
  char s2[] = {"hello"};

  printf("strlen(s1): %llu\n", strlen(s1));
  printf("sizeof(s1): %llu\n", sizeof(s1));
  printf("strlen(s2): %llu\n", strlen(s2));
  printf("sizeof(s2): %llu\n", sizeof(s2));
  strcpy_s(s1, strlen(s2) + 1, s2);
  // strncpy_s(s1, strlen(s2)+1, s2, 3);
  printf("s1=%s\n", s1);
  printf("*****************************\n");

  char s3[] = {"world"};
  // strstr return pointer
  char *p = strstr(s3, "ld");
  printf("p=%s\n", p);
  // calculate the offset
  printf("%lld\n", p-s3);

  p = strchr(s3, 'o');
  printf("strchr p=%s\n", p);
  printf("strchr offset:%lld\n", p-s3);

  p = strrchr(s3, 'l');
  printf("strrchr p=%s\n", p);
  printf("strrchr offset:%lld\n", p-s3);

  p = strtok(s3, "or");
  while (p) {
    printf("strtok p=%s\n", p);
    p = strtok(NULL, "or");
  }

  printf("after strtok, s3=%s\n", s3);
  int a = strcmp(s3, "world");
  printf("strcmp a=%d\n", a);
  a = strncmp(s3, "world", 1);
  printf("strncmp a=%d\n", a);

  char s4[] = {"hello"};
  memmove(s4+1, s4, 1);
  printf("memmove s4=%s\n", s4);

  memcpy(s4, "hello", 5);
  printf("memccpy s4=%s\n", s4);

  return 0;
}

// output:
// origin strlen(s): 6
// after memset, strlen(s): 14
// sizeof(i): 8
// strlen(s1): 6
// sizeof(s1): 8
// strlen(s2): 5
// sizeof(s2): 6
// s1=hello
// *****************************
// p=ld
// 3
// strchr p=orld
// strchr offset:1
// strrchr p=ld
// strrchr offset:3
// strtok p=w
// strtok p=ld
// after strtok, s3=w
// strcmp a=-1
// strncmp a=0
// memmove s4=hhllo
// memccpy s4=hello

```

# 二、stdlib.h中字符串与数字相互转换处理函数

#### 1. 数字转化为字符串:

● itoa()：将整型值转换为字符串。
● ltoa()：将长整型值转换为字符串。
● ultoa()：将无符号长整型值转换为字符串。
● gcvt()：将浮点型数转换为字符串，取四舍五入。
● ecvt()：将双精度浮点型值转换为字符串，转换结果中不包含十进制小数点。
● fcvt()：指定位数为转换精度，其余同ecvt()。
例子：



```cpp
# include <stdio.h>
# include <stdlib.h>
int main ()
{
   int num_int = 435;
   double num_double = 435.10f;
   char str_int[30];
   char str_double[30];
   itoa(num_int, str_int, 10);  //把整数num_int转成字符串str_int
   gcvt(num_double, 8, str_double);  //把浮点数num_double转成字符串str_double
   printf("str_int: %s\n", str_int);
   printf("str_double: %s\n", str_double);
   return 0;
}
```

程序输出结果：
str_int: 435
str_double: 435.10001
- 代码第11行中的参数10表示按十进制类型进行转换，转换后的结果是“435”，如果按二进制类型进行转换，则结果为“1101110011”。
- 代码第12行中的参数8表示精确位数，这里得到的结果是“435.10001”。

#### 2. 字符串转化为数字

- atof()：将字符串转换为双精度浮点型值。
- atoi()：将字符串转换为整型值。
- atol()：将字符串转换为长整型值。
- strtod()：将字符串转换为双精度浮点型值，并报告不能被转换的所有剩余数字。
- strtol()：将字符串转换为长整值，并报告不能被转换的所有剩余数字。
- strtoul()：将字符串转换为无符号长整型值，并报告不能被转换的所有剩余数字。
例子：

```cpp
# include <stdio.h>
# include <stdlib.h>
int main ()
{
    int num_int;
    double num_double;
    char str_int[30] = "435";         //将要被转换为整型的字符串
    char str_double[30] = "436.55";  //将要被转换为浮点型的字符串
    num_int = atoi(str_int);          //转换为整型值
    num_double = atof(str_double);  //转换为浮点型值
    printf("num_int: %d\n", num_int);
    printf("num_double: %lf\n", num_double);
    return 0;
}
// 输出结果：
// num_int: 435
// num_double: 436.550000
```

## 进制互转
```c
  // int to binary
  char tmp[10] = {};
  char *conv = itoa(5, tmp, 2);
  printf("itoa to 2: conv=%s\n", conv);

  // int to hex
  conv = itoa(17, tmp, 16);
  printf("itoa to 16: conv=0x%s\n", conv);

  // hex to 10
  int hex = strtol("0x11", NULL, 16);
  printf("strtol: hex=%d\n", hex);
  // binary to 10
  int bin = strtol("101", NULL, 2);
  printf("strtol: bin=%d\n", bin);

// output
// itoa to 2: conv=101
// itoa to 16: conv=0x11
// strtol: hex=17
// strtol: bin=5?
```

---

> 作者: map[avatar:<nil> email:<nil> link:<nil> name:<nil>]  
> URL: https://blog-12x.pages.dev/c%E8%AF%AD%E8%A8%80%E5%B8%B8%E7%94%A8%E5%87%BD%E6%95%B0/  

