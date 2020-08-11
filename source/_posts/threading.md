---
title: Python中的多线程库threading
date: 2020-07-17 11:46:53
author: Sun YuHong
tags: [python,多线程]
---

# Python中的多线程库threading



## 1.概要

### 1.1 常见方法与属性

| 方法与属性         | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| current_thread()   | 返回当前线程Object，如 <Thread(Thread-1, started 21148)>2    |
| active_count()     | 返回当前活跃的线程数，1个主线程+n个子线程                    |
| get_ident()        | 返回当前线程id                                               |
| enumerater()       | 返回当前活动 Thread 对象列表                                 |
| main_thread()      | 返回主 Thread 对象                                           |
| settrace(func)     | 为所有线程设置一个 trace 函数                                |
| setprofile(func)   | 为所有线程设置一个 profile 函数                              |
| stack_size([size]) | 返回新创建线程栈大小；或为后续创建的线程设定栈大小为 size    |
| TIMEOUT_MAX        | Lock.acquire(), RLock.acquire(), Condition.wait() 允许的最大超时时间 |

### 1.2 包含的类

| 类名      | 描述                                                         |
| --------- | ------------------------------------------------------------ |
| Thread    | 基本线程类                                                   |
| Lock      | 互斥锁                                                       |
| RLock     | 可重用锁，使单一线程再次获得已持有的锁（递归锁）             |
| Condition | 条件锁，使得一个线程等待另一个线程满足特定条件，比如改变状态或某个值 |
| Semaphore | 信号量，为线程间共享的有限资源提供一个”计数器”，如果没有可用资源则会被阻塞 |
| Event     | 事件锁，任意数量的线程等待某个事件的发生，在该事件发生后所有线程被激活 |
| Timer     | 计时器                                                       |
| Barrier   | 障碍，必须达到指定数量的线程后才可以继续执行，否则阻塞       |



## 2.创建线程

### 2.1 第一种创建方法

​	最常见的方法就是实例化threading.Thread，将要执行的任务函数作为参数传入线程。

​	函数定义如下：

```python
threading.Thread(self, group=None, target=None, name=None,
     args=(), kwargs=None, *, daemon=None)
```

- 参数group是预留的，用于将来扩展；
- 参数target是一个可调用对象，在线程启动后执行；
- 参数name是线程的名字。默认值为“Thread-N“，N是一个数字。
- 参数args和kwargs分别表示调用target时的参数列表和关键字参数。

```python
import threading
import time

def show(arg):
    time.sleep(1)
    print('thread '+str(arg)+" running....")

if __name__ == '__main__':
    for i in range(10):
        t = threading.Thread(target=show, args=(i,))
        t.start()
```

​	其中args为参数表，必须是一个元组类型；如果参数只有一个，要写成args=(x1,)，不能直接写args=(x1)。



### 2.2 第二种创建方法

​	继承Thread类，并重写它的run()方法

```python
import threading

class MyThread(threading.Thread):
    def __init__(self, thread_name):
        # 注意：一定要显式的调用父类的初始化函数。
        super(MyThread, self).__init__(name=thread_name)

    def run(self):
        print("%s正在运行中......" % self.name)

if __name__ == '__main__':    
    for i in range(10):
        MyThread("thread-" + str(i)).start()
```



## 3.关于Thread类的说明

​	Thread类包含如下常见方法：

| 方法与属性                 | 说明                                                         |
| -------------------------- | ------------------------------------------------------------ |
| start()                    | 启动线程，等待CPU调度                                        |
| run()                      | 线程被cpu调度后自动执行的方法                                |
| getName()、setName()、name | 分别用于获取和设置线程的名称。                               |
| setDaemon()                | 设置为后台线程或前台线程（默认是False，前台线程）。如果是后台线程，主线程执行过程中，后台线程也在进行，主线程执行完毕后，后台线程不论成功与否，均停止。如果是前台线程，主线程执行过程中，前台线程也在进行，主线程执行完毕后，等待前台线程执行完成后，程序才停止。 |
| ident                      | 获取线程的标识符。线程标识符是一个非零整数，只有在调用了start()方法之后该属性才有效，否则它只返回None。 |
| is_alive()                 | 判断线程是否是激活的（alive）。从调用start()方法启动线程，到run()方法执行完毕或遇到未处理异常而中断这段时间内，线程是激活的。 |
| isDaemon()方法和daemon属性 | 是否为守护线程                                               |
| join([timeout])            | 调用该方法将会使主调线程堵塞，直到被调用线程运行结束或超时。参数timeout是一个数值类型，表示超时时间，如果未提供该参数，那么主调线程将一直堵塞到被调线程结束。 |



