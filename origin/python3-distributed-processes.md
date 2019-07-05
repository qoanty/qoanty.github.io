---
title: "Python3 学习笔记（分布式进程）"
date: 2019-04-20T20:07:34+08:00
tags: ["python", "processes"]
categories: ["Learn", "Python"]
draft: false
---

## 进程与多进程

对于操作系统来说，一个任务就是一个进程（Process），比如打开一个浏览器就是启动一个浏览器进程，打开一个记事本就启动了一个记事本进程，打开两个记事本就启动了两个记事本进程，打开一个Word就启动了一个Word进程。

Python中的`multiprocessing`模块提供了一个`Process`类来代表一个进程对象，例如启动一个子进程并等待其结束：

```python
from multiprocessing import Process
import os

# 子进程要执行的代码
def run_proc(name):
    print('Run child process %s (%s) and parent process is (%s).' % (name, os.getpid(), os.getppid()))

if __name__=='__main__':
    print('Parent process %s.' % os.getpid())
    p = Process(target=run_proc, args=('test',))
    print('Child process will start.')
    p.start()
    p.join()
    print('Child process end.')
```

执行结果为：

```python
Parent process 8488.
Child process will start.
Run child process test (1684) and parent process is (8488).
Child process end.
```

创建子进程时，只需要传入一个执行函数和函数的参数，创建一个`Process`实例，用`start()`方法启动，`getpid()`获取子进程的ID，`getppid()`获取父进程的ID，`join()`方法可以等待子进程结束后再继续往下运行，通常用于进程间的同步。

如果要启动大量的子进程，可以用进程池的方式批量创建子进程：

```python
from multiprocessing import Pool
import os, time, random

def long_time_task(name):
    print('Run task %s (%s)...' % (name, os.getpid()))
    start = time.time()
    time.sleep(random.random() * 3)
    end = time.time()
    print('Task %s runs %0.2f seconds.' % (name, (end - start)))

if __name__=='__main__':
    print('Parent process %s.' % os.getpid())
    p = Pool(4)
    for i in range(5):
        p.apply_async(long_time_task, args=(i,))
    print('Waiting for all subprocesses done...')
    p.close()
    p.join()
    print('All subprocesses done.')
```

执行结果为：

```python
Parent process 3932.
Waiting for all subprocesses done...
Run task 0 (1748)...
Run task 1 (9512)...
Run task 2 (9196)...
Run task 3 (1484)...
Task 2 runs 0.85 seconds.
Run task 4 (9196)...
Task 0 runs 1.29 seconds.
Task 3 runs 1.33 seconds.
Task 1 runs 2.30 seconds.
Task 4 runs 2.57 seconds.
All subprocesses done.
```

对`Pool`对象调用`join()`方法会等待所有子进程执行完毕，调用`join()`之前必须先调用`close()`，调用`close()`之后就不能继续添加新的`Process`了。

请注意输出的结果，task 0，1，2，3是立刻执行的，而task 4要等待前面某个task完成后才执行，这是因为`Pool`被设置为4，因此最多同时执行4个进程，并不是操作系统的限制。

Python的`multiprocessing`模块提供了`Queue`、`Pipes`等多种方式来交换数据，以`Queue`为例，在父进程中创建两个子进程，一个往`Queue`里写数据，一个从`Queue`里读数据：

```python
from multiprocessing import Process, Queue
import os, time, random

# 写数据进程执行的代码:
def write(q):
    print('Process to write: %s' % os.getpid())
    for value in ['A', 'B', 'C']:
        print('Put %s to queue...' % value)
        q.put(value)
        time.sleep(random.random())

# 读数据进程执行的代码:
def read(q):
    print('Process to read: %s' % os.getpid())
    while True:
        value = q.get(True)
        print('Get %s from queue.' % value)

if __name__=='__main__':
    # 父进程创建Queue，并传给各个子进程
    q = Queue()
    pw = Process(target=write, args=(q,))
    pr = Process(target=read, args=(q,))
    # 启动子进程pw，写入
    pw.start()
    # 启动子进程pr，读取
    pr.start()
    # 等待pw结束
    pw.join()
    # pr进程里是死循环，无法等待其结束，只能强行终止
    pr.terminate()
```

执行结果为：

