---
title: "Python3 学习笔记（内建模块）"
date: 2019-04-21T11:58:09+08:00
tags: ["python", "functions"]
categories: ["Learn", "Python"]
draft: false
---

## 常用内建模块

### datetime

**获取当前日期和时间**

```python
>>> from datetime import datetime
>>> now = datetime.now()  # 获取当前datetime
>>> print(now)
2019-05-15 19:34:57.142524
>>> print(type(now))
<class 'datetime.datetime'>
>>>
```

注意到`datetime`是模块，`datetime`模块还包含一个`datetime`类，通过`from datetime import datetime`导入的才是`datetime`这个类，如果仅导入`import datetime`，则必须引用全名`datetime.datetime`，`datetime.now()`返回当前日期和时间，其类型是`datetime`。

**获取指定日期和时间**

```python
>>> from datetime import datetime
>>> dt = datetime(2019, 4, 18, 17, 20)   # 用指定日期时间创建datetime
>>> print(dt)
2019-04-18 17:20:00
```

**datetime转换为timestamp**

我们把1970年1月1日 00:00:00 UTC+00:00时区的时刻称为epoch time，记为0（1970年以前的时间timestamp为负数），当前时间就是相对于epoch time的秒数，称为timestamp，与时区毫无关系。

```python
>>> from datetime import datetime
>>> dt = datetime(2019, 4, 18, 17, 20)   # 用指定日期时间创建datetime
>>> dt.timestamp()  # 把datetime转换为timestamp
1555579200.0
```

**timestamp转换为datetime**

```python
>>> from datetime import datetime
>>> t = 1555579200.0
>>> print(datetime.fromtimestamp(t))
2019-04-18 17:20:00
```

**str转换为datetime**

```python
>>> from datetime import datetime
>>> cday = datetime.strptime('2019-4-20 15:19:53', '%Y-%m-%d %H:%M:%S')
>>> print(cday)
2019-04-20 15:19:53
```

**datetime转换为str**

```python
>>> from datetime import datetime
>>> now = datetime.now()
>>> print(now.strftime('%a, %b %d %H:%M'))
Sat, Apr 26 15:20
```

**datetime加减**

```python
>>> from datetime import datetime, timedelta
>>> now = datetime.now()
>>> now
datetime.datetime(2019, 4, 20, 15, 7, 45, 249705)
>>> now + timedelta(hours=10)
datetime.datetime(2019, 4, 21, 1, 7, 45, 249705)
>>> now - timedelta(days=1)
datetime.datetime(2019, 4, 21, 15, 7, 45, 249705)
>>> now + timedelta(days=2, hours=12)
datetime.datetime(2019, 4, 23, 3, 7, 45, 249705)
```

### collections

collections是Python内建的一个集合模块，提供了许多有用的集合类。

**namedtuple**

`namedtuple`是一个函数，它可以创建一个自定义的`tuple`对象，并且规定了`tuple`元素的个数，可以用属性而不是索引来引用`tuple`的某个元素，而且创建的`Point`对象是`tuple`的一种子类：

```python
>>> from collections import namedtuple
>>> Point = namedtuple('Point', ['x', 'y'])
>>> p = Point(1, 2)
>>> p.x
1
>>> p.y
2
>>> isinstance(p, Point)
True
>>> isinstance(p, tuple)
True
# namedtuple('名称', [属性list]):
Circle = namedtuple('Circle', ['x', 'y', 'r'])
```

**deque**

`deque`是为了高效实现插入和删除操作的双向列表，适合用于队列和栈，除了实现`list`的`append()`和`pop()`外，还支持`appendleft()`和`popleft()`，这样就可以非常高效地往头部添加或删除元素：

```python
>>> from collections import deque
>>> q = deque(['a', 'b', 'c'])
>>> q.appten('x')
>>> q.appendleft('y')
>>> q
deque(['y', 'a', 'b', 'c', 'x'])
```

**defaultdict**

使用`dict`时，若希望key不存在时，返回一个默认值，就可以用`defaultdict`，默认值是调用函数返回的，而函数在创建`defaultdict`对象时传入：

```python
>>> from collections import defaultdict
>>> dd = defaultdict(lambda: 'N/A')
>>> dd['key1'] = 'abc'
>>> dd['key1'] # key1存在
'abc'
>>> dd['key2'] # key2不存在，返回默认值
'N/A'
```