### 3.1 关于join方法的一些说明

join可以用于在主线程中阻塞线程，相当于让主线程等待子线程执行结束。

比如：

```python
import time
import threading

def doWaiting():
    print('start waiting:', time.strftime('%H:%M:%S'))
    time.sleep(3)
    print('stop waiting', time.strftime('%H:%M:%S'))

t = threading.Thread(target=doWaiting)
t.start()
# 确保线程t已经启动
time.sleep(1)
print('start join')
# 将一直堵塞，直到t运行结束。
t.join()
print('end join')
```

如果没有t.join()，那么主线程将不会等待子线程的运行完成，输出结果为：

```
start waiting: 12:08:22
start join
end join
stop waiting 12:08:25
```

如果有t.join()，那么主线程将会等待子线程doWaiting执行完毕后，再继续执行后续的结果，输出结果为：

```
start waiting: 12:09:21
start join
stop waiting 12:09:24
end join
```



### **3.2 关于setDaemon方法的一些说明**

语法：

```python
t=threading.Thread(target=func,args=())
t.setDaemon(True)#设置线程t为守护线程
t.start()
```

Q：什么是守护线程？

- 如果当前python线程是守护线程，那么意味着这个线程是“不重要”的，“不重要”意味着如果他的主进程结束了但该守护线程没有运行完，守护进程就会被强制结束。如果线程是非守护线程，那么父进程只有等到守护线程运行完毕后才能结束。

- 守护线程必须在start方法前设置（不能将正在运行的线程设置为守护线程）

  

```python
import threading
import time

def run():
    print(threading.current_thread().getName(), "开始工作")
    time.sleep(2)       # 子线程停2s
    print("子线程工作完毕")

for i in range(3):
    t = threading.Thread(target=run,)
    t.setDaemon(True)   # 把子线程设置为守护线程，必须在start()之前设置
    t.start()

time.sleep(1)     # 主线程停1秒
print("主线程结束了！")
print(threading.active_count())  # 输出活跃的线程数
```

```python
#输出结果
Thread-1 开始工作
Thread-2 开始工作
Thread-3 开始工作
主线程结束了！
4
```

​	由上例子可以看出，当主线程结束时，虽然之前创建的3个线程仍然在运行，但主线程结束了，这些守护线程也随之结束。



## 4.线程锁

​	引入线程锁的动机：限制多个线程对于数据的并发访问。

​	threading库中提供了如下6种线程锁机制：

- Lock 互斥锁
- RLock 可重入锁
- Semaphore 信号
- Event 事件
- Condition 条件
- Barrier “阻碍”

### 4.1 互斥锁（Lock）

​	有很多教程是这样写的：利用互斥锁去锁住共享数据，从而保证线程对数据访问的同步。

​	其实这种描述并不确切，准确地来说，**互斥锁锁住的是“对共享数据进行修改的代码”，而并不是数据本身。**

​	我们来看下面这个例子：

```python
import threading
import time

number = 0
lock = threading.Lock()

def plus(lk):
    global number       # global声明此处的number是外面的全局变量number
    lk.acquire()        # 开始加锁
    for _ in range(1000000):    # 进行一个大数级别的循环加一运算
        number += 1
    print("子线程%s运算结束后，number = %s" % (threading.current_thread().getName(), number))
    lk.release()        # 释放锁，让别的线程也可以访问number

if __name__ == '__main__':
    for i in range(2):      # 用2个子线程，就可以观察到脏数据
        t = threading.Thread(target=plus, args=(lock,)) # 需要把锁当做参数传递给plus函数
        t.start()
    time.sleep(2)       # 等待2秒，确保2个子线程都已经结束运算。
    print("主线程执行完毕后，number = ", number)
```

​	lk.acquire()和lk.release()是对锁的增加/释放操作，其中间的部分是对共享数据（global number）的修改操作。



### 4.2 可重用锁（RLock）

​	Rlock是互斥锁的拓展，它在锁内部的数据结构中维护了一个整形变量count，表示上了几次锁，从而支持重复上锁而不阻塞。



### 4.3 信号量（Semaphore）

​	在OS中经常出现的一个东西，它允许指定有限的线程去并发访问同一资源，不再详述。

