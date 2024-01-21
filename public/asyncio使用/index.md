# Asyncio使用


<!--more-->

# 介绍

> **异步IO**：就是发起一个IO操作（如：网络请求，文件读写等），这些操作一般是比较耗时的，不用等待它结束，可以继续做其他事情，结束时会发来通知。 **协程**：又称为微线程，在一个线程中执行，执行函数时可以随时中断，由程序（用户）自身控制，执行效率极高，与多线程比较，没有切换线程的开销和多线程锁机制。

------

## asyncio中几个重要概念

### 1.事件循环

事件循环是每个 asyncio 应用的核心，管理所有的事件，在整个程序运行过程中不断循环执行并追踪事件发生的顺序将它们放在队列中，空闲时调用相应的事件处理者来处理这些事件。 

-  创建事件循环 `loop = asyncio.get_event_loop()` 获取当前事件循环。 如果当前 OS 线程没有设置当前事件循环并且 set_event_loop() 还没有被调用，asyncio 将创建一个新的事件循环并将其设置为当前循环。

- 另起一个线程创建事件循环

```
from threading import Thread
import asyncio

def start_thread_loop(loop):
    asyncio.set_event_loop(loop)
    loop.run_forever()

new_loop = asyncio.new_event_loop()
loop_thread = Thread(target=start_thread_loop, args=(new_loop,))
loop_thread.setDaemon(True) # 守护线程
loop_thread.start()
```

### 2.Future

Future对象表示尚未完成的计算，还未完成的结果，它和task上没有本质上的区别

### 3.Task

是Future的子类，作用是在运行某个任务的同时可以并发的运行多个任务。 asyncio.Task用于实现协作式多任务的库，且Task对象不能用户手动实例化，通过下面2个函数创建： `loop.create_task() 或 asyncio.ensure_future()` 

- loop.create_task() ,要在定义loop对象之后，调用将方法对象转化成了task的对象 
- asyncio.ensure_future() 直接调用asyncio 的ensure_future() 方法，返回的也是task 对象（我们还没有声明 loop 也可以提前定义好 task 对象）

### 4.async/await 关键字

asyncio实现了TCP、UDP、SSL等协议，async定义一个协程，await用于挂起阻塞的异步调用接口。 对于异步io你需要知道的重点，要注意的是，await语法只能出现在通过async修饰的函数中，否则会报SyntaxError错误。而且await后面的对象需要是一个*Awaitable*，或者实现了相关的协议。

> 注意 1. 所有需要异步执行的函数，都需要asyncio中的轮询器去轮询执行，如果函数阻塞，轮询器就会去执行下一个函数。所以所有需要异步执行的函数都需要加入到这个轮询器中。若在协程中需要有延时操作，应该使用 await asyncio.sleep()，而不是使用time.sleep()，因为使用time.sleep()后会释放GIL，阻塞整个主线程，从而阻塞整个事件循环。

# 创建一个协程

> 使用async可以定义协程对象，使用await可以针对耗时的操作进行挂起，就像生成器里的yield一样，函数让出控制权。协程遇到await，事件循环将会挂起该协程，执行别的协程，直到其他的协程也挂起或者执行完毕，再进行下一个协程的执行

耗时的操作一般是一些IO操作，例如网络请求，文件读取等。我们使用asyncio.sleep函数来模拟IO操作。协程的目的也是让这些IO操作异步化。

## 简单例子

```
import asyncio
from asyncio import tasks
from asyncore import loop


async def main():
    await asyncio.sleep(2)
    print('this is main running')


async def add_log():
    await asyncio.sleep(1)
    print('add log before running')


async def run_background():
    await asyncio.sleep(3)
    print('running task in background')


async def parameter_verification():
    await asyncio.sleep(1)
    print('parameter verification before running')


if __name__ == '__main__':
    # running async function method:

    # first method:
    tasks = asyncio.wait([main(), add_log(), run_background(), parameter_verification()])
    # the sequence of the tasks is not the same as the sequence of the function calling, and the same as define sequence
    result = asyncio.run(tasks)
    print(result)

    print(f'{"*" * 20}{"another method"}{"*" * 20}')

    # second method:
    tasks = asyncio.wait([main(), add_log(), run_background(), parameter_verification()])
    loop = asyncio.get_event_loop()
    result = loop.run_until_complete(tasks)
    print(result)
```

