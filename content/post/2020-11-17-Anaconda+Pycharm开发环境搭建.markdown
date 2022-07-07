---
layout:     post
title:      Anaconda+Pycharm开发环境搭建
subtitle:   python基础教学
date:       2020-11-17
author:     Bingo
header-img: img/python_1.png
catalog: true
tags:
    - python
    - python基础教学
---

> 最近有个小伙伴问我，怎么能快速开始python开发，为了不让他踩坑，推荐了一套《anaconda+pycharm开发大法》

# 查看python和conda版本号
```SHELL
python --version
conda --version
```
# 0. 安装anaconda和pycharm
安装anaconda和pycharm的教程就不说了，自行百度，嘿嘿~

# 1. [配置conda数据源](https://blog.csdn.net/Justdoforever/article/details/104152801)
方便我们以后pip install某些python依赖库的时候提高速度，不然国内的网络环境... 你懂得~

# 2. [配置conda库的默认安装路径](https://www.jb51.net/article/149625.htm)
防止你的系统盘越来越大，毕竟很多小伙伴不会”修电脑“~

# 3. Pycharm + Anaconda环境搭建
正文开始

### a. 创建项目
create new project
### b. 配置编译器
setting -> Project:hello-python -> Project Interpreter
...
Apply -> OK
### c. 新建一个python file
右键文件夹 -> new -> python file -> 输入文件名
**注意：文件名为：字母、下划线、数字的组合，不要出现其他字符，且开头必须是字母或下划线**

#### 输出Hello World
``` python
print("Hello World")
```
右键hello.py -> run hello.py

# Q&A
### 1. Q. 字符串和数字的区别是什么？
### 2. Q. 拼接两个字符串，并输出拼接结果？(使用join)
- "你好，"  
- "George"
<br>
#### 答案
``` python
s1 = "Hello"
s2 = "George"
# 方法1
print(s1 + " " + s2)
# 方法2
print("".join([s1, " ", s2]))
```

### 3. 判断一下a, b, c的数据类型？(使用isinstance)
``` python
a = 3
b = 3.5
c = "sleep time"
```
#### 答案
``` python
def find_instance(input):
    if isinstance(input, int):
        print("输入了一个整数")
    elif isinstance(input, float):
        print("输入了一个浮点数")
    elif isinstance(input, str):
        print("输入了一个字符串")
    else:
        print("没见过这个类型呢: " + str(type(input)))

if __name__ == '__main__':
    inputs = [3, 3.5, "sleep time", [], dict(), {}, ()]
    for input in inputs:
        find_instance(input)
```


