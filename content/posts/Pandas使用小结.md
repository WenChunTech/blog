---
title: "Pandas使用小结"
subtitle: ""
date: 2022-11-12T21:12:01+08:00
draft: false
author: ""
authorLink: ""
authorEmail: ""
description: ""
keywords: ""
license: ""
comment: false
weight: 0

tags:
  - pandas
categories:
  - Python

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

# 索引

```python
import pandas as pd
import numpy as np

df = pd.DataFrame(np.arange(50).reshape(-1, 5), columns=list('abcde'))
# 根据列标签索引
df['a']
# 或者
df.loc[:,'a']

# 根据列索引
df.iloc[:, 0]

# df['a'] == df.loc[:,'a'] == df.iloc[:, 0]
```

# 计算

```python
# 所有元素乘法
df * 10

# 指定列相乘
# 注意：不会在原地修改，需要重新赋值
df['a'] = df['a'] * 10

# 指定位置操作
df.iloc[0,3] = 100

# 也可以指定多列操作，生成新列
df['e'] = df['a'] * df['b']

```

# 合并

```python

df_new = pd.DataFrame(np.arange(200, 230).reshape(-1, 2), columns=['z','x'])

# 注意：这里因为2个DataFrame行索引没有冲突，索引这里没有问题。
# axis 为0时以行合并(可以是实现数据追加)，为1时以列合并
pd.concat([df, df_new], axis=1)

# 当下面的df的行索引不连续时，df_new就会缺少一些行的数据
df = df[df.loc[:, 'a'] % 100 == 0]
pd.concat([df, df_new], axis=1)

# 如果当出现上面一种情况，而又不想丢掉df_new的数据时可以使用重新索引的方法
df = df.reset_index(drop=True)
pd.concat([df, df_new], axis=1)
```

# 追加数据

```python
# 不能这样追加，需要列标签相对应
df.append([1, 3, 5, 3,5])

# 新Dataframe需要和df标签对应
new_row = pd.DataFrame(np.full((1,4), 5), columns=list('abcd'))
df.append(new_row)

# 也可以这样
# pd.concat([df, new_row], axis=0)
```

# Apply 使用

```python
# 当Dateframe中不再是存数字时，就不能根据直接对列进行操作生成新列，有些时候还含有复杂的条件判断，这时候既可以使用apply函数
# 注意当新加列时需要设置axis=1
df['f'] = df[['b','c']].apply(lambda x: x[0]*100 if x[0] % 2 == 0 else x[1] - 100, axis=1)

```

# 排序

```python
# 根据索引排序,axis=0代表行，axis=1代表列
df.sort_index()

# 根据值排序,可根据多个值排序
df.sort_values(by=['a'])

# 如果Dataframe含有重复列，重复列的数据又不相同，但又想按值排序，可以参考下面
# 比如说2，4列名相同，想按照第2列的值排序
# sort_col_values = df.iloc[:, 2].to_numpy()
# sort_indexs = sort_col_values.argsort()
# df = df.iloc[:, sort_indexs]
```

# 去重

```python
# 使用函数drop_duplicates，其中subset指定要去重的列名，不能写列索引，即指定的相同列才进行去重，默认按照所有列相同去重，keep参数可以指定保留first，last。inplace选择是否修改原Dataframe
# df.drop_duplicates(subset=[], keep='first', inplace=False)

# 当Dataframe有重复列名，重复列名数据不相同时，可以将Dataframe拆分，再进行去重，去重后再合并
# 例如：2，4列名重复，需要按照1，2列去重
# 拆分
front = df.iloc[:, :2]
back = df.iloc[:, 2:]

# 去重
front.drop_duplicates(subset=['a', 'b'], keep='first', inplace=True)
back = back.iloc[front.index, :]

# 合并,这里能保证行索引能一一对应，所以不用重建索引也行。当时也可以选择重建索引
pd.concat([front, back], axis=1)

```

# 字符串操作
> 在pandas中可以针对需要操作列方便的使用字符串的所有函数
```python
# 举例：
# 注意：这里的replace方法默认使用正则表达式，可以添加regex=False来关闭
df['a'].str.replace(regex=False)
```

