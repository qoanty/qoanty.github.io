---
title: "Python3 数据结构"
date: 2019-04-25T15:36:02+08:00
tags: ["python", "data"]
categories: ["Learn", "Python"]
draft: false
---

Python支持以下数据结构：列表list，字典dict，元组tuple，集合set。

使用字典：

- 需要键：值对之间的逻辑关联时。
- 需要基于自定义密钥快速查找数据时。
- 数据不断修改时，字典是可变的。

使用其他类型：

- 不需要随机访问的数据集合，请使用列表。当你需要一个简单的，可迭代的频繁修改的集合可以使用列表。
- 需要元素的唯一性，使用集合。
- 当数据无法更改时使用元组。

很多时候，元组与字典结合使用，例如元组可能代表一个关键字，因为它是不可变的。

列表、字典、元组、集合的区别以及各自的使用方法区别：

1. 列表

列表是处理一组有序项目的数据结构，一旦创建了一个列表，就可以添加，删除，搜索列表中的项目。列表是可变的数据类型，并且列表是可以嵌套的。

2. 元组

元祖和列表十分相似，不过元组是不可变的。元组通常用在使语句或用户定义的函数能够安全的采用一组值的时候，即被使用的元组的值不会改变。元组可以嵌套。

3. 字典

字典类似于通过联系人名称查找地址和联系人详细情况的地址簿，键必须是唯一的，键/值对是没有顺序的。

4. 集合

与字典类似，但只包含键，而没有对应的值，包含的数据不重复，重复的值在集合中只存在一个。

## 转换操作

1、字典

```python
>>> dict = {'name': 'Zara', 'age': 7, 'class': 'First'}
>>> print(type(str(dict)), str(dict))
<class 'str'> {'name': 'Zara', 'age': 7, 'class': 'First'}
>>> print(tuple(dict))
('name', 'age', 'class')
>>> print(tuple(dict.values()))
('Zara', 7, 'First')
>>> print(list(dict))
['name', 'age', 'class']
>>> print(list(dict.values()))
['Zara', 7, 'First']
```

2、元组

```python
>>> tup = (1, 2, 3, 4, 5)
>>> print(tup.__str__())
(1, 2, 3, 4, 5)
>>> print(list(tup))
[1, 2, 3, 4, 5]
```

3、列表

```python
>>> nums = [6, 7, 8, 9, 10]
>>> print(str(nums))
[6, 7, 8, 9, 10]
>>> print(tuple(nums))
(6, 7, 8, 9, 10)
```

4、字符串

```python
>>> print(tuple(eval('(11,12,13)')))
(11, 12, 13)
>>> print(list(eval('(11,12,13)')))
[11, 12, 13]
>>> print(type(eval("{'name':'ljq', 'age':24}")))
<class 'dict'>
```

## 字符串操作

```python
# 去掉空格和特殊符号
name1 = name.strip()
# 去掉左边的空格和换行符
name2 = name.lstrip()
# 去掉右边的空格和换行符
name3 = name.rstrip()
# 字符串的搜索和替换
name.count('e')      # 查找某个字符在字符串中出现的次数
name.capitalize()    # 首字母大写
name.center(100,'-') # 把字符串方中间,两边用-补齐,100表示占位多少
name.find('a')       # 找到字符返回下标，多个时返回第一个，不存在时返回-1
name.index('a')      # 找到字符返回下标，多个时返回第一个，不存在时报错
name.replace('abc','123') # 字符串的替换
# 字符串的测试和替换函数
name.startswith("abc") # 是否以abc开头
name.endswith("def")   # 是否以def结尾
name.isalnum() # 是否全是字母和数字，并且至少包含一个字符
name.isalpha() # 是否全是字母，并至少包含一个字符
name.isdigit() # 是否全是数字，并且至少包含一个字符
name.isspace() # 是否全是空白字符，并且至少包含一个字符
name.islower() # 是否全是小写
name.isupper() # 是否全是大写
name.istitle() # 是否是首字母大写
# 字符串的分割
name.split('') # 默认按照空格进行分隔，从前往后分隔
name.rsplit()  # 从后往前进行分隔
# 连接字符串
'.'.join(name) # 用.号将一个可迭代的序列拼接起来
# 截取字符串(切片)
name = 'The quick brown fox jumps over the lazy dog'
name1 = name[0:3]    # 第一位到第三位的字符，和range一样不包含结尾索引
name2 = name[:]      # 截取全部的字符
name3 = name[6:]     # 截取第6个字符到结尾
name4 = name[:-3]    # 截取从开头到最后一个字符之前
name5 = name[-1]     # 截取最后一个字符
name6 = name[::-1]   # 创造一个与原字符串顺序相反的字符串
name7 = name[:-5:-1] # 逆序截取
```