首先我们引入了 asyncio 这个包，这样我们才可以使用 async 和 await，然后我们使用 async 定义了一个 execute() 方法，方法接收一个数字参数，方法执行之后会打印这个数字。

随后我们直接调用了这个方法，然而这个方法并没有执行，而是返回了一个 coroutine 协程对象。随后我们使用 get_event_loop() 方法创建了一个事件循环 loop，并调用了 loop 对象的 run_until_complete() 方法将协程注册到事件循环 loop 中，然后启动。最后我们才看到了 execute() 方法打印了输出结果。

可见，async 定义的方法就会变成一个无法直接执行的 coroutine 对象，必须将其注册到事件循环中才可以执行。

## 进阶例子

> 多个任务，定义一个task列表，使用asyncio.gather(*tasks) 或 asyncio.wait(tasks) 接收

```
import asyncio
import time

now = lambda: time.time()

"""
asyncio.gather主要集中在收集结果上。它等待一堆task并按给定的顺序返回结果。

asyncio.wait等待task。而不是直接给你结果，它提供完成和待处理的任务。你必须手工收集结果。
asyncio.wait(tasks) 
ps:asyncio.wait([1,2,3]) 也可以使用 asyncio.gather(*tasks) 
ps: asyncio.gather(1,2,3),前者接受一个task列表，后者接收一堆task。
"""


# 定义一个异步任务
async def do_some_work(x):
    print("waiting:", x)
    # 模拟io阻塞
    await asyncio.sleep(x)
    return "Done after {}s".format(x)


async def main(loop):
    """
    :param loop: loop.create_task（需要传进loop参数）
    :return: None
    """
    coroutine1 = do_some_work(1)
    coroutine2 = do_some_work(2)
    coroutine3 = do_some_work(4)
    # asyncio.ensure_future
    tasks = [
        asyncio.ensure_future(coroutine1),
        asyncio.ensure_future(coroutine2),
        asyncio.ensure_future(coroutine3)
    ]
    # loop.create_task（需要传进loop参数）
    # tasks = [
    #     loop.create_task(coroutine1),
    #     loop.create_task(coroutine2),
    #     loop.create_task(coroutine3)
    # ]
    # 返回 完成的 task object
    dones, pendings = await asyncio.wait(tasks)
    print(dones, pendings)
    for task in dones:
        print("Task ret:", task.result())

    # 返回 task 方法的 返回值
    # results = await asyncio.gather(*tasks)
    # for result in results:
    #     print("Task ret:",result)


start = now()
loop = asyncio.get_event_loop()
loop.run_until_complete(main(loop))
print("Time:", now() - start)
```

# gather和wait 的区别

> 把多个协程注册进一个事件循环中的两种方法

## 使用方式区别

1. 使用`asyncio.wait()`

```
loop = asyncio.get_event_loop()
loop.run_until_complete(asyncio.wait(tasks))
```

1. 使用`asyncio.gather()`

```
loop = asyncio.get_event_loop()
loop.run_until_complete(asyncio.gather(*tasks)) # *接收args参数 
```

## 接收参数区别

### asyncio.wait

> 参数必须是list对象 ，list 对象存放多个 task object

- 用asyncio.ensure_future转为task对象

```
tasks=[
       asyncio.ensure_future(coroutine1),
       asyncio.ensure_future(coroutine2),
       asyncio.ensure_future(coroutine3)
]

loop = asyncio.get_event_loop()
loop.run_until_complete(asyncio.wait(tasks))
```

- 不转为task对象

```
loop = asyncio.get_event_loop()

tasks=[
       coroutine1,
       coroutine2,
       coroutine3
]

loop.run_until_complete(asyncio.wait(tasks))
```

