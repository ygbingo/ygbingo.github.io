---
title: "[算法基础]LectureNote1"
date: 2022-10-12T22:39:41+08:00
tags:
    - Python
    - Algorithm
    - 《算法导论》
---

# 算法的作用
1. 什么是算法？
    - 计算问题
    - 把输入转换成输出的计算步骤
2. 为什么要研究算法？
    - 计算机的运行速度不是无限快的
    - 存储器不是免费的
3. 算法解决了什么？
4. 比较下面几个时间复杂度能解决的，t时间内的，最大规模n:
    - 函数列表: $\lg{n}, \sqrt[2]{n}, n, n\lg{n}, n^2, n^3, 2^n, n!$
    - t列表: 1s, 1min, 1hour, 1day
5. 什么是RAM(random-access machine)？

# 算法基础
## 排序

**输入**: $n个数的一个序列<a_1, a_2, ..., a_n>$
<br>

**输出**: 输入序列的排序$<{a_1}',{a_2}', ..., {a_n}'> $, 满足 ${a_1}'\leq{a_2}'...\leq{a_n}'$

### 插入排序

![插入排序](https://www.runoob.com/wp-content/uploads/2019/03/insertionSort.gif)

$INSERTION-SORT(A)$
```Pseudocode
for j = 2 to A.length:  //c1
    key = A[j]  //c2
    // Insert A[j] into the sorted sequence A[1..j-1]  //0
    i = j-1  //c4
    while i>0 and A[i]>key:  //c5
      A[i+1] = A[i]  //c6
      i = i - 1  //c7
    A[i+1] = key  //c8
```
- 最好情况下，插入排序的运行时间为多少？
$T(n)=(c_1+c_2+c_4+c_5+c_8)n-(c_2+c_4+c_5+c_8)$
- 最坏情况下，插入排序的运行时间为多少？
$T(n)=(\frac{c_5}{2}+\frac{c_6}{2}+\frac{c_7}{2})n^2+(c_1+c_2+c_4+\frac{c_5}{2}-\frac{c_6}{2}-\frac{c_7}{2}+c^8)n-(c_2+c_4+c_5+c_8)$

- 时间复杂度
- 空间复杂度

- Python实现
```python
def insert_sort(A):
    for i in range(1,len(A)):  #i=1,2,3,4,5,6
        #比较A[1]和A[0],比较A[2]和A[0,1]....
        Key=A[i]
        j=i-1  #j=0,1,2,3,4,5
        while j >= 0 and Key < A[j]:
            A[j+1]=A[j]
            j=j-1
        A[j+1] = Key
    return A
```

### 归并排序

![归并排序](https://www.runoob.com/wp-content/uploads/2019/03/mergeSort.gif)

- A: 数组
- p、q、r: 数组下标
    - 满足: $p\leq{q}<r, 且A[p..q]、A[q+1..r]已排序$<br>

$MERGE(A, p, q, r)$
```Pseudocode
n1 = q-p+1
n2 = r-q
let L[1..n1+1] and R[1..n2+1] be new arrays
for i=1 to n1:
  L[i] = A[p+i-1]
for j=1 to n2:
  R[j] = A[q+j]
L[n1+1] = [MAX]
R[n2+1] = [MAX]
i=1,j=1
for k=p to r:
  if L[i] <= R[j]:
    A[k] = L[i]
    i = i +1
  else:
    A[k] = R[j]
    j = j + 1
```

$MERGE-SORT(A, p, r): 排序子数组A[p..r]$
```Pseudocode
if p<r:
  q = math.floor((p+r)/2)
  MERGE-SORT(A,p,q)
  MERGE-SORT(A,q+1,r)
  MERGE(A,p,q,r)
```

- 时间复杂度
- 空间复杂度

- 思考：能否结合归并排序和插入排序，排列数组: $[2,5,3,9,1,8,6,7]$

### 冒泡排序

![冒泡排序](https://www.runoob.com/wp-content/uploads/2019/03/bubbleSort.gif)

$BUBBLESORT(A)$
```Pseudocode
for i=1 to A.length-1:
  for j=A.length downto i+1:
    if A[j] < A[j-1]:
        exchange A[j] with A[j-1]
```
- 时间复杂度
- 空间复杂度

## 查找
**输入**: $n个数的一个序列<a_1, a_2, ..., a_n>$

**输出**: $输出序列中x的位置$

## 最大子数组问题
**输入**: $10天内公司每天的股价[10,11,7,10,6]$

**输出**: 请计算出你能做到的最大收益(卖出时的股价-买入时的股价)?

- 写一个斐波那契数列

# 基本的数据结构

## 栈与队列
### 常用概念
- 栈(stack)
    - 先入后出(last-in, first-out, LIFO)
    - 压入(PUSH): insert
    - 弹出(POP): delete
    - 上溢(overflow): 往一个容量为n的栈，已经存在n个元素后，继续进行压入操作
    - 下溢(underflow): 从一个空栈中进行弹出操作

- 队列(queue):
    - 先入先出(first-in, first-out, FIFO)
    - 入队(ENQUEUE): insert
    - 出队(DEQUEUE): delete
    - head: 队头
    - tail: 队尾
    - EMPTY: 空队列，此时head==tail
    - 上溢：向一个容量为n且插满元素的队列继续执行入队操作
    - 下溢: 从一个空队列中执行出队操作
    - 双端队列(deque): 插入和删除操作都可以在队列两端执行

### 习题
1. 思考：栈```S```初始为空，执行下列操作后，栈的结构？PUSH(S, 4)、PUSH(S, 1)、PUSH(S, 3)、POP(S)、PUSH(S, 8)
2. 编程：实现两个栈，使得两个栈的元素之和不为n时，两个栈都不会发生上溢(可以一直进行PUSH操作)

