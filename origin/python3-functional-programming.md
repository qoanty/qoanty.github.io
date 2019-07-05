---
title: "Python3 学习笔记（函数式编程）"
date: 2019-04-19T19:23:49+08:00
tags: ["python", "functions"]
categories: ["Learn", "Python"]
draft: false
---

## 函数式编程

函数是Python内建支持的一种封装，我们通过把大段代码拆成函数，通过一层一层的函数调用，就可以把复杂任务分解成简单的任务，这种分解可以称之为面向过程的程序设计。函数就是面向过程的程序设计的基本单元。

对于编程语言，就是越低级的语言，越贴近计算机，抽象程度低，执行效率高，比如C语言；越高级的语言，越贴近计算，抽象程度高，执行效率低，比如Lisp语言。

函数式编程(Functional Programming)的一个特点就是，允许把函数本身作为参数传入另一个函数，还允许返回一个函数！

Python对函数式编程提供部分支持。由于Python允许使用变量，因此，Python不是纯函数式编程语言。

### 高阶函数

**变量可以指向函数**

以Python内置的求绝对值的函数`abs()`为例。

```python
>>> abs(-10)
10
>>> abs
<built-in function abs>
>>> x = abs(-10)
>>> x
10
>>> f = abs
>>> f(-10)
10
```

可见，`abs(-10)`是函数调用，而`abs`是函数本身，我们可以把结果赋值给变量，也可以把函数本身赋值给变量，即：变量可以指向函数。

**函数名也是变量**

函数名其实就是指向函数的变量，对于`abs()`这个函数，完全可以把函数名`abs`看成变量，如果把`abs`指向10后，就无法通过`abs(-10)`调用该函数了。

```python
>>> abs = 10
>>> abs(-10)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'int' object is not callable
```

**传入函数**

既然变量可以指向函数，函数的参数能接收变量，那么一个函数就可以接收另一个函数作为参数，这种函数就称之为高阶函数。

```python
>>> def add(x, y, f):
...     return f(x) + f(y)
...
>>> x = -5
>>> y = 6
>>> f = abs
>>> add(x, y, f)
11
```

把函数作为参数传入，这样的函数称为高阶函数，函数式编程就是指这种高度抽象的编程范式。

#### map/reduce

Python内建了`map()`和`reduce()`函数。

`map()`函数接收两个参数，一个是函数，一个是`Iterable`，`map`将传入的函数依次作用到序列的每个元素，并把结果作为新的`Iterator`返回。

```python
>>> def f(x):
...     return x * x
...
>>> r = map(f, [1, 2, 3, 4, 5, 6, 7, 8, 9])
>>> list(r)
[1, 4, 9, 16, 25, 36, 49, 64, 81]
```

`map()`传入的第一个参数是`f`，即函数对象本身。由于结果`r`是一个`Iterator`，因此可以通过`list()`函数把整个序列都计算出来并返回一个list。

`map()`作为高阶函数，事实上它把运算规则抽象了，因此它可以计算任意复杂的函数，比如，把list所有数字转为字符串：

```python
>>> list(map(str, [1, 2, 3, 4, 5, 6, 7, 8, 9]))
['1', '2', '3', '4', '5', '6', '7', '8', '9']
```

`reduce`把一个函数作用在一个序列`[x1, x2, x3, ...]`上，这个函数必须接收两个参数，`reduce`把结果继续和序列的下一个元素做累积计算，其效果就是：

```python
reduce(f, [x1, x2, x3, x4]) = f(f(f(x1, x2), x3), x4)
```

把序列`[1, 3, 5, 7, 9]`变换成整数`13579`，就可以用`reduce`实现：

```python
>>> from functools import reduce
>>> def fn(x, y):
...     return x * 10 + y
...
>>> reduce(fn, [1, 3, 5, 7, 9])
13579
```

考虑到字符串`str`也是一个序列，配合`map()`可以写出把`str`转换为`int`的函数：

```python
from functools import reduce

DIGITS = {'0': 0, '1': 1, '2': 2, '3': 3, '4': 4, '5': 5, '6': 6, '7': 7, '8': 8, '9': 9}

def char2num(s):
    return DIGITS[s]

def str2int(s):
    return reduce(lambda x, y: x * 10 + y, map(char2num, s))
```

#### filter