### asyncio.gather

必须用 `*` 来接收 list 对象

```
tasks=[
       asyncio.ensure_future(coroutine1),
       asyncio.ensure_future(coroutine2),
       asyncio.ensure_future(coroutine3)
]

loop = asyncio.get_event_loop()
loop.run_until_complete(asyncio.gather(*tasks))
```

### 返回结果区别

#### asyncio.wait

`asyncio.wait`返回`dones`和`pendings` 

- dones：表示已经完成的任务 
- pendings：表示未完成的任务

我们需要手动去获取结果

```
dones, pendings = await asyncio.wait(tasks)
    print(dones, pendings)
    for task in dones:
        print("Task ret:", task.result())
```

#### asyncio.gather

它的返回值就是 return的结果 ，不用再task.result() 来获取

```
# 返回 task 方法的 返回值
    results = await asyncio.gather(*tasks)
    for result in results:
         print("Task ret:",result)
```

#### 另 asyncio.wait 带有控制功能

> 【控制运行任务数】：运行第一个任务就返回 FIRST_COMPLETED ：第一个任务完全返回 FIRST_EXCEPTION：产生第一个异常返回 ALL_COMPLETED：所有任务完成返回 （默认选项）

```
import asyncio
import random


async def coro(tag):
    print(">", tag)
    await asyncio.sleep(random.uniform(0.5, 5))
    print("<", tag)
    return tag


loop = asyncio.get_event_loop()

tasks = [coro(i) for i in range(1, 11)]

# 第一次wait 完成情况
print("Get first result:")
finished, unfinished = loop.run_until_complete(
    asyncio.wait(tasks, return_when=asyncio.FIRST_COMPLETED)) # 第一个任务完全返回

for task in finished:
    print(task.result())
print("unfinished:", len(unfinished))

# 继续第一次未完成任务
print("Get more results in 2 seconds:")
finished2, unfinished2 = loop.run_until_complete(
    asyncio.wait(unfinished, timeout=2)) # 超时2s 返回

for task in finished2:
    print(task.result())
print("unfinished2:", len(unfinished2))

# 继续第2次未完成任务
print("Get all other results:")
finished3, unfinished3 = loop.run_until_complete(asyncio.wait(unfinished2)) # ALL_COMPLETED：所有任务完成返回 （默认项）

for task in finished3:
    print(task.result())

loop.close()
```

# 动态添加协程

> 很多时候，我们的事件循环用于注册协程，而有的协程需要动态的添加到事件循环中。一个简单的方式就是使用多线程。当前线程创建一个事件循环，然后在新建一个线程，在新线程中启动事件循环。当前线程不会被block

相关函数介绍：

- loop.call_soon_threadsafe() ：与 call_soon()类似，等待此函数返回后马上调用回调函数，返回值是一个 asyncio.Handle 对象，此对象内只有一个方法为 cancel()方法，用来取消回调函数。

- loop.call_soon() ： 与call_soon_threadsafe()类似，call_soon_threadsafe() 是线程安全的

- loop.call_later()：延迟多少秒后执行回调函数

- loop.call_at()：在指定时间执行回调函数，这里的时间统一使用 loop.time() 来替代 time.sleep()

- asyncio.run_coroutine_threadsafe()： 动态的加入协程，参数为一个回调函数和一个loop对象，返回值为future对象，通过future.result()获取回调函数返回值

动态添加协程同步方式 通过调用 call_soon_threadsafe()函数，传入一个回调函数callback和一个位置参数

注意：同步方式，回调函数 more_work()为普通函数

```
import asyncio
from threading import Thread
import time

now = lambda: time.time()


def start_loop(loop):
    asyncio.set_event_loop(loop)
    loop.run_forever()


def more_work(x):
    print('More work {}'.format(x))
    time.sleep(x)
    print('Finished more work {}'.format(x))


start = now()
new_loop = asyncio.new_event_loop()
t = Thread(target=start_loop, args=(new_loop,))
t.start()
print('TIME: {}'.format(time.time() - start))

new_loop.call_soon_threadsafe(more_work, 6)
new_loop.call_soon_threadsafe(more_work, 3)
print('here')
```

