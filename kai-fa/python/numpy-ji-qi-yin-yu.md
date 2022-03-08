# numpy 及其隐喻

## Array creation

1. Conversion from other Python structures (i.e. lists and tuples)
2. Intrinsic NumPy array creation functions (e.g. arange, ones, zeros, etc.)
3. Replicating, joining, or mutating existing arrays
4. Reading arrays from disk, either from standard or custom formats
5. Creating arrays from raw bytes through the use of strings or buffers
6. Use of special library functions (e.g., random)

## Indexing on ndarrays



## I/O with NumPy

## 数据类型 dtype

* 数据的类型（整数，浮点数或者 Python 对象）
* 数据的大小（例如， 整数使用多少个字节存储）
* 数据的字节顺序（小端法或大端法）
* 在结构化类型的情况下，字段的名称、每个字段的数据类型和每个字段所取的内存块的部分
* 如果数据类型是子数组，那么它的形状和数据类型是什么。



| 字符   | 对应类型            |
| ---- | --------------- |
| b    | 布尔型             |
| i    | (有符号) 整型        |
| u    | 无符号整型 integer   |
| f    | 浮点型             |
| c    | 复数浮点型           |
| m    | timedelta（时间间隔） |
| M    | datetime（日期时间）  |
| O    | (Python) 对象     |
| S, a | (byte-)字符串      |
| U    | Unicode         |
| V    | 原始数据 (void)     |



## Broadcasting

## Byte-swapping

#### numpy.ndarray.byteswap()

numpy.ndarray.byteswap() 函数将 ndarray 中每个元素中的字节进行大小端转换

## 结构化数组 <a href="#jie-gou-hua-shu-zu" id="jie-gou-hua-shu-zu"></a>

* 对应的是C语言的结构体数组
* 对应tuple数组
* 对应dataframe
* 对应sql中的table

对应的功能：

按行获取，按列获取， 过滤



mytype='int,float,int'

使用元组创建dtype类型

\[ (' 字段一 ‘，’类型一‘，（形状一）),(' 字段二 ‘，’类型二，（形状二）),(' 字段三 ‘，’类型三‘，（形状三）)] \[('name', '\<U10'), ('age', '\<i4'), ('sex', '\<U6'), ('weight', '\<f8')]

student\_type={'names':('name', 'age', 'sex','weight'), 'formats':('U10', 'i4','U6', 'f8')}

**访问和修改字段名称——names属性**

print(x.dtype.names) #访问

x.dtype.names=('age','height','weight','width') #修改字段名称

**一次访问多个列**

x<mark style="color:red;">**\[\['**</mark>col1','col2','col3'<mark style="color:red;">**]]**</mark>** ** #使用**两个**中括号

class numpy.<mark style="color:red;">**recarray**</mark>(shape, dtype=None, buf=None, offset=0, strides=None, formats=None, names=None, titles=None, byteorder=None, aligned=False, order='C')

构造一个允许使用属性进行字段访问的 ndarray。

## 自定义容器



## 容器子类



## ufunc - udf

#### 创建:

`frompyfunc()`方法采用以下参数：

* _`function`_- 函数的名称。
* _`inputs`_- 输入参数(数组)的数目。
* _`outputs`_- 输出数组的数目。

#### ufunc的方法：

| [`ufunc.reduce`](https://numpy.org/doc/stable/reference/generated/numpy.ufunc.reduce.html#numpy.ufunc.reduce)(array\[, axis, dtype, out, ...])        | Reduces [`array`](https://numpy.org/doc/stable/reference/generated/numpy.array.html#numpy.array)'s dimension by one, by applying ufunc along one axis. |
| ----------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| [`ufunc.accumulate`](https://numpy.org/doc/stable/reference/generated/numpy.ufunc.accumulate.html#numpy.ufunc.accumulate)(array\[, axis, dtype, out]) | Accumulate the result of applying the operator to all elements.                                                                                        |
| [`ufunc.reduceat`](https://numpy.org/doc/stable/reference/generated/numpy.ufunc.reduceat.html#numpy.ufunc.reduceat)(array, indices\[, axis, ...])     | Performs a (local) reduce with specified slices over a single axis.                                                                                    |
| [`ufunc.outer`](https://numpy.org/doc/stable/reference/generated/numpy.ufunc.outer.html#numpy.ufunc.outer)(A, B, /, \*\*kwargs)                       | Apply the ufunc _op_ to all pairs (a, b) with a in _A_ and b in _B_.                                                                                   |
| [`ufunc.at`](https://numpy.org/doc/stable/reference/generated/numpy.ufunc.at.html#numpy.ufunc.at)(a, indices\[, b])                                   | Performs unbuffered in place operation on operand 'a' for elements specified by 'indices'.                                                             |



## Copies and views - 视图

#### a=b

a,b 的id一致

#### ndarray.view()

对视图的修改会直接反映到原数据。

数组的维数变化不会改变原始数据的维数

#### 切片 \[n:m]

对视图的修改会直接反映到原数据。

数组的维数变化不会改变原始数据的维数

#### ndarray.copy()

创建一个副本。 对副本数据进行修改，不会影响到原始数据，它们物理内存不在同一位置。