**OrderedDict**

使用`dict`时，Key是无序的，若要保持Key的顺序，可以`用OrderedDict`，Key会按照插入的顺序排列，不是Key本身排序：

```python
>>> from collections import OrderedDict
>>> d = dict([('a', 1), ('b', 2), ('c', 3)])
>>> d # dict的Key是无序的
{'a': 1, 'c': 3, 'b': 2}
>>> od = OrderedDict([('a', 1), ('b', 2), ('c', 3)])
>>> od # OrderedDict的Key是有序的
OrderedDict([('a', 1), ('b', 2), ('c', 3)])
```

**Counter**

`Counter`是一个简单的计数器也是`dict`的一个子类，例如统计字符出现的个数：

```python
>>> from collections import Counter
>>> c = Counter() # c = Counter('programming')
>>> for ch in 'programming':
...     c[ch] = c[ch] + 1
...
>>> c
Counter({'r': 2, 'g': 2, 'm': 2, 'p': 1, 'o': 1, 'a': 1, 'i': 1, 'n': 1})
```

### base64

```python
>>> import base64
>>> base64.b64encode(b'binary\x00string')
b'YmluYXJ5AHN0cmluZw=='
>>> base64.b64decode(b'YmluYXJ5AHN0cmluZw==')
b'binary\x00string'
>>> base64.b64encode(b'i\xb7\x1d\xfb\xef\xff')
b'abcd++//'
>>> base64.urlsafe_b64encode(b'i\xb7\x1d\xfb\xef\xff')
b'abcd--__'
>>> base64.urlsafe_b64decode('abcd--__')
b'i\xb7\x1d\xfb\xef\xff'
```

**struct**

`struct`模块来解决`bytes`和其他二进制数据类型的转换。`pack`函数把任意数据类型变成`bytes`，`unpack`把`bytes`变成相应的数据类型：

```python
>>> import struct
>>> struct.pack('>I', 10240099)
b'\x00\x9c@c'
>>> struct.unpack('>IH', b'\xf0\xf0\xf0\xf0\x80\x80')
(4042322160, 32896)
```

根据`>IH`的说明，后面的`bytes`依次变为`I`：4字节无符号整数和`H`：2字节无符号整数。

**hashlib**

hashlib提供了常见的摘要算法，如MD5，SHA1等。

```python
>>> import hashlib
>>> md5 = hashlib.md5()
>>> md5.update('how to use md5 in python hashlib?'.encode('utf-8'))
>>> print(md5.hexdigest())
d26a53750bc40b38b65a520292f69306
>>> sha1 = hashlib.sha1()
>>> sha1.update('how to use sha1 in '.encode('utf-8'))
>>> sha1.update('python hashlib?'.encode('utf-8'))
>>> print(sha1.hexdigest())
2c76b57293ce30acef38d98f6046927161b46a44
```

**hmac**

Hmac算法：Keyed-Hashing for Message Authentication。它通过一个标准算法，在计算哈希的过程中，把key混入计算过程中。

```python
>>> import hmac
>>> message = b'Hello, world!'
>>> key = b'secret'
>>> h = hmac.new(key, message, digestmod='MD5')
>>> h.hexdigest()
'fa4ee7d173f2d97ee79022d1a7355bcf'
```

验证用户口令：

```python
import hmac, random

def hmac_md5(key, s):
    return hmac.new(key.encode('utf-8'), s.encode('utf-8'), 'MD5').hexdigest()

class User(object):
    def __init__(self, username, password):
        self.username = username
        self.key = ''.join([chr(random.randint(48, 122)) for i in range(20)])
        self.password = hmac_md5(self.key, password)

db = {
    'michael': User('michael', '123456'),
    'bob': User('bob', 'abc999'),
    'alice': User('alice', 'alice2008')
}

def login(username, password):
    user = db[username]
    return user.password == hmac_md5(user.key, password)

# 测试:
assert login('michael', '123456')
assert login('bob', 'abc999')
assert login('alice', 'alice2008')
assert not login('michael', '1234567')
assert not login('bob', '123456')
assert not login('alice', 'Alice2008')
print('ok')
```

所有的第三方模块都会在[PyPI](https://pypi.python.org/)注册，只要找到对应的模块名字，即可用pip安装。

