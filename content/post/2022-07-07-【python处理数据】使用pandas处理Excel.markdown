---
layout:     post
title:      【python处理数据】使用pandas处理Excel
subtitle:   使用pandas处理Excel
date:       2022-07-07
author:     Bingo
header-img: resources/img/post-bg-1104.png
catalog: true
tags:
    - Python
    - Pandas
---

> 思考一下这个问题：<br>
> 有一个年纪成绩表.xlsx文件，包含：班级、姓名、语文、数学、英语，这个年级一共有五个班，现在想把五个班的成绩分别存到不同的excel文件中，并按总成绩排序。<br><br>
> **注意：本篇文章只做记录，想学习请直接参考官方文档**

# 前提
- 已经构建好python环境(conda、pandas)
- 了解Python基本数据类型
- 了解Python循环操作
- 了解Excel基本结构
- 了解DataFrame基本结构

# 读取Excel文件
```python
# 引用pandas包
import pandas as pd
# 直接读取文件(文件名：path_to_file.xlsx，表名：Sheet1)到DataFrame

## 方法1
df1 = pd.read_excel("path_to_file.xlsx", sheet_name="Sheet1")  

## 方法2
with pd.ExcelFile("path_to_file.xlsx") as excel:
  print(excel.sheet_names)  # 打印excel全部表名称
  df2 = excel.parse("Sheet1")
```

# 写入Excel文件
```python
# 把DataFrame数据写入到文件中(文件名：path_to_file.xlsx，表名：Sheet1)
df.to_excel("path_to_file.xlsx", sheet_name="Sheet1")

# 两个DataFrame数据分别写入到同一个文件的不同表中(文件名：path_to_file.xlsx，表名：Sheet1、Sheet2)
with pd.ExcelWriter("path_to_file.xlsx") as writer:
    df1.to_excel(writer, sheet_name="Sheet1")
    df2.to_excel(writer, sheet_name="Sheet2")
```

# 参考
1. [pandas: Excel files](https://pandas.pydata.org/docs/user_guide/io.html#excel-files)