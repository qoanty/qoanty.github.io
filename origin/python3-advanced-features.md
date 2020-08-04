---
title: "Python3 学习笔记（高级特性）"
date: 2019-04-19T15:04:36+08:00
tags: ["python", "features"]
categories: ["Learn", "Python"]
draft: false
---

## 高级特性

在Python中，代码不是越多越好，而是越少越好。代码不是越复杂越好，而是越简单越好，代码越少，开发效率越高。

### 切片

先创建一个0-99的数列：

```python
>>> L = list(range(100))
>>> L
[0, 1, 2, 3, ..., 99]
```

可以通过切片轻松取出某一段数列。比如前10个数：

```python
>>> L[:10]
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

后10个数：

```python
>>> L[-10:]
[90, 91, 92, 93, 94, 95, 96, 97, 98, 99]
```

前11-20个数：

```python
>>> L[10:20]
[10, 11, 12, 13, 14, 15, 16, 17, 18, 19]
```

前10个数，每两个取一个：

```python
>>> L[:10:2]
[0, 2, 4, 6, 8]
```

所有数，每5个取一个：

```python
>>> L[::5]
[0, 5, 10, 15, 20, 25, 30, 35, 40, 45, 50, 55, 60, 65, 70, 75, 80, 85, 90, 95]
```

甚至什么都不写，只写`[:]`就可以原样复制一个list：

```python
>>> L[:]
[0, 1, 2, 3, ..., 99]
```

tuple也是一种list，唯一区别是tuple不可变。因此，tuple也可以用切片操作，只是操作的结果仍是tuple。

字符串`'xxx'`也可以看成是一种list，每个元素就是一个字符。因此，字符串也可以用切片操作，只是操作结果仍是字符串。

### 迭代

给定一个list或tuple，我们可以通过`for`循环来遍历这个list或tuple，这种遍历我们称为迭代（Iteration），在Python中，迭代是通过`for ... in`来完成的。

只要是可迭代对象，无论有无下标，都可以迭代，字符串也是可迭代对象，dict也可以迭代，默认情况下，dict迭代的是key，如果要迭代value，可以用`for value in d.values()`，如果要同时迭代key和value，可以用`for k, v in d.items()`。

判断一个对象是可迭代对象的方法是通过collections模块的Iterable类型判断：

```python
>>> from collections import Iterable
>>> isinstance('abc', Iterable) # str是否可迭代
True
>>> isinstance([1,2,3], Iterable) # list是否可迭代
True
>>> isinstance(123, Iterable) # 整数是否可迭代
False
```

### 列表生成式

列表生成式即List Comprehensions，是Python内置的非常简单却强大的可以用来创建list的生成式。

要生成[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]列表，可以用：

```python
>>> list(range(1, 11))
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

如果要生成[1x1, 2x2, 3x3, ..., 10x10]列表，则可以用一行语句代替循环生成的list：

```python
>>> [x * x for x in range(1, 11)]
[1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
```

写列表生成式时，把要生成的元素`x * x`放到前面，后面跟`for`循环，就可以把list创建出来，for循环后面还可以加上if判断，这样就可以筛选出仅偶数的平方：

```python
>>> [x * x for x in range(1, 11) if x % 2 == 0]
[4, 16, 36, 64, 100]
```

还可以使用两层循环，可以生成全排列：

```python
>>> [m + n for m in 'ABC' for n in 'XYZ']
['AX', 'AY', 'AZ', 'BX', 'BY', 'BZ', 'CX', 'CY', 'CZ']
```

运用列表生成式，可以写出非常简洁的代码。例如，列出当前目录下的所有文件和目录名：

```python
>>> import os # 导入os模块
>>> [d for d in os.listdir('.')] # os.listdir可以列出文件和目录
```

列表生成式也可以使用两个变量来生成list：

```python
>>> d = {'x': 'A', 'y': 'B', 'z': 'C' }
>>> [k + '=' + v for k, v in d.items()]
['y=B', 'x=A', 'z=C']
```