```python
Process to write: 1384
Put A to queue...
Process to read: 7308
Get A from queue.
Put B to queue...
Get B from queue.
Put C to queue...
Get C from queue.
```

## 分布式进程

Python的`multiprocessing`模块不但支持多进程，其中`managers`子模块还支持把多进程分布到多台机器上。一个服务进程可以作为调度者，将任务分布到其他多个进程中，依靠网络通信。

先看服务进程，服务进程负责启动`Queue`，把`Queue`注册到网络上，然后往`Queue`里面写入任务：

```python
import random, time, queue
from multiprocessing.managers import BaseManager
from multiprocessing import freeze_support

# 任务个数
task_number = 10
# 发送任务的队列
task_queue = queue.Queue(task_number)
# 接收任务的队列
result_queue = queue.Queue(task_number)

def gettask():
    return task_queue

def getresult():
    return result_queue

def test():
    # windows下绑定调用接口不能使用lambda，所以只能先定义函数再绑定
    BaseManager.register('get_task', callable=gettask)
    BaseManager.register('get_result', callable=getresult)
    # 绑定端口并设置验证码，windows下需要填写ip地址，linux下不填默认为本地
    manager = BaseManager(address=('127.0.0.1', 5002), authkey=b'abc')
    # 启动Queue
    manager.start()
    try:
        # 通过网络获取任务队列和结果队列
        task = manager.get_task()
        result = manager.get_result()
        # 添加任务
        for i in range(task_number):
            n = random.randint(0, 10000)
            print('Put task %d...' % n)
            task.put(n)
        # 每秒检测一次是否所有任务都被执行完
        print('Try get results...')
        while not result.full():
            time.sleep(1)
        for i in range(result.qsize()):
            ans = result.get()
            print('Result: %s, runtime: %ds' % ans)
    except Exception:
        print('Manager error')
    finally:
        # 一定要关闭，否则会爆管道未关闭的错误
        manager.shutdown()

if __name__ == '__main__':
    # windows下多进程可能会炸，添加这句可以缓解
    freeze_support()
    test()
```

再看任务进程：

```python
import time, sys, random
from multiprocessing.managers import BaseManager

# 从网络上获取Queue，注册时只提供名字
BaseManager.register('get_task')
BaseManager.register('get_result')
# 连接到服务器，注意端口和验证码要保持一致
conn = BaseManager(address=('127.0.0.1', 5002), authkey=b'abc')

try:
    conn.connect()
except Exception:
    print('连接失败')
    sys.exit();

# 获取Queue的对象
task = conn.get_task()
result = conn.get_result()

# 从task队列取任务，并把结果写入result队列
while not task.empty():
    n = task.get(timeout=1)
    print('run task %d * %d...' % (n, n))
    r = '%d * %d = %d' % (n, n, n*n)
    sleeptime = random.randint(0, 3)
    time.sleep(sleeptime)
    rt = (r, sleeptime)
    result.put(rt)

if __name__ == '__main__':
    pass
```

执行结果为：

```python
Put task 8399...
Put task 4368...
Put task 8098...
Put task 6887...
Put task 3504...
Put task 7503...
Put task 8492...
Put task 5150...
Put task 7444...
Put task 8359...
Try get results...
Result: 8399 * 8399 = 70543201, runtime: 1s
Result: 4368 * 4368 = 19079424, runtime: 0s
Result: 8098 * 8098 = 65577604, runtime: 0s
Result: 6887 * 6887 = 47430769, runtime: 1s
Result: 3504 * 3504 = 12278016, runtime: 1s
Result: 7503 * 7503 = 56295009, runtime: 2s
Result: 8492 * 8492 = 72114064, runtime: 2s
Result: 5150 * 5150 = 26522500, runtime: 3s
Result: 7444 * 7444 = 55413136, runtime: 2s
Result: 8359 * 8359 = 69872881, runtime: 1s
```

先启动服务进程，发送完任务后，开始等待result队列的结果，再启动任务进程，任务进程结束后，服务进程会继续打印出结果。

这就是一个简单但真正的分布式计算，把代码稍加改造，启动多个worker，就可以把任务分布到几台甚至几十台机器上，比如把计算`n*n`的代码换成发送邮件，就实现了邮件队列的异步发送。

