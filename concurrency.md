# Python中使用协程实现并发

## 操作系统的I/O模型介绍

### 阻塞I/O

特点：在执行I/O操作时，应用程序会被阻塞

```
socket.recv(1024)
```

### 非阻塞I/O

I/O操作如果不能完成，就返回 EAGAIN/EWOULDBLOCK，应用程序发起多次重试（系统调用）

在python中使用异常捕获：[`BlockingIOError`](https://docs.python.org/3/library/exceptions.html#BlockingIOError)

```python
class socket:
    def send(self, data):
        while True:
            try:
                return self.send(data)
            except BlockingIOError:
                wait
```

### I/O Multipleing

另外一个阻塞解决方案是带有阻塞通知的非阻塞 I/O。在这种模型中，配置的是非阻塞 I/O，然后使用阻塞 select 系统调用来确定一个 I/O 描述符何时有操作。使 select 调用非常有趣的是它可以用来为多个描述符提供通知，而不仅仅为一个描述符提供通知。对于每个提示符来说，我们可以请求这个描述符可以写数据、有读数据可用以及是否发生错误的通知。

使用I/O Multiplexing实现：

* ```select.select(rlist, wlist, xlist[, timeout])```
* ```poll.poll([timeout])```

```
stdin = sys.stdin
select.select([stdin.fileno()], [], [], 5); stdin.read()
```

### 异步I/O

最后，异步非阻塞 I/O 模型是一种处理与 I/O 重叠进行的模型。读请求会立即返回，说明 read 请求已经成功发起了。在后台完成读操作时，应用程序然后会执行其他处理操作。当 read 的响应到达时，就会产生一个信号或执行一个基于线程的回调函数来完成这次 I/O 处理过程。


### 理解同步、异步、阻塞、非阻塞之间的区别

要区分阻塞，非阻塞，同步和异步，要说明指定的主体。
引用 [知乎上面的回答](https://www.zhihu.com/question/19732473)

同步和异步关注的是消息通信机制（描述的主体是操作或者调用）：

* 所谓同步，就是在发出一个*调用*时，在没有得到结果之前，该*调用*就不返回。但是一旦调用返回，就得到返回值了。
* 异步则是相反，*调用*在发出之后，这个调用就直接返回了，所以没有返回结果。

```data = get_data()```

```
task = get_data()
wait
data = task.get_result()

get_data(callback=handle_data)
```

阻塞和非阻塞关注的是程序在等待调用结果（消息，返回值）时的状态（描述的主体是程序)，我们在说一个操作是阻塞还是非阻塞的时候，隐含的意思其实是对于程序（进程、线程）是否是阻塞的：

* 阻塞调用是指调用结果返回之前，当前线程会被挂起。调用线程只有在得到结果之后才会返回。
* 非阻塞调用指在不能立刻得到结果之前，该调用不会阻塞当前线程。

## 同步非阻塞编程模型

对于开发人员来说，需要一个好的同步非阻塞编程模型，当然，这个模型可以构建在操作系统的任意I/O操作之上。

### Asyncio

[PEP3156](https://www.python.org/dev/peps/pep-3156/)

provides infrastructure for writing single-threaded concurrent code using coroutines, multiplexing I/O access over sockets and other resources

一些概念：

* concurrency: 同时存在，同时发生 [wikipedia](https://en.wikipedia.org/wiki/Concurrenc…)
* parallelism: 完成并发的一种方式，通常指多个CPU同时执行
* coroutine: A coroutine is a generalization of a subroutine which can be paused at some point and returns a value. When the coroutine is called again, it is resumed right from the point it was paused and its environment remains intact.
* multiplexing I/O: 多个模拟或者数字信号数据流被合并到同一个共享媒介中，目标是共享昂贵的资源

### 演示