Python内建的`filter()`函数用于过滤序列。

`filter()`接收一个函数和一个序列，把传入的函数依次作用于每个元素，然后根据返回值是`True`还是`False`决定保留还是丢弃该元素。

例如，在一个list中，删掉偶数，只保留奇数：

```python
def is_odd(n):
    return n % 2 == 1

list(filter(is_odd, [1, 2, 4, 5, 6, 9, 10, 15]))
# 结果: [1, 5, 9, 15]
```

把一个序列中的空字符串删掉：

```python
def not_empty(s):
    return s and s.strip()

list(filter(not_empty, ['A', '', 'B', None, 'C', '  ']))
# 结果: ['A', 'B', 'C']
```

可见用`filter()`这个高阶函数，关键在于正确实现一个“筛选”函数。

求素数：

```python
>>> def _odd_iter():  # 生成器，构造一个从3开始的奇数序列并且是一个无限序列
...     n = 1
...     while True:
...         n = n + 2
...         yield n
...
>>> def _not_divisible(n):  # 定义一个筛选函数
...     return lambda x: x % n > 0
...
>>> def primes():  # 定义一个生成器，不断返回下一个素数
...     yield 2
...     it = _odd_iter()  # 初始序列
...     while True:
...         n = next(it)  # 返回序列的第一个数
...         yield n
...         it = filter(_not_divisible(n), it)  # 构造新序列
...
>>> for n in primes():  # 打印100以内的素数
...     if n < 100:
...         print(n)
...     else:
...         break
...
```

筛选出回数：

```python
>>> def is_palindrome(n):
...     return str(n) == str(n)[::-1]
...
>>> output = filter(is_palindrome, range(1, 200))
>>> list(output)
[1, 2, 3, 4, 5, 6, 7, 8, 9, 11, 22, 33, 44, 55, 66, 77, 88, 99, 101, 111, 121, 131, 141, 151, 161, 171, 181, 191]
```

#### sorted

Python内置的`sorted()`函数可以对list进行排序：

```python
>>> sorted([36, 5, -12, 9, -21])
[-21, -12, 5, 9, 36]
```

此外，`sorted()`函数也是一个高阶函数，它还可以接收一个`key`函数来实现自定义的排序，例如按绝对值大小排序：

```python
>>> sorted([36, 5, -12, 9, -21], key=abs)
[5, 9, -12, -21, 36]
```

字符串排序的例子：

```python
>>> sorted(['bob', 'about', 'Zoo', 'Credit'])
['Credit', 'Zoo', 'about', 'bob']
```

字符串忽略大小写的排序：

```python
>>> sorted(['bob', 'about', 'Zoo', 'Credit'], key=str.lower)
['about', 'bob', 'Credit', 'Zoo']
```

字符串反向排序：

```python
>>> sorted(['bob', 'about', 'Zoo', 'Credit'], key=str.lower, reverse=True)
['Zoo', 'Credit', 'bob', 'about']
```

从上述例子可以看出，高阶函数的抽象能力是非常强大的，而且，核心代码可以保持得非常简洁。

### 返回函数

**函数作为返回值**

高阶函数除了可以接受函数作为参数外，还可以把函数作为结果值返回。

通常情况下，求和的函数为：

```python
def calc_sum(*args):
    ax = 0
    for n in args:
        ax = ax + n
    return ax
```

但是，如果不需要立刻求和，而是在后面的代码中根据需要再计算，则可以不返回求和的结果，而是返回求和的函数：

```python
def lazy_sum(*args):
    def sum():
        ax = 0
        for n in args:
            ax = ax + n
        return ax
    return sum
```

当我们调用`lazy_sum()`时，返回的并不是求和结果，而是求和函数，调用函数`f`时，才真正计算求和的结果：

```python
>>> f = lazy_sum(1, 3, 5, 7, 9)
>>> f
<function lazy_sum.<locals>.sum at 0x101c6ed90>
>>> f()
25
```

在函数`lazy_sum`中又定义了函数`sum`，并且，内部函数`sum`可以引用外部函数`lazy_sum`的参数和局部变量，当`lazy_sum`返回函数`sum`时，相关参数和变量都保存在返回的函数中，这种称为“闭包（Closure）”的程序结构拥有极大的威力。