# 分组和聚合
```python
# 对指定列进行聚合计算，agg会返回分组结果，并且列数据为聚合操作后的结果
# 使用以下访问生成聚合结果
lang = df.groupby(by='姓名').agg({'语文': 'sum'})
math = df.groupby(by='姓名').agg({'数学': 'sum'})
english = df.groupby(by='姓名').agg({'英语': 'sum'})
# pd.concat([lang, math, english], axis=1)
# 以上等于下面这行。
# df.groupby(by='姓名').sum()
# 如果要对不同列进行不同聚合操作，可以使用以下方法：
df = df.groupby(by='姓名').agg({'语文': 'sum', '姓名': 'count'})

# 注意：如果使用以上方法会出现重复列，那就需要重命名列名
df = df.rename({'姓名': 'count'}, axis=1)

# 使用transform返回DataFrame
# df['count'] = df.groupby(by='姓名')['数学'].transform('sum')
```

# 读取和保存
```python
# 读取csv提供了许多使用的选项，比如可以指定分隔符，读取部分数据，跳过表头，指定列名读取
# 参考：https://www.gairuo.com/p/pandas-read-csv
pd.read_csv(
    filepath_or_buffer: 'FilePathOrBuffer',
    sep=<no_default>,
    delimiter=None,
    header='infer',
    names=<no_default>,
    index_col=None,
    usecols=None,
    squeeze=False,
    prefix=<no_default>,
    mangle_dupe_cols=True,
    dtype: 'DtypeArg | None' = None,
    engine=None,
    converters=None,
    true_values=None,
    false_values=None,
    skipinitialspace=False,
    skiprows=None,
    skipfooter=0,
    nrows=None,
    na_values=None,
    keep_default_na=True,
    na_filter=True,
    verbose=False,
    skip_blank_lines=True,
    parse_dates=False,
    infer_datetime_format=False,
    keep_date_col=False,
    date_parser=None,
    dayfirst=False,
    cache_dates=True,
    iterator=False,
    chunksize=None,
    compression='infer',
    thousands=None,
    decimal: 'str' = '.',
    lineterminator=None,
    quotechar='"',
    quoting=0,
    doublequote=True,
    escapechar=None,
    comment=None,
    encoding=None,
    encoding_errors: 'str | None' = 'strict',
    dialect=None,
    error_bad_lines=None,
    warn_bad_lines=None,
    on_bad_lines=None,
    delim_whitespace=False,
    low_memory=True,
    memory_map=False,
    float_precision=None,
    storage_options: 'StorageOptions' = None,
)

# 读取excel
# 参考https://www.gairuo.com/p/pandas-read-excel
pd.read_excel(io, sheet_name=0, header=0,
              names=None, index_col=None,
              usecols=None, squeeze=False,
              dtype=None, engine=None,
              converters=None, true_values=None,
              false_values=None, skiprows=None,
              nrows=None, na_values=None,
              keep_default_na=True, verbose=False,
              parse_dates=False, date_parser=None,
              thousands=None, comment=None, skipfooter=0,
              convert_float=True, mangle_dupe_cols=True, **kwargs)

# 有时候需要在已有excel中追加数据，这个在csv中很好实现设置moda='a'即可，在excel就有点麻烦了。
# 因为excel中格式比较复杂，如果表头想要合并单元格，设置颜色等样式时，我们可以先用excel设计好文件模板，然后再将数据追加到excel保存即可
# 实现如下：

import pandas as pd
from openpyxl import load_workbook


def append_excel(filepath: str):
    filepath = ''
    with pd.ExcelWriter(filepath, engine='openpyxl', mode='a', if_sheet_exists="overlay") as writer:
        df_origin = pd.DataFrame(
            pd.read_excel(filepath, sheet_name='sheet_name'))  #读取原数据文件和表
        book = load_workbook(filepath)
        writer.book = book
        # 复制原有excel所有sheet
        writer.sheets = dict((ws.title, ws) for ws in book.worksheets)
        #获取原数据的行数
        df_rows = df_origin.shape[0]
        df.to_excel(writer,
                    sheet_name=writer.book.active.title,
                    startrow=df_rows + 1,
                    index=False,
                    header=False)  #将数据写入excel中的aa表,从第一个空行开始写
        writer.save()  #保存
```
# 最后再说一句：
> 如果想要在保存DataFram数据为excel并且想要同时设置样式时可以使用pd.ExcelWriter生成writer对象，创建sheet的样式，然后再使用`df.to_excel(writer)`保存即可

# 参考连接：
- https://www.gairuo.com/p/pandas

- https://www.pypandas.cn/docs/