启动上述代码之后，当前线程不会被block，新线程中会按照顺序执行call_soon_threadsafe方法注册的more_work方法， 后者因为time.sleep操作是同步阻塞的，因此运行完毕more_work需要大致6 + 3

------

异步方式

```
import asyncio
import time
from threading import Thread

now = lambda: time.time()


def start_loop(loop):
    asyncio.set_event_loop(loop)
    loop.run_forever()


async def do_some_work(x):
    print('Waiting {}'.format(x))
    await asyncio.sleep(x)
    print('Done after {}s'.format(x))


start = now()
new_loop = asyncio.new_event_loop()
t = Thread(target=start_loop, args=(new_loop,))
t.start()
print('TIME: {}'.format(time.time() - start))

asyncio.run_coroutine_threadsafe(do_some_work(6), new_loop)
asyncio.run_coroutine_threadsafe(do_some_work(4), new_loop)
```

上述的例子，主线程中创建一个new_loop，然后在另外的子线程中开启一个无限事件循环。 主线程通过run_coroutine_threadsafe新注册协程对象。这样就能在子线程中进行事件循环的并发操作，同时主线程又不会被block。一共执行的时间大概在6s左右。

# 协程的停止

future对象有几个状态：

Pending Running Done Cacelled 创建future的时候，task为pending，事件循环调用执行的时候当然就是running，调用完毕自然就是done，如果需要停止事件循环，就需要先把task取消。可以使用asyncio.Task获取事件循环的task

```
import asyncio
import time

now = lambda: time.time()

async def do_some_work(x):
    print("Waiting:", x)
    await asyncio.sleep(x)
    return "Done after {}s".format(x)


coroutine1 = do_some_work(1)
coroutine2 = do_some_work(2)
coroutine3 = do_some_work(2)

tasks = [
    asyncio.ensure_future(coroutine1),
    asyncio.ensure_future(coroutine2),
    asyncio.ensure_future(coroutine3),
]

start = now()

loop = asyncio.get_event_loop()
try:
    loop.run_until_complete(asyncio.wait(tasks))
except KeyboardInterrupt as e:
    print(asyncio.Task.all_tasks())
    for task in asyncio.Task.all_tasks():
        print(task.cancel())
    loop.stop()
    loop.run_forever()
finally:
    loop.close()

print("Time:", now() - start)
```

启动事件循环之后，马上ctrl+c，会触发run_until_complete的执行异常 KeyBorardInterrupt。然后通过循环asyncio.Task取消future。 True表示cannel成功，loop stop之后还需要再次开启事件循环，最后在close，不然还会抛出异常 循环task，逐个cancel是一种方案，可是正如上面我们把task的列表封装在main函数中，main函数外进行事件循环的调用。这个时候，main相当于最外出的一个task，那么处理包装的main函数即可。

# 协程中生产-消费模型设计

> 通过上面的动态添加协程的思想，我们可以设计一个生产-消费的模型，至于中间件（管道）是什么无所谓，下面以内置队列和redis队列来举例说明。

提示：若想主线程退出时，子线程也随之退出，需要将子线程设置为守护线程，函数 setDaemon(True)

```
import asyncio
from threading import Thread
from collections import deque
import random
import time

def start_thread_loop(loop):
    asyncio.set_event_loop(loop)
    loop.run_forever()

def consumer():
    while True:
        if dq:
            msg = dq.pop()
            if msg:
                asyncio.run_coroutine_threadsafe(thread_example('Zarten'+ msg), new_loop)


async def thread_example(name):
    print('正在执行name:', name)
    await asyncio.sleep(2)
    return '返回结果：' + name



dq = deque()

new_loop = asyncio.new_event_loop()
loop_thread = Thread(target= start_thread_loop, args=(new_loop,))
loop_thread.setDaemon(True)
loop_thread.start()

consumer_thread = Thread(target= consumer)
consumer_thread.setDaemon(True)
consumer_thread.start()

while True:
    i = random.randint(1, 10)
    dq.appendleft(str(i))
    time.sleep(2)
```