```python
import time
import threading

def run(n, se):
    se.acquire()
    print("run the thread: %s" % n)
    time.sleep(1)
    se.release()

# 设置允许5个线程同时运行
semaphore = threading.BoundedSemaphore(5)
for i in range(20):
    t = threading.Thread(target=run, args=(i,semaphore))
    t.start()
```



### 4.4 事件锁（Event）

​	事件线程锁的运行机制：全局定义了一个Flag，如果Flag的值为False，那么当程序执行wait()方法时就会阻塞，如果Flag值为True，线程不再阻塞。

​	这种锁，类似交通红绿灯（默认是红灯），它属于在红灯的时候一次性阻挡所有线程，在绿灯的时候，**一次性放行所有**排队中的线程。

​	事件主要提供了四个方法：set()、wait()、clear()和is_set()。

1. clear()：将事件的Flag设置为False。
2. set()：会将Flag设置为True。
3. wait()：将等待“红绿灯”信号。
4. is_set()：判断当前是否"绿灯放行"状态



​	下面是一个模拟红绿灯，然后汽车通行的例子：

```python
#利用Event类模拟红绿灯
import threading
import time

event = threading.Event()

def lighter():
    green_time = 5       # 绿灯时间
    red_time = 5         # 红灯时间
    event.set()          # 初始设为绿灯
    while True:
        print("\33[32;0m 绿灯亮...\033[0m")
        time.sleep(green_time)
        event.clear()
        print("\33[31;0m 红灯亮...\033[0m")
        time.sleep(red_time)
        event.set()

def run(name):
    while True:
        if event.is_set():      # 判断当前是否"放行"状态
            print("一辆[%s] 呼啸开过..." % name)
            time.sleep(1)
        else:
            print("一辆[%s]开来，看到红灯，无奈的停下了..." % name)
            event.wait()
            print("[%s] 看到绿灯亮了，瞬间飞起....." % name)

if __name__ == '__main__':

    light = threading.Thread(target=lighter,)
    light.start()

    for name in ['奔驰', '宝马', '奥迪']:
        car = threading.Thread(target=run, args=(name,))
        car.start()
```



### 4.5 条件变量（Condition）

​	Condition称作条件锁，依然是通过acquire()/release()加锁解锁。

​	wait([timeout])方法将使线程进入Condition的等待池等待通知，并释放锁。使用前线程必须已获得锁定，否则将抛出异常。

​	notify()方法将从等待池挑选一个线程并通知，收到通知的线程将自动调用acquire()尝试获得锁定（进入锁定池），其他线程仍然在等待池中。调用这个方法不会释放锁定。使用前线程必须已获得锁定，否则将抛出异常。

​	notifyAll()方法将通知等待池中所有的线程，这些线程都将进入锁定池尝试获得锁定。调用这个方法不会释放锁定。使用前线程必须已获得锁定，否则将抛出异常。

​	下面的例子，有助于你理解Condition的使用方法：

```python
import threading
import time

num = 0
con = threading.Condition()

class Foo(threading.Thread):

    def __init__(self, name, action):
        super(Foo, self).__init__()
        self.name = name
        self.action = action

    def run(self):
        global num
        con.acquire()
        print("%s开始执行..." % self.name)
        while True:
            if self.action == "add":
                num += 1
            elif self.action == 'reduce':
                num -= 1
            else:
                exit(1)
            print("num当前为：", num)
            time.sleep(1)
            if num == 5 or num == 0:
                print("暂停执行%s！" % self.name)
                con.notify()
                con.wait()
                print("%s开始执行..." % self.name)
        con.release()

if __name__ == '__main__':
    a = Foo("线程A", 'add')
    b = Foo("线程B", 'reduce')
    a.start()
    b.start()
```

```
线程A开始执行...
num当前为： 1
num当前为： 2
num当前为： 3
num当前为： 4
num当前为： 5
暂停执行线程A！
线程B开始执行...
num当前为： 4
num当前为： 3
num当前为： 2
num当前为： 1
num当前为： 0
暂停执行线程B！
线程A开始执行...
num当前为： 1
num当前为： 2
num当前为： 3
num当前为： 4
num当前为： 5
暂停执行线程A！
线程B开始执行...
```

### 4.6 通过with来对锁进行操作

​	我们都知道一个锁的lock()和release()方法应当是一一对应的。如果申请锁之后忘记释放锁，那很有可能导致死锁的情况出现，因此使用with是一个好的办法。

​	with类似于我们python的文件操作（with open...），它在程序段结束后会自动close()。同理，with lock也可以达到程序段结束后自动release()的效果。

```python
with lock:
	# do something
```