## 列表操作

```python
# 创建一个列表
list1 = ['1', '2', '3', '4']
list2 = list("1234")
print(list1, list2)
print(list1 == list2) # 以上创建的两个列表是等价的
# 添加新元素，末尾追加
a = [1, 2, 3, 4, 5]
a.append(6)
# 指定位置的前面插入一个元素
a.insert(2, 100)
# 扩展列表list.extend(iterable)
a.extend([10, 11, 12])
# 遍历列表
for i in a:
    print(i)
for index, i in enumerate(a):
    print(i, index)
# 访问列表中的值
print(a[2])
# 删除元素
a.remove(2)
# 删除指定位置的元素，默认为最后一个元素
a.pop()
a.pop(2)
print(a)
# 删除列表或指定元素或者列表切片，list删除后无法访问
a = [1, 2, 3, 4, 5, 6]
del a[1]
del a[1:]
del a
# 排序和反转列表
a.sort()
a.reverse()
# 列表的截取
L = ['spam','Spam','SPAM!']
print(L[-1]) # ['SPAM']
# 列表操作的函数和方法
len(a)  # 列表元素的个数
max(a)  # 返回列表元素最大值
min(a)  # 返回列表元素最小值
list(tuple) #将一个可迭代对象转换为列表
```

## 字典操作

```python
# 创建一个空字典
D = {}  或 D = dict()
# 键构建值都为None的字典
D = dict.fromkeys(['a','b','c'])  ==> {'a':None,'b':None,'c':None}
# 通过zip函数构建字典
D = dict(zip(keyslist,valueslist))
# 通过赋值表达式元组构造字典
D = dict(name='Bob', age=42)  ==> {'name': 'Bob', 'age': 42}
# 成员关系判断
if key in D:
  pass
# 列出所有的键/值
D.keys()  D.values()  D.items()
# 拷贝一个字典的副本
D1 = D.copy()
# 合并更新字典
D.update(D2)
# 删除字典以及长度
D.pop(key)  len(D)  del D[key]
# 新增或者是修改键对应的值
D[key] = value
# 字典键的列表、值的列表
list(D.keys())  list(D.values())
```

## 元组操作

```python
# 创建空元组
t1 = ()
t2 = tuple()
# 单个元素的元组
x = (40,)
# 多个元素的元组（括号可以省略）
T  = (1, 2, 3)
# 用一个可迭代对象生成元组
T = tuple('abc')
# 元组的访问
T[i]  T[i][j]
# 合并和重复
T1 + T2  T * 3
# 迭代和成员关系
for x in T:
  print(x)
'spam' in T
# 对元组进行排序
T = ('c','a','d','b')
tmp = list(T)
tmp.sort()  ==> ['a','b','c','d']
T = tunple(tmp)
sorted(T)
```

## 集合操作

```python
# 创建空集合
a = set()
# 用可迭代对象创建一个集合
b = set(iterable)
# 集合的运算
& 生成两个集合的交集 s3 = s1 & s2
| 生成两个集合的并集 s3 = s1 | s2
- 生成两个集合的补集 s3 = s1 - s2
^ 对称补集 s3 = s1 ^ s2 (只能属于s1或者s2，也就是去除掉s1和s2重合的部分)
```