需注意一点，当我们调用`lazy_sum()`时，每次调用都会返回一个新的函数，即使传入相同的参数：

```python
>>> f1 = lazy_sum(1, 3, 5, 7, 9)
>>> f2 = lazy_sum(1, 3, 5, 7, 9)
>>> f1==f2
False
```

**闭包**

返回的函数在其定义内部引用了局部变量`args`，所以，当一个函数返回了一个函数后，其内部的局部变量还被新函数引用，所以，闭包用起来简单，实现起来可不容易。另一个需要注意的问题是，返回的函数并没有立刻执行，而是直到调用了`f()`才执行。

返回闭包时：返回函数不要引用任何循环变量，或者后续会发生变化的变量。

### 匿名函数

在Python中，对匿名函数提供了有限支持。计算$f(x)=x^2$时，除了定义一个`f(x)`的函数外，还可以直接传入匿名函数：

```python
>>> list(map(lambda x: x * x, range(1, 10)))
[1, 4, 9, 16, 25, 36, 49, 64, 81]
```

关键字`lambda`表示匿名函数，冒号前面的`x`表示函数参数，匿名函数有个限制，就是只能有一个表达式，不用写`return`，返回值就是该表达式的结果。

此外，匿名函数也是一个函数对象，也可以把匿名函数赋值给一个变量，再利用变量来调用该函数，也可以把匿名函数作为返回值返回。

### 装饰器

在代码运行期间动态增加功能的方式，称之为“装饰器”（Decorator），本质上decorator就是一个返回函数的高阶函数。

一个完整的decorator的写法如下：

```python
import functools

def log(func):
    @functools.wraps(func)
    def wrapper(*args, **kw):
        print('call %s():' % func.__name__)
        return func(*args, **kw)
    return wrapper
```

或者针对带参数的decorator：

```python
import functools

def log(text):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kw):
            print('%s %s():' % (text, func.__name__))
            return func(*args, **kw)
        return wrapper
    return decorator
```

`log()`是一个decorator，返回`wrapper()`函数，`wrapper()`函数的参数定义是`(*args, **kw)`，因此，`wrapper()`函数可以接受任意参数的调用。在`wrapper()`函数内，首先打印日志，再紧接着调用原始函数。

Python内置的`functools.wraps`可以把原始函数的`__name__`等属性复制到`wrapper()`函数中，否则，有些依赖函数签名的代码执行就会出错。

在面向对象（OOP）的设计模式中，decorator被称为装饰模式。OOP的装饰模式需要通过继承和组合来实现，而Python除了能支持OOP的decorator外，直接从语法层次支持decorator。Python的decorator可以用函数实现，也可以用类实现。

### 偏函数

Python的`functools`模块提供了很多有用的功能，其中一个就是偏函数（Partial function）。

```python
>>> import functools
>>> int2 = functools.partial(int, base=2)
>>> int2('1000000')
64
>>> int2('1010101')
85
```

`functools.partial`的作用就是把一个函数的某些参数给固定住（也就是设置默认值），返回一个新的函数，调用这个新函数会更简单。

创建偏函数时，实际上可以接收函数对象、`*args`和`**kw`这3个参数，当传入：

```python
int2 = functools.partial(int, base=2)
```

实际上固定了`int()`函数的关键字参数`base`，`int2('10010')`相当于：

```python
kw = { 'base': 2 }
int('10010', **kw)
```

当传入：

```python
max2 = functools.partial(max, 10)
```

实际上会把10作为`*args`的一部分自动加到左边，`max2(5, 6, 7)`相当于：

```python
args = (10, 5, 6, 7)
max(*args)
```

## 模块

在Python中，一个.py文件就称之为一个模块（Module），使用模块的好处是提高了代码的可维护性，可以被其他地方引用，还可以避免函数名和变量名冲突。

为了避免模块名冲突，Python又引入了按目录来组织模块的方法，称为包（Package），每一个包目录下面都会有一个`__init__.py`的文件，这个文件是必须存在的，否则，Python就把这个目录当成普通目录，而不是一个包。

在命令行运行模块文件时，Python解释器把一个特殊变量`__name__`设置为`__main__`，而如果在其他地方导入该模块时，`if`判断将失效，这种`if`测试可以让一个模块通过命令行运行时执行一些额外的代码，最常见的就是运行测试。