# redis队列模型

生产者代码：

```
import redis

conn_pool = redis.ConnectionPool(host='127.0.0.1')
redis_conn = redis.Redis(connection_pool=conn_pool)

redis_conn.lpush('coro_test', '1')
redis_conn.lpush('coro_test', '2')
redis_conn.lpush('coro_test', '3')
redis_conn.lpush('coro_test', '4')
```

消费者代码：

```
import asyncio
from threading import Thread
import redis

def get_redis():
    conn_pool = redis.ConnectionPool(host= '127.0.0.1')
    return redis.Redis(connection_pool= conn_pool)

def start_thread_loop(loop):
    asyncio.set_event_loop(loop)
    loop.run_forever()

async def thread_example(name):
    print('正在执行name:', name)
    await asyncio.sleep(2)
    return '返回结果：' + name


redis_conn = get_redis()

new_loop = asyncio.new_event_loop()
loop_thread = Thread(target= start_thread_loop, args=(new_loop,))
loop_thread.setDaemon(True)
loop_thread.start()

#循环接收redis消息并动态加入协程
while True:
    msg = redis_conn.rpop('coro_test')
    if msg:
        asyncio.run_coroutine_threadsafe(thread_example('Zarten' + bytes.decode(msg, 'utf-8')), new_loop)
```

# asyncio在aiohttp中的应用

> aiohttp是一个异步库，分为客户端和服务端，下面只是简单对客户端做个介绍以及一个经常遇到的异常情况。aiohttp客户端为异步网络请求库

```
import asyncio
import aiohttp

count = 0

async def get_http(url):
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as res:
            global count
            count += 1
            print(count, res.status)

def main():
    loop = asyncio.get_event_loop()
    url = 'https://www.baidu.com/s?ie=utf-8&f=8&rsv_bp=1&ch=&tn=baiduerr&bar=&wd={0}'
    tasks = [get_http(url.format(i)) for i in range(10)]
    loop.run_until_complete(asyncio.wait(tasks))
    loop.close()
if __name__ == '__main__':
    main()
```

aiohttp并发量太大的异常解决方案 在使用aiohttp客户端进行大量并发请求时，程序会抛出 ValueError: too many file descriptors in select() 的错误。

异常代码示例

说明：测试机器为windows系统

```
import asyncio
import aiohttp

count = 0

async def get_http(url):
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as res:
            global count
            count += 1
            print(count, res.status)

def main():
    loop = asyncio.get_event_loop()
    url = 'https://www.baidu.com/s?ie=utf-8&f=8&rsv_bp=1&ch=&tn=baiduerr&bar=&wd={0}'
    tasks = [get_http(url.format(i)) for i in range(600)]
    loop.run_until_complete(asyncio.wait(tasks))
    loop.close()
if __name__ == '__main__':
    main()
```

原因分析：使用aiohttp时，python内部会使用select()，操作系统对文件描述符最大数量有限制，linux为1024个，windows为509个。

解决方案：

最常见的解决方案是：限制并发数量（一般500），若并发的量不大可不作限制。其他方案这里不做介绍，如windows下使用loop = asyncio.ProactorEventLoop() 以及使用回调方式等

限制并发数量方法 提示：此方法也可用来作为异步爬虫的限速方法（反反爬）

使用semaphore = asyncio.Semaphore(500) 以及在协程中使用 async with semaphore: 操作

具体代码如下：

```
import asyncio
import aiohttp


async def get_http(url):
    async with semaphore:
        async with aiohttp.ClientSession() as session:
            async with session.get(url) as res:
                global count
                count += 1
                print(count, res.status)

if __name__ == '__main__':
    count = 0

    semaphore = asyncio.Semaphore(500)
    loop = asyncio.get_event_loop()
    url = 'https://www.baidu.com/s?ie=utf-8&f=8&rsv_bp=1&ch=&tn=baiduerr&bar=&wd={0}'
    tasks = [get_http(url.format(i)) for i in range(600)]
    loop.run_until_complete(asyncio.wait(tasks))
    loop.close()
```

