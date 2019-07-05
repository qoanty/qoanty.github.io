---
title: "Python3 学习笔记（面向对象编程）"
date: 2019-04-20T11:39:49+08:00
tags: ["python", "OOP"]
categories: ["Learn", "Python"]
draft: false
---

## 面向对象编程

面向对象编程——Object Oriented Programming，简称OOP，是一种程序设计思想。OOP把对象作为程序的基本单元，一个对象包含了数据和操作数据的函数。

面向对象的程序设计把计算机程序视为一组对象的集合，而每个对象都可以接收其他对象发过来的消息，并处理这些消息，计算机程序的执行就是一系列消息在各个对象之间传递。

在Python中，所有数据类型都可以视为对象，当然也可以自定义对象。自定义的对象数据类型就是面向对象中的类（Class）的概念。

**类和实例**

面向对象最重要的概念就是类（Class）和实例（Instance），类是抽象的模板，而实例是根据类创建出来的一个个具体的“对象”，每个对象都拥有相同的方法，但各自的数据可能不同。

在Python中，定义类是通过`class`关键字，后面紧接着是类名，创建实例是通过类名+()实现的，通过定义一个特殊的`__init__`方法，可以在创建实例的时候把认为必须绑定的属性填写进去。

`__init__`方法的第一个参数永远是`self`，表示创建的实例本身，因此，在`__init__`方法内部，就可以把各种属性绑定到`self`，因为`self`就指向创建的实例本身。

普通的函数相比，在类中定义的函数第一个参数永远是实例变量`self`，并且，调用时不用传递该参数。除此之外，类的方法和普通函数没有什么区别，你仍然可以用默认参数、可变参数、关键字参数和命名关键字参数。

在类中每个实例拥有各自的数据，要访问这些数据可以直接在类的内部定义访问数据的函数，这样，就把“数据”给封装起来了。这些封装数据的函数是和类本身是关联起来的，称之为类的方法。

**访问限制**

在Class内部，可以有属性和方法，而外部代码可以通过直接调用实例变量的方法来操作数据，这样就隐藏了内部的复杂逻辑。但是外部代码还是可以自由地修改一个实例的属性，如果要让内部属性不被外部访问，可以把属性的名称前加上两个下划线`__`，在Python中，实例的变量名如果以`__`开头，就变成了一个私有变量（private），只有内部可以访问，外部不能访问，通过访问限制的保护使得代码更加健壮，如果需要外部访问内部属性，可以通过增加类的方法实现。

```python
class Student(object):

    def __init__(self, name, score):
        self.__name = name
        self.__score = score

    def print_score(self):
        print('%s: %s' % (self.__name, self.__score))

    def get_name(self):
        return self.__name

    def get_score(self):
        return self.__score

    def set_score(self, score):
        if 0 <= score <= 100:
            self.__score = score
        else:
            raise ValueError('bad score')
```

**继承和多态**

在OOP程序设计中，当我们定义一个class的时候，可以从某个现有的class继承，新的class称为子类（Subclass），而被继承的class称为基类、父类或超类（Base class、Super class）。

继承的好处是子类可以获得了父类的全部功能，当子类和父类存在相同的方法时，在代码运行的时候，总是会调用子类的方法，这是继承的另一个好处：多态。

**获取对象信息**

使用`type()`函数来判断对象类型，使用`isinstance()`函数来判断class的类型，能用`type()`判断的基本类型也可以用`isinstance()`判断。

如果要获得一个对象的所有属性和方法，可以使用`dir()`函数，它返回一个包含字符串的list。

**实例属性和类属性**

由于Python是动态语言，根据类创建的实例可以任意绑定属性。给实例绑定属性的方法是通过实例变量，或者通过`self`变量。如果类本身需要绑定一个属性，可以直接在class中定义属性，这种属性是类属性，但类的所有实例都可以访问到。

## 面向对象高级编程

**使用__slots__**

动态绑定允许我们在程序运行的过程中动态给class加上功能，这在静态语言中很难实现，可以通过一个特殊的`__slots__`变量，来限制该class实例添加的属性，但对继承的子类不起作用。

**使用@property**

Python内置的`@property`装饰器，可以把一个`getter`方法变成属性，同时又创建了另一个装饰器`@score.setter`，负责把一个`setter`方法变成属性赋值，只定义getter方法，不定义setter方法就是一个只读属性。

```python
class Student(object):

    @property
    def birth(self):
        return self._birth

    @birth.setter
    def birth(self, value):
        self._birth = value

    @property
    def age(self):
        return 2019 - self._birth
```

上面的`birth`是可读写属性，而`age`就是一个只读属性，因为`age`可以根据`birth`和当前时间计算出来。

**多重继承**

通过多重继承，一个子类就可以同时获得多个父类的所有功能。在设计类的继承关系时，通常主线都是单一继承下来的，但是如果需要“混入”额外的功能，通过多重继承就可以实现，这种设计通常称之为MixIn。

MixIn的目的是给一个类增加多个功能，这样在设计类的时候，我们优先考虑通过多重继承来组合多个MixIn的功能，而不是设计多层次的复杂的继承关系。

**定制类**

`__str__()`、`__repr__()`方法：返回一个好看的字符串。

```python
class Student(object):
    def __init__(self, name):
        self.name = name
    def __str__(self):
        return 'Student object (name=%s)' % self.name
    __repr__ = __str__

print(Student('Michael'))
```