把一个list中所有的字符串变成小写：

```python
>>> L = ['Hello', 'World', 18, 'Apple', None]
>>> [s.lower() for s in L if isinstance(s, str)]
['hello', 'world', 'apple']
```

### 生成器

通过列表生成式创建的列表，受到内存限制，列表容量是有限的。如果列表元素可以按照某种算法推算出来，就不必创建完整的list，从而节省大量的空间，这种一边循环一边计算的机制，称为生成器：generator。

要创建一个generator，有很多种方法。第一种方法很简单，只要把一个列表生成式的`[]`改成`()`，就创建了一个generator：

```python
>>> L = [x * x for x in range(10)]
>>> L
[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
>>> g = (x * x for x in range(10))
>>> g
<generator object <genexpr> at 0x1022ef630>
```

可以通过`for`循环来迭代它：

```python
>>> g = (x * x for x in range(10))
>>> for n in g:
...     print(n)
...
```

如果推算的算法比较复杂，用类似列表生成式的`for`循环无法实现的时候，还可以用函数来实现。

例如，斐波拉契数列用列表生成式写不出来，但是用函数把它打印出来却很容易：

```python
def fib(max):
    n, a, b = 0, 0, 1
    while n < max:
        print(b)
        a, b = b, a + b
        n = n + 1
    return 'done'
```

要把上面`fib`函数变成generator，只需要把`print(b)`改为`yield b`就可以了：

```python
def fib(max):
    n, a, b = 0, 0, 1
    while n < max:
        yield b
        a, b = b, a + b
        n = n + 1
    return 'done'
```

这就是定义generator的另一种方法。如果一个函数定义中包含`yield`关键字，那么这个函数就不再是一个普通函数，而是一个generator，可以使用`for`循环来迭代：

```python
>>> for n in fib(6):
...     print(n)
...
1
1
2
3
5
8
```

但是用`for`循环调用generator时，会得不到generator的`return`语句的返回值。如果想要得到返回值，必须捕获`StopIteration`错误，返回值包含在`StopIteration`的`value`中。

实例：杨辉三角

```python
def triangles():
    L = [1]
    while True:
        yield L
        L = [sum(i) for i in zip([0] + L, L + [0])]

n = 0
for t in triangles():
    print(t)
    n = n + 1
    if n == 10:
        break
```

### 迭代器

可直接作用于`for`循环的数据类型有：一类是集合数据类型，如`list`、`tuple`、`dict`、`set`、`str`等；一类是`generator`，包括生成器和带`yield`的generator function，这些统称为可迭代对象：`Iterable`，可以使用`isinstance()`进行判断。

```python
>>> from collections import Iterable
>>> isinstance([], Iterable)
True
>>> isinstance({}, Iterable)
True
>>> isinstance('abc', Iterable)
True
>>> isinstance((x for x in range(10)), Iterable)
True
>>> isinstance(100, Iterable)
False
```

可以被`next()`函数调用并不断返回下一个值的对象称为迭代器：`Iterator`，可以使用`isinstance()`进行判断。

```python
>>> from collections import Iterator
>>> isinstance((x for x in range(10)), Iterator)
True
>>> isinstance([], Iterator)
False
>>> isinstance({}, Iterator)
False
>>> isinstance('abc', Iterator)
False
```

生成器都是`Iterator`对象，但`list`、`dict`、`str`虽然是`Iterable`，但不是`Iterator`，若要变成`Iterator`可以使用`iter()`函数。

```python
>>> isinstance(iter([]), Iterator)
True
>>> isinstance(iter('abc'), Iterator)
True
```

Python的`Iterator`对象表示的是一个数据流，我们可以看做是一个有序序列，但不能提前知道序列的长度，只能不断通过`next()`函数实现按需计算下一个数据，所以`Iterator`的计算是惰性的，只有在需要返回下一个数据时它才会计算。
