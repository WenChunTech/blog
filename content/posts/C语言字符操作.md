---
title: "C语言字符操作"
subtitle: ""
date: 2022-11-27T20:26:39+08:00
draft: true
author: ""
authorLink: ""
authorEmail: ""
description: ""
keywords: ""
license: ""
comment: false
weight: 0

tags:
- draft
categories:
- draft

hiddenFromHomePage: false
hiddenFromSearch: false

summary: ""
resources:
- name: featured-image
  src: featured-image.jpg
- name: featured-image-preview
  src: featured-image-preview.jpg

toc:
  enable: true
math:
  enable: false
lightgallery: false
seo:
  images: []

repost:
  enable: true
  url: ""

# See details front matter: https://fixit.lruihao.cn/theme-documentation-content/#front-matter
---

<!--more-->



# C语言字符串处理



# string.h中字符串处理函数

​	在头文件<string.h>中定义了两组字符串函数。第一组函数的名字以str开头；第二组函数的名字以mem开头。
只有函数memmove对重叠对象间的拷贝进行了定义，而其他函数都未定义。比较类函数将其变量视为unsigned char类型的数组。

#### 1. strcpy

`#include <string.h>`
char *strcpy(char *str1, const char *str2);
把字符串str2(包括'\0')拷贝到字符串str1当中，并返回str1。

#### 2. strncpy

`#include <string.h>`
char *strncpy(char *str1, const char *str2, size_t count);
把字符串str2中最多count个字符拷贝到字符串str1中，并返回str1。如果str2中少于count个字符，那么就用'\0'来填充，直到满足count个字符为止。

#### 3.  strcat

`#include <string.h>`
char *strcat(char *str1, const char *str2);
把str2(包括'\0')拷贝到str1的尾部(连接)，并返回str1。其中终止原str1的'\0'被str2的第一个字符覆盖。

#### 4. strncat

`#include <string.h>`
char *strncat(char *str1, const char *str2, size_t count);
把str2中最多count个字符连接到str1的尾部，并以'\0'终止str1，返回str1。其中终止原str1的'\0'被str2的第一个字符覆盖。
注意，最大拷贝字符数是count+1。

#### 5.  strcmp

`#include <string.h>`
int strcmp(const char *str1, const char *str2);
按字典顺序比较两个字符串，返回整数值的意义如下：
小于0，str1小于str2；
等于0，str1等于str2；
大于0，str1大于str2；

#### 6.  strncmp

`#include <string.h>`
int strncmp(const char *str1, const char *str2, size_t count);
同strcmp，除了最多比较count个字符。根据比较结果返回的整数值如下：
小于0，str1小于str2；
等于0，str1等于str2；
大于0，str1大于str2；

#### 7.  strchr

`#include <string.h>`
char *strchr(const char *str, int ch);
返回指向字符串str中字符ch第一次出现的位置的指针，如果str中不包含ch，则返回NULL。

#### 8. strrchr

`#include <string.h>`
char *strrchr(const char *str, int ch);
返回指向字符串str中字符ch最后一次出现的位置的指针，如果str中不包含ch，则返回NULL。

#### 9. strspn

`#include <string.h>`
size_t strspn(const char *str1, const char *str2);
返回字符串str1中由字符串str2中字符构成的第一个子串的长度。

#### 10. strcspn

`#include <string.h>`
size_t strcspn(const char *str1, const char *str2);
返回字符串str1中由不在字符串str2中字符构成的第一个子串的长度。

#### 11. strpbrk

`#include <string.h>`
char *strpbrk(const char *str1, const char *str2);
返回指向字符串str2中的任意字符第一次出现在字符串str1中的位置的指针；如果str1中没有与str2相同的字符，那么返回NULL。

#### 12. strstr

`#include <string.h>`
char *strstr(const char *str1, const char *str2);
返回指向字符串str2第一次出现在字符串str1中的位置的指针；如果str1中不包含str2，则返回NULL。

#### 13. strlen

`#include <string.h>`
size_t strlen(const char *str);
返回字符串str的长度，'\0'不算在内。

#### 14.  strerror

`#include <string.h>`
char *strerror(int errnum);
返回指向与错误序号errnum对应的错误信息字符串的指针(错误信息的具体内容依赖于实现)。

#### 15.  strtok

`#include <string.h>`
char *strtok(char *str1, const char *str2);
在str1中搜索由str2中的分界符界定的单词。
对strtok()的一系列调用将把字符串str1分成许多单词，这些单词以str2中的字符为分界符。第一次调用时str1非空，它搜索str1，找出由非str2中的字符组成的第一个单词，将str1中的下一个字符替换为'\0'，并返回指向单词的指针。随后的每次strtok()调用(参数str1用NULL代替)，均从前一次结束的位置之后开始，返回下一个由非str2中的字符组成的单词。当str1中没有这样的单词时返回NULL。每次调用时字符串str2可以不同。
如：

#### 16.  memcpy

`#include <string.h>`
void *memcpy(void *to, const void *from, size_t count);
把from中的count个字符拷贝到to中。并返回to。

#### 17.  memmove

`#include <string.h>`
void *memmove(void *to, const void *from, size_t count);
功能与memcpy类似，不同之处在于，当发生对象重叠时，函数仍能正确执行。

#### 18.  memcmp

`#include <string.h>`
int memcmp(const void *buf1, const void *buf2, size_t count);
比较buf1和buf2的前count个字符，返回值与strcmp的返回值相同。

#### 19.  memchr

`#include <string.h>`
void *memchr(const void *buffer, int ch, size_t count);
返回指向ch在buffer中第一次出现的位置指针，如果在buffer的前count个字符当中找不到匹配，则返回NULL。

#### 20.  memset

`#include <string.h>`
void *memset(void *buf, int ch, size_t count);
把buf中的前count个字符替换为ch，并返回buf。



**参考资料**

- [C语言字符串处理库函数大全 - 简书 (jianshu.com)](https://www.jianshu.com/p/28773877ffba)