# 在线程或进程池中执行代码

在《流畅的python》中有这样一段话。

> 函数(例如io读写，requests网络请求)阻塞了客户代码与asycio事件循环的唯一线程，因此在执行调用时，整个应用程序都会冻结。这个问题的解决方法是，使用事件循环对象的 run_in_executor方法。asyncio的事件循环在背后维护着一个ThreadPoolExecutor对象，我们可以调用run_in_executor方法，把可调用对象发给它执行。

```
import asyncio
from time import sleep, strftime
from concurrent import futures

executor = futures.ThreadPoolExecutor(max_workers=5)


async def blocked_sleep(name, t):
    print(strftime('[%H:%M:%S]'), end=' ')
    print('sleep {} is running {}s'.format(name, t))
    loop = asyncio.get_event_loop()
    await loop.run_in_executor(executor, sleep, t)
    print(strftime('[%H:%M:%S]'), end=' ')
    print('sleep {} is end'.format(name))
    return t


async def main():
    future = (blocked_sleep(i, i) for i in range(1, 6))
    fs = asyncio.gather(*future)
    return await fs


loop = asyncio.get_event_loop()
results = loop.run_until_complete(main())
print('results: {}'.format(results))
```

在同一个线程里，两个 event loop 无法同时 run，但这不能阻止您用两个线程分别跑两个 event loop, 其次再说 ThreadPoolExecutor。您也可以看到，它根本不是 asyncio 库的东西。当您创建一个 ThreadPoolExecutor 对象时，您实际上是创建了一个线程池。仅此而已，与 asyncio、event loop 并无瓜葛。而当您明确使用一个 event loop 的 run_in_executor() 方法时，其实底层做的只有两件事：

1,用线程池执行给定函数，与 asyncio 毫无关系； 2,给线程池执行结果增加一个回调，该回调会在 event loop 的下一次循环中保存执行结果。 所以 run_in_executor() 只是将传统的线程池结果拉回到给定 event loop 中，以便进一步处理而已，不存在谁共享谁的关系，指定谁是谁。您可以尝试一下，在多个线程中跑多个 event loop，然后都向同一个线程池扔任务，然后返回结果：

```
import asyncio
import threading
import time
from concurrent.futures.thread import ThreadPoolExecutor

e = ThreadPoolExecutor()


def worker(index):
    print(index, 'before:', time.strftime('%X'))
    time.sleep(1)
    print(index, 'after:', time.strftime('%X'))
    return index


def main(index):
    loop = asyncio.new_event_loop()
    res = loop.run_until_complete(loop.run_in_executor(e, worker, index))
    print('Thread', index, 'got result', res)


threads = []
for i in range(5):
    t = threading.Thread(target=main, args=(i,))
    t.start()
    threads.append(t)

for t in threads:
    t.join()
```

不同于上面的方法，这里是把阻塞的方法放到新的线程里跑。

# 参考引用

- [Python中异步协程的使用方法介绍：崔庆才](https://cuiqingcai.com/6160.html) [Python中协程异步IO（asyncio）详解](https://zhuanlan.zhihu.com/p/59621713) [Python中asyncio与aiohttp入门教程](https://www.jb51.net/article/148926.htm) [python中重要的模块--asyncio](https://www.cnblogs.com/zhaof/p/8490045.html) [MING's BLOG](http://python-online.cn/zh_CN/latest/c02/c02_10.html) [官方文档中文](https://docs.python.org/zh-cn/3/library/asyncio-eventloop.html)

- [python协程库asyncio(异步io) - 少年白 (xhongc.github.io)](https://xhongc.github.io/asyncio_course.html)


---

> 作者: map[avatar:<nil> email:<nil> link:<nil> name:<nil>]  
> URL: https://blog-12x.pages.dev/asyncio%E4%BD%BF%E7%94%A8/  

