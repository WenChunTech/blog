# Cmake简易教程


<!--more-->

# cmake简易教程

##  1. 添加头文件目录**INCLUDE_DIRECTORIES**

它相当于g++选项中的-I参数的作用，也相当于环境变量中增加路径到CPLUS_INCLUDE_PATH变量的作用。

**语法：**

```cmake
include_directories([AFTER|BEFORE] [SYSTEM] dir1 [dir2 ...])
```

## 2. 添加需要链接的库文件目录**LINK_DIRECTORIES**

**语法：**

```cmake
link_directories(directory1 directory2 ...)
```

它相当于g++命令的-L选项的作用，也相当于环境变量中增加LD_LIBRARY_PATH的路径的作用。

```
link_directories("/home/server/third/lib")
```

## 3. 添加需要链接的库文件路径**LINK_LIBRARIES**

**语法：**

```cmake
link_libraries(library1 <debug | optimized> library2 ...)
```

## 4. 设置要链接的库文件的名称**TARGET_LINK_LIBRARIES** 

**语法：**

```cmake
target_link_libraries(<target> [item1 [item2 [...]]]
                      [[debug|optimized|general] <item>] ...)
```

## 5. 查找库所在目录**FIND_LIBRARY**

**语法：**

```cmake
find_library (<VAR> name1 [path1 path2 ...])
find_library (
          <VAR>
          name | NAMES name1 [name2 ...] [NAMES_PER_DIR]
          [HINTS path1 [path2 ... ENV var]]
          [PATHS path1 [path2 ... ENV var]]
          [PATH_SUFFIXES suffix1 [suffix2 ...]]
          [DOC "cache documentation string"]
          [NO_DEFAULT_PATH]
          [NO_CMAKE_ENVIRONMENT_PATH]
          [NO_CMAKE_PATH]
          [NO_SYSTEM_ENVIRONMENT_PATH]
          [NO_CMAKE_SYSTEM_PATH]
          [CMAKE_FIND_ROOT_PATH_BOTH |
           ONLY_CMAKE_FIND_ROOT_PATH |
           NO_CMAKE_FIND_ROOT_PATH]
         )
```

例子如下：

```
FIND_LIBRARY(RUNTIME_LIB rt /usr/lib  /usr/local/lib NO_DEFAULT_PATH)
```

cmake会在目录中查找，如果所有目录中都没有，值RUNTIME_LIB就会被赋为NO_DEFAULT_PATH

## 6. 查找源文件和头文件

```cmake
# 查找指定目录下的所有.cpp与.h文件 并存放到指定变量名SC_FILES中

FILE(GLOB SC_FILES "*.cpp" "*.h")
```

## 7. 添加源文件目录

使用 `aux_source_directory` 命令，该命令会查找指定目录下的所有源文件，然后将结果存进指定变量名。

**语法：**

```cmake
aux_source_directory(<dir> <variable>)

# 生成链接库
add_library (MathFunctions ${DIR_LIB_SRCS})
```

**参考链接：**

- [cmake 添加头文件目录，链接动态、静态库 - 王彬彬 - 博客园 (cnblogs.com)](https://www.cnblogs.com/binbinjx/p/5626916.html)


---

> 作者:   
> URL: https://release.blog-12x.pages.dev/cmake%E7%AE%80%E6%98%93%E6%95%99%E7%A8%8B/  

