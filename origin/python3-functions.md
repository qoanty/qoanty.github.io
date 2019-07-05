---
title: "Python3 学习笔记（函数）"
date: 2019-04-19T10:14:31+08:00
tags: ["python", "functions"]
categories: ["Learn", "Python"]
draft: false
---

## 函数

### 调用函数

Python内置了很多有用的函数，可以直接调用，[参见官方文档](https://docs.python.org/3/library/functions.html)。

调用`abs`、`max`函数：

```python
>>> abs(-20)
20
>>> abs(12.34)
12.34
>>> max(1, 2)
2
>>> max(2, 3, 1, -5)
3
```

调用数据类型转换函数：

```python
>>> int('123')
123
>>> int(12.34)
12
>>> float('12.34')
12.34
>>> str(1.23)
'1.23'
>>> str(100)
'100'
>>> bool(1)
True
>>> bool('')
False
```

## 定义函数

定义函数时，需要确定函数名和参数个数；

如果有必要，可以先对参数的数据类型做检查；

函数体内部可以用return返回函数结果；

函数执行完没有return语句时，自动return None；

函数可以同时返回多个值，其实就是一个tuple。

## 函数的参数

Python的函数定义非常简单，但灵活度却非常大。除了正常定义的必选参数外，还可以使用默认参数、可变参数和关键字参数，使得函数定义出来的接口，不但能处理复杂的参数，还可以简化调用者的代码。

### 位置参数

定义计算$x^2$的函数：

```python
def power(x):
    return x * x
```

对于`power(x)`函数，参数`x`就是一个位置参数，调用`power`函数时，必须传入有且仅有的一个参数`x`：

```python
>>> power(5)
25
>>> power(15)
225
```

若要计算$x^3$、$x^4$、$x^5$......，可以把函数`power(x)`修改为`power(x, n)`，来计算$x^n$：

```python
def power(x, n):
    s = 1
    while n > 0:
        n = n - 1
        s = s * x
    return s
```

修改后的`power(x, n)`函数，可以计算任意n次方，其中参数`x`和`n`都是位置参数：

```python
>>> power(5, 2)
25
>>> power(5, 3)
125
```

### 默认参数

由于我们经常计算$x^2$，所以，完全可以把第二个参数n的默认值设定为2：

```python
def power(x, n=2):
    s = 1
    while n > 0:
        n = n - 1
        s = s * x
    return s
```

这样，当我们调用`power(5)`时，相当于调用`power(5, 2)`：

```python
>>> power(5)
25
>>> power(5, 2)
25
```

而对于n > 2的其他情况，就必须明确地传入n，比如`power(5, 3)`。

设置默认参数时，有几点要注意：

一是必选参数在前，默认参数在后；

二是当函数有多个参数时，把变化大的参数放前面，变化小的参数放后面，变化小的参数就可以作为默认参数。

**使用默认参数最大的好处是能降低调用函数的难度，并且默认参数必须指向不变对象！**

例如写个小学生注册信息的函数，需要传入姓名、性别、年龄、城市参数，可以把年龄和城市设为默认参数：

```python
def enroll(name, gender, age=6, city='Beijing'):
    print('name:', name)
    print('gender:', gender)
    print('age:', age)
    print('city:', city)
```

这样大多数学生注册时不需要提供年龄和城市等信息，只提供必须的两个参数：

```python
>>> enroll('Sarah', 'F')
name: Sarah
gender: F
age: 6
city: Beijing
```

只有与默认参数不符的学生才需要提供额外的信息：

```python
enroll('Bob', 'M', 7)
enroll('Adam', 'M', city='Tianjin')
```

可见，默认参数降低了函数调用的难度，而一旦需要更复杂的调用时，又可以传递更多的参数来实现。无论是简单调用还是复杂调用，函数只需要定义一个。

### 可变参数

顾名思义，可变参数就是传入的参数个数是可变的，可以是1个、2个到任意个，还可以是0个。

例如，给定一组数字a，b，c……，请计算$a^2$ + $b^2$ + $c^2$ + ……。

要定义这个函数，必须确定输入的参数。由于参数个数不确定，我们首先想到可以把a，b，c……作为一个list或tuple传进来，这样，函数可以定义如下：

```python
def calc(numbers):
    sum = 0
    for n in numbers:
        sum = sum + n * n
    return sum
```

但是调用的时候，需要先给出一个list或tuple：

```python
>>> calc([1, 2, 3])
14
>>> calc((1, 3, 5, 7))
84
```

如果利用可变参数，调用函数的方式可以简化成这样：

```python
>>> calc(1, 2, 3)
14
>>> calc(1, 3, 5, 7)
84
```

所以，我们把函数的参数改为可变参数：

```python
def calc(*numbers):
    sum = 0
    for n in numbers:
        sum = sum + n * n
    return sum
```

定义可变参数和定义一个list或tuple参数相比，仅仅在参数前面加了一个`*`号。在函数内部，参数`numbers`接收到的是一个tuple，因此，函数代码完全不变。但是，调用该函数时，可以传入任意个参数，包括0个参数：

```python
>>> calc(1, 2)
5
>>> calc()
0
```

如果已经有一个list或者tuple，可以在前面加一个`*`号，把list或tuple的元素变成可变参数传进去：

```python
>>> nums = [1, 2, 3]
>>> calc(*nums)
14
```

`*nums`表示把`nums`这个list的所有元素作为可变参数传进去。这种写法相当有用，而且很常见。

### 关键字参数

可变参数允许你传入0个或任意个参数，这些可变参数在函数调用时自动组装为一个tuple。而关键字参数允许你传入0个或任意个含参数名的参数，这些关键字参数在函数内部为一个dict。

```python
def person(name, age, **kw):
    print('name:', name, 'age:', age, 'other:', kw)
```

函数`person`除了必选参数`name`和`age`外，还接受关键字参数`kw`。在调用该函数时，可以只传入必选参数：

```python
>>> person('Michael', 30)
name: Michael age: 30 other: {}
```

也可以传入任意个数的关键字参数：

```python
>>> person('Bob', 35, city='Beijing')
name: Bob age: 35 other: {'city': 'Beijing'}
>>> person('Adam', 45, gender='M', job='Engineer')
name: Adam age: 45 other: {'gender': 'M', 'job': 'Engineer'}
```

关键字参数可以扩展函数的功能。比如，在`person`函数里，我们保证能接收到`name`和`age`这两个参数，但是，如果调用者愿意提供更多的参数，我们也能收到。

和可变参数类似，也可以先给出一个dict，然后，把该dict转换为关键字参数传进去：

```python
>>> extra = {'city': 'Beijing', 'job': 'Engineer'}
>>> person('Jack', 24, city=extra['city'], job=extra['job'])
name: Jack age: 24 other: {'city': 'Beijing', 'job': 'Engineer'}
```

当然，上面复杂的调用可以用简化的写法：

```python
>>> extra = {'city': 'Beijing', 'job': 'Engineer'}
>>> person('Jack', 24, **extra)
name: Jack age: 24 other: {'city': 'Beijing', 'job': 'Engineer'}
```

`**extra`表示把`extra`这个dict的所有key-value用关键字参数传入到函数的`**kw`参数，`kw`将获得一个dict，注意`kw`获得的dict是`extra`的一份拷贝，对`kw`的改动不会影响到函数外的`extra`。

### 命名关键字参数

如果要限制关键字参数的名字，可以用命名关键字参数，例如，只接收city和job作为关键字参数。

函数定义如下：

```python
def person(name, age, *, city, job):
    print(name, age, city, job)
```

和关键字参数`**kw`不同，命名关键字参数需要一个特殊分隔符`*`，`*`后面的参数被视为命名关键字参数。

调用方式如下：

```python
>>> person('Jack', 24, city='Beijing', job='Engineer')
Jack 24 Beijing Engineer
```

如果函数定义中已经有了一个可变参数，后面跟着的命名关键字参数就不再需要一个特殊分隔符`*`了：

```python
def person(name, age, *args, city, job):
    print(name, age, args, city, job)
```

命名关键字参数必须传入参数名，这和位置参数不同。

```python
>>> person('Jack', 24, city='Beijing', job='Engineer')
Jack 24 () Beijing Engineer
```

命名关键字参数可以有缺省值，从而简化调用：

```python
def person(name, age, *, city='Beijing', job):
    print(name, age, city, job)
```

由于命名关键字参数`city`具有默认值，调用时，可不传入`city`参数：

```python
>>> person('Jack', 24, job='Engineer')
Jack 24 Beijing Engineer
```

### 参数组合

在Python中定义函数，可以用必选参数、默认参数、可变参数、关键字参数和命名关键字参数，这5种参数都可以组合使用。但是请注意，参数定义的顺序必须是：必选参数、默认参数、可变参数、命名关键字参数和关键字参数。

对于任意函数，都可以通过类似`func(*args, **kw)`的形式调用它，无论它的参数是如何定义的，其中`*args`是可变参数，接收的是一个tuple，`**kw`是关键字参数，接收的是一个dict。

## 递归函数

在函数内部，可以调用其他函数。如果一个函数在内部调用自身，这个函数就是递归函数。

计算阶乘`n! = 1 x 2 x 3 x ... x n`，用函数`fact(n)`表示，可以看出：

fact(n) = n! = 1 x 2 x 3 x ... x (n-1) x n = (n-1)! x n = fact(n-1) x n

所以，`fact(n)`可以表示为`n x fact(n-1)`，只有n=1时需要特殊处理。

于是，`fact(n)`用递归的方式写出来就是：

```python
def fact(n):
    if n==1:
        return 1
    return n * fact(n - 1)
```

计算`fact(5)`，可以根据函数定义看到计算过程如下：

```python
===> fact(5)
===> 5 * fact(4)
===> 5 * (4 * fact(3))
===> 5 * (4 * (3 * fact(2)))
===> 5 * (4 * (3 * (2 * fact(1))))
===> 5 * (4 * (3 * (2 * 1)))
===> 5 * (4 * (3 * 2))
===> 5 * (4 * 6)
===> 5 * 24
===> 120
```

递归函数的优点是定义简单，逻辑清晰，但是需要注意防止栈溢出，可通过尾递归优化解决，事实上尾递归和循环的效果是一样的，因为Python标准的解释器没有针对尾递归做优化，所以任何递归函数都存在栈溢出的问题。

实例：汉诺塔，编写`hanoi(n, a, b, c)`函数，它接收参数n，表示3个柱子A、B、C中第1个柱子A的盘子数量，然后打印出把所有盘子从A借助B移动到C的方法：

```python
def hanoi(n, a, b, c):
    if n == 1:
        print(a, '-->', c)
    else:
        hanoi(n-1, a, c, b)
        hanoi(1, a, b, c)
        hanoi(n-1, b, a, c)

hanoi(4, 'a', 'b', 'c')
```