`__iter__()`方法：返回一个迭代对象。

```python
class Fib(object):
    def __init__(self):
        self.a, self.b = 0, 1 # 初始化两个计数器a，b

    def __iter__(self):
        return self # 实例本身就是迭代对象，故返回自己

    def __next__(self):
        self.a, self.b = self.b, self.a + self.b # 计算下一个值
        if self.a > 100000: # 退出循环的条件
            raise StopIteration()
        return self.a # 返回下一个值

for n in Fib():
    print(n)
```

`__getitem__()`、`__setitem__()`、`__delitem__()`方法：把对象视作list或dict来获取某个元素值、对某个元素合赋值及删除某个元素。

`__getattr__()`方法：动态返回一个属性。`__call__()`方法：直接对实例进行调用。通过`callable()`函数，可以判断一个对象是否是“可调用”对象。

```python
class Chain(object):
    def __init__(self, path=''):
       self.__path = path

    def __getattr__(self, path):
       return Chain('%s/%s' % (self.__path, path))

    def __call__(self, path):
       return Chain('%s/%s' % (self.__path, path))

    def __str__(self):
       return self.__path

    __repr__ = __str__

print(Chain().status.user.timeline.list)
print(Chain().users('michael').repos) # /users/michael/repos
```

每次调用类`Chain()`都会运行里面的`__init__()`函数进行初始化里面的相应属性，在这里边相当于重建实例覆盖之前的实例，具体来看：

- `Chain()`：初始化一个实例
- `Chain().users`：由于没有给实例传入初始化对应属性的具体信息，从而自动调用`__getattr__()`函数，从而有`Chain().users = Chain('/users')`，这是重建实例
- `Chain().users('michael') = Chain('/users')('michael')`：这是对实例直接调用，相当于调用普通函数，返回的是`Chain('/users/michael')`，再一次重建实例，覆盖掉`Chain('/users')`
- `Chain().users('michael').repos`：是查询实例的属性`repos`，由于没有这一属性就会执行`__getattr__()`函数，再一次返回新的实例`Chain('/users/michael/repos')`并且覆盖点之前的实例，并且一但定义了一个新的实例，就会执行`__init__`方法
- `print(Chain().users('michael').repos)`：由于我们定义了`__str__()`方法，那么打印的时候就会调用此方法，打印的是`path`属性，即`/users/michael/repos`

**使用枚举类**

`Enum`类可以把一组相关常量定义在一个class中，且class不可变，而且成员可以直接比较。例如，定义一个`Month`类型的枚举类，可以直接使用`Month.Jan`来引用一个常量，或者枚举它的所有成员：

```python
>>> from enum import Enum
>>> Month = Enum('Month', ('Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'))
>>> Month.Jan
<Month.Jan: 1>
>>> for name, member in Month.__members__.items():
...     print(name, '=>', member, ',', member.value)
...
Jan => Month.Jan , 1
Feb => Month.Feb , 2
Mar => Month.Mar , 3
Apr => Month.Apr , 4
May => Month.May , 5
Jun => Month.Jun , 6
Jul => Month.Jul , 7
Aug => Month.Aug , 8
Sep => Month.Sep , 9
Oct => Month.Oct , 10
Nov => Month.Nov , 11
Dec => Month.Dec , 12
```

`value`属性则是自动赋给成员的`int`常量，默认从1开始计数。

如果需要更精确地控制枚举类型，可以从`Enum`派生出自定义类：

```python
from enum import Enum, unique

@unique
class Weekday(Enum):
    Sun = 0 # Sun的value被设定为0
    Mon = 1
    Tue = 2
    Wed = 3
    Thu = 4
    Fri = 5
    Sat = 6

for date in Weekday:
    print(date.name, '=>', date, ':', date.value)
```

`@unique`装饰器可以帮助我们检查保证没有重复值。

**使用元类**

动态语言和静态语言最大的不同，就是函数和类的定义，不是编译时定义的，而是运行时动态创建的。`type()`函数可以查看一个类或变量的类型，还可以创建一个class对象。

除了使用`type()`动态创建类以外，还可以使用metaclass，直译为元类，简单的解释就是：当我们定义了类以后，就可以根据这个类创建出实例，所以：先定义类，然后创建实例，如果要想创建出类就必须根据metaclass创建，所以：先定义metaclass，然后创建类，也就是：先定义metaclass，就可以创建类，最后创建实例。

## 错误、调试

**错误处理**

高级语言通常都内置了一套`try...except...finally...`的错误处理机制，使用`try...except`捕获错误的一个巨大的好处就是可以跨越多层调用，不需要在每个可能出错的地方去捕获错误，只要在合适的层次去捕获错误就可以了。

**调试**

第一种方法简单直接粗暴有效，就是用`print()`把可能有问题的变量打印出来；第二种方法是用断言（assert）来替代`print()`，可以用`-O`参数来关闭`assert`；第三种方式把`print()`替换为`logging`，和`assert`比`logging`不会抛出错误，而且可以输出到文件，还能指定记录信息的级别，有`debug`，`info`，`warning`，`error`等几个级别，也不用删除，最后统一控制输出哪个级别的信息；第四种方式是启动Python的调试器pdb，让程序以单步方式运行，可以随时查看运行状态，此外还可以`import pdb`，然后在可能出错的地方设置一个`pdb.set_trace()`断点，使用命令`p`查看变量，或者用命令`c`继续运行。

