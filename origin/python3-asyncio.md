---
title: "Python3 学习笔记（异步IO）"
date: 2019-04-21T15:01:59+08:00
tags: ["python", "asyncio"]
categories: ["Learn", "Python"]
draft: false
---

由于CPU的速度远远快于磁盘、网络等IO，在一个线程中CPU执行代码的速度极快，一旦遇到IO操作，如读写文件、发送网络数据时，就需要等待IO操作完成，才能继续进行下一步操作，这种情况称为同步IO。

异步IO是当代码需要执行一个耗时的IO操作时，它只发出IO指令，并不等待IO结果，然后就去执行其他代码了，一段时间后，当IO返回结果时，再通知CPU进行处理。

同步：就是发出一个“调用”时，在没有得到结果之前，该“调用”就不返回，“调用者”需要一直等待该“调用”结束，才能进行下一步工作。

异步：“调用”在发出之后，就直接返回了，也就没有返回结果。“被调用者”完成任务后，通过状态来通知“调用者”继续回来处理该“调用”。

协程，又称微线程，纤程，英文名Coroutine。在执行过程中，在子程序内部可中断，然后转而执行别的子程序，在适当的时候再返回来接着执行。

Python对协程的支持是通过`generator`（生成器）实现的。在Python3.4中，协程都是通过使用`yield from`和`asyncio`模块中的@asyncio.coroutine来实现的。`asyncio`专门被用来实现异步IO操作。

生成器中的关键字`yield`可以实现中断功能，可以传出值，也可以从函数外部接收值，而`yield from`的实现就是简化了`yield`操作。

```python
def generator_1(titles):
    yield titles

def generator_2(titles):
    yield from titles

titles = ['Python','Java','C++']

for title in generator_1(titles):
    print('生成器1:',title)

for title in generator_2(titles):
    print('生成器2:',title)
```

执行结果如下：

```python
生成器1: ['Python', 'Java', 'C++']
生成器2: Python
生成器2: Java
生成器2: C++
```

`yield from`还有一个主要功能是省去了很多异常的处理，其内部已经实现大部分异常处理。

在协程中，只要是和IO任务类似的、耗费时间的任务都可以使用`yield from`来进行中断，达到异步功能。

```python
import time
import asyncio

@asyncio.coroutine  # 标志协程的装饰器
def taskIO_1():
    print('开始运行IO任务1...')
    yield from asyncio.sleep(2)  # 假设该任务耗时2s
    print('IO任务1已完成，耗时2s')
    return taskIO_1.__name__

@asyncio.coroutine  # 标志协程的装饰器
def taskIO_2():
    print('开始运行IO任务2...')
    yield from asyncio.sleep(3)  # 假设该任务耗时3s
    print('IO任务2已完成，耗时3s')
    return taskIO_2.__name__

@asyncio.coroutine  # 标志协程的装饰器
def main():  # 调用方
    tasks = [taskIO_1(), taskIO_2()]  # 把所有任务添加到task中
    done, pending = yield from asyncio.wait(tasks)  # 子生成器
    for r in done:  # done和pending都是一个任务，所以返回结果需要逐个调用result()
        print('协程无序返回值：' + r.result())

if __name__ == '__main__':
    start = time.time()
    loop = asyncio.get_event_loop()  # 创建一个事件循环对象loop
    try:
        loop.run_until_complete(main())  # 完成事件循环，直到最后一个任务结束
    finally:
        loop.close()  # 结束事件循环
    print('所有IO任务总耗时%.2f秒' % float(time.time()-start))
```

执行结果如下：

```python
开始运行IO任务1...
开始运行IO任务2...
IO任务1已完成，耗时2s
IO任务2已完成，耗时3s
协程无序返回值：taskIO_2
协程无序返回值：taskIO_1
所有IO任务总耗时3.02秒
```

执行过程：

1. 首先通过`get_event_loop()`获取了一个标准事件循环loop（因为是一个，所以协程是单线程）。

2. 然后通过`run_until_complete(main())`来运行协程（此处把调用方协程main()作为参数，调用方负责调用其他委托生成器），该函数的特点是直到循环事件的所有事件都处理完才能完整结束。

3. 进入调用方协程，把多个任务`taskIO_1()`和`taskIO_2()`放到一个`task`列表中，可理解为打包任务。

4. 现在使用`asyncio.wait(tasks)`来获取一个`awaitable objects`，即可等待对象的集合（此处的aws是协程的列表），并发运行传入的aws，同时通过`yield from`返回一个包含`(done, pending)`的元组，`done`表示已完成的任务列表，`pending`表示未完成的任务列表；如果使用`asyncio.as_completed(tasks)`则会按完成顺序生成协程的迭代器（常用于for循环中），因此当你用它迭代时，会尽快得到每个可用的结果。此外，当轮询到某个事件时（如`taskIO_1()`），直到遇到该任务中的`yield from`中断，开始处理下一个事件（如`taskIO_2()`），当`yield from`后面的子生成器完成任务时，该事件才再次被唤醒。

5. 因为`done`里面有需要的返回结果，但它目前还是个任务列表，所以要取出返回的结果值，要遍历它并逐个调用`result()`取出结果即可。（注：对于`asyncio.wait()`和`asyncio.as_completed()`返回的结果均是先完成的任务结果排在前面，所以此时打印出的结果不一定和原始顺序相同，但使用`gather()`的话可以得到原始顺序的结果集。）

6. 最后通过`loop.close()`关闭事件循环。

在Python 3.5开始引入了新的语法`async`和`await`，以简化并更好地标识异步IO。

要使用新的语法，只需要做两步简单的替换：

- 把`@asyncio.coroutine`替换为`async`；
- 把`yield from`替换为`await`。

更改上面的代码如下，可得到同样的结果：

```python
import time
import asyncio

async def taskIO_1():
    print('开始运行IO任务1...')
    await asyncio.sleep(2)  # 假设该任务耗时2s
    print('IO任务1已完成，耗时2s')
    return taskIO_1.__name__

async def taskIO_2():
    print('开始运行IO任务2...')
    await asyncio.sleep(3)  # 假设该任务耗时3s
    print('IO任务2已完成，耗时3s')
    return taskIO_2.__name__

async def main():  # 调用方
    tasks = [taskIO_1(), taskIO_2()]  # 把所有任务添加到task中
    done, pending = await asyncio.wait(tasks)  # 子生成器
    for r in done:  # done和pending都是一个任务，所以返回结果需要逐个调用result()
        print('协程无序返回值：' + r.result())

if __name__ == '__main__':
    start = time.time()
    loop = asyncio.get_event_loop()  # 创建一个事件循环对象loop
    try:
        loop.run_until_complete(main())  # 完成事件循环，直到最后一个任务结束
    finally:
        loop.close()  # 结束事件循环
    print('所有IO任务总耗时%.2f秒' % float(time.time()-start))
```
