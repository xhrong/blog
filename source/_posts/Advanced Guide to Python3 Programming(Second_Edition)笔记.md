---
title: Advanced Guide to Python3 Programming(Second_Edition)笔记
tags: [Python]
grammar_cjkRuby: true
categories: [Python]
date: 2025-02-15
---

# 《Advanced Guide to Python3 Programming(Second_Edition)》笔记

### Python Type Hints
Type Hint is additional type information that can be used with a function definition to indicate what types parameters should be and what type is returned. 
This is illustrated below: 


``` python
def add(x: int, y: int) -> int:
    return x + y

```

But it is still possible to pass a string into the add() function as far as Python is concerned. 
However, static analysis tools (such as **MyPy**) can be applied to the code to check for such misuse.

### Class Slots
问题：python可以在运行时动态定义类成员，导致无法提前预知和管理类成员

解决方案：使用__slots__


``` python
class Person: 
    __slots__ = ['name', 'age'] 
    def __init__(self, name, age): 
        self.name = name 
        self.age = age 
    def __repr__(self): 
        return f'Person({self.name} is {self.age})'
```

这样，在动态定义__slots__之外的类成员时，将会在运行时报异常AttributeError

__slots__会减小类实例大小，有利于提升性能


### Weak References

弱引用相关接口：
- **weakref.ref(object[, callback])**—This returns a weak reference to the object.
- **weakref.proxy(object[, callback])**—This returns a proxy to object which uses a weak reference.
- **weakref.getweakrefcount(object)**—Return the number of weak references and proxies which refer to object.
- **weakref.getweakrefs(object)**—Return a list of all weak reference and proxy objects which refer to object.

1、将对象的所有强引用被释放时，弱引用自动失效

2、weakref.proxy 获取的弱引用对象是原对象的代理，可直接像原对象一样调用各成员；weakref.ref 获取的弱引用对象，需要额外解一层，其它一致


``` python
class Data: 
    def __init__(self, value): 
        self.value = value 
    def __repr__(self): 
        return f'Data({self.value})' 
    def out(self):
        return "out msg"

data = Data(1) 
weakref_data = weakref.ref(data) 
print(weakref_data().out())
proxy_object = weakref.proxy(data) 
print(proxy_object.out())
```

### Data Classes
数据类是为了方便快捷定义实体类的语法糖

``` python
from dataclasses import dataclass 
@dataclass 
class Trade: 
    """Class for representing Equity Trades""" 
    counter_party1: str 
    counter_party2: str 
    symbol: str 
    amount: int = 0 
```

这种便捷也带来了几个问题：

1、初始化问题:可变类型成员的初始化，不能使用普通的语法，需要使用工厂方法，否则会导致所有实例的该成员变量，实际指向一个对象。（因为成员是在类加载的时候初始化的，而默认的__init__没有对成员变量再次显式初始化）


``` python
from dataclasses import dataclass, field

@dataclass
class Trade:
    counter_party1: str
    counter_party2: str
    symbol: str
    amount: int = 0  # 默认值为 0
    history: list = field(default_factory=list)  # 默认值为空列表
```

2、继承链中参数默认值问题：如果父类使用的默认值来初始化成员，会导致子类的所有成员变量都要提供默认值。（因为没有显式的__init__，子类的默认__init__要想正确初始化所有成员，就要满足所有有默认值的参数就放在参数列表的后面）

3、为了对数据类增加日志监控，同时避免增加方法成员破坏数据类的简洁，额外引入__post_init__成员，该成员在__init__之后执行


### Singleton Metaclass

``` python
class SingletonMetaclass(type):
    _instances = {}

    def __call__(cls, *args, **kwargs):
        print('In SingletonMetaclass.__call__')
        if cls not in cls._instances:
            print(f'Creating new instance of {cls}')
            cls._instances[cls] = super().__call__(*args, **kwargs)
        print('Returning instance')
        return cls._instances[cls]




class Session(metaclass=SingletonMetaclass):
    def __init__(self):
        print('In Session initialiser')
```

_instances中维护了所有使用SingletonMetaclass当metaclass的类的实例。这些实例会一直存在。这是单例模式的特点。




### Threading vs Multprocessing

在 Python 中，threading和multiprocessing模块都用于实现并发编程，但它们有着不同的实现方式和适用场景，下面详细介绍它们的异同点。

**相同点**

- 目标相同：threading和multiprocessing的主要目标都是为了提高程序的执行效率，通过并发执行任务来充分利用计算机的资源，尤其在处理多个独立任务时，能显著减少整体执行时间。
- 使用方式类似：两者在使用上有相似的编程接口。

**不同点**
- 执行单元

    threading：threading模块基于线程实现并发。线程是轻量级的执行单元，它们共享同一进程的内存空间。这意味着线程之间可以方便地共享数据，通过全局变量或类的成员变量就能实现数据的共享和通信。
multiprocessing：multiprocessing模块基于进程实现并发。每个进程都有自己独立的内存空间，不同进程之间的数据是相互隔离的。这保证了数据的安全性，但也增加了进程间通信（IPC）的复杂度。

- 资源开销

    threading：线程的创建和销毁开销相对较小，因为它们共享进程的资源，不需要额外分配大量的系统资源。同时，线程之间的切换速度也比较快，因此在处理一些轻量级任务时，使用线程可以更高效地利用系统资源。
multiprocessing：进程的创建和销毁开销较大，因为每个进程都需要独立的内存空间和系统资源。进程之间的切换也需要更多的系统开销。因此，在创建大量进程时，会消耗较多的系统资源。

- 全局解释器锁（GIL）影响

    threading：由于 Python 的全局解释器锁（GIL）的存在，同一时刻只有一个线程可以执行 Python 字节码。这意味着在 CPU 密集型任务中，多线程并不能充分利用多核 CPU 的优势，反而可能因为线程切换的开销而导致性能下降。
multiprocessing：每个进程都有自己独立的 Python 解释器和 GIL，因此可以充分利用多核 CPU 的优势，在 CPU 密集型任务中表现更好。

- 数据共享和通信

    threading：线程之间共享同一进程的内存空间，因此可以直接访问和修改共享数据。但这种共享也带来了线程安全问题，需要使用锁机制（如threading.Lock、threading.RLock等）来保证数据的一致性。
multiprocessing：进程之间的数据是相互隔离的，不能直接共享数据。需要使用特定的进程间通信（IPC）机制，如管道（multiprocessing.Pipe）、队列（multiprocessing.Queue）、共享内存（multiprocessing.Value、multiprocessing.Array）等来实现数据的交换和共享。

- 适用场景

    threading：适用于 I/O 密集型任务，如网络请求、文件读写等。在这些任务中，线程在等待 I/O 操作完成时会释放 GIL，让其他线程有机会执行，从而提高程序的整体性能。
multiprocessing：适用于 CPU 密集型任务，如科学计算、图像处理等。通过使用多个进程，可以充分利用多核 CPU 的计算能力，提高程序的执行效率。


Demo对比情况：
```python
import multiprocessing
import time
import threading

# 定义一个 CPU 密集型函数，计算斐波那契数列
def fibonacci(n):
    if n <= 1:
        return n
    else:
        return fibonacci(n - 1) + fibonacci(n - 2)

# 定义一个普通函数来打印结果
def print_fibonacci_result(num):
    result = fibonacci(num)
    print(f"Fibonacci({num}) = {result}")


# 多进程方式。多进程会多次运行代码，这种写法是避免每个进程都运行一次代码
if __name__ == '__main__':
    # 要计算的斐波那契数列的项数
    numbers = [35, 35, 35]

    # ============单进程方式================
    start_time_single = time.time()
    for num in numbers:
        result = fibonacci(num)
        print(f"Fibonacci({num}) = {result}")
    end_time_single = time.time()
    print(f"Single-process execution time: {end_time_single - start_time_single} seconds")


    # =============多线程方式===============
    start_time_multi_thread = time.time()
    threads = []
    for num in numbers:
        thread = threading.Thread(target=print_fibonacci_result, args=(num,))
        threads.append(thread)
        thread.start()

    # 等待所有线程执行完毕
    for thread in threads:
        thread.join()

    end_time_multi_thread = time.time()
    print(f"Multi - threaded execution time: {end_time_multi_thread - start_time_multi_thread} seconds")

    # =============多进程方式===============
    start_time_multi = time.time()
    processes = []
    for num in numbers:
        process = multiprocessing.Process(target=print_fibonacci_result, args=(num,))
        processes.append(process)
        process.start()

    # 等待所有进程执行完毕
    for process in processes:
        process.join()

    end_time_multi = time.time()
    print(f"Multi-process execution time: {end_time_multi - start_time_multi} seconds")
```
运行结果：可以看出多线程没有明显提升整体速度
    
    Fibonacci(35) = 9227465
    Fibonacci(35) = 9227465
    Fibonacci(35) = 9227465
    Single-process execution time: 8.550055027008057 seconds
    Fibonacci(35) = 9227465
    Fibonacci(35) = 9227465
    Fibonacci(35) = 9227465
    Multi - threaded execution time: 8.086357593536377 seconds
    Fibonacci(35) = 9227465
    Fibonacci(35) = 9227465
    Fibonacci(35) = 9227465
    Multi-process execution time: 3.1431312561035156 seconds
















### 杂项

#### 类的__str__ 与 __repr__的区别

在 Python 中，__str__ 和 __repr__ 都用于返回对象的字符串表示，但它们的用途和设计目标不同。以下是主要区别：

- 1. 目标用户不同

__str__：面向用户，提供更友好、可读性强的字符串表示。

__repr__：面向开发者，提供明确、无歧义的字符串表示，通常用于调试或日志。

- 2. 调用场景不同

__str__ 会在以下情况被调用：

    使用 print(obj) 或 str(obj) 时。

    在格式化字符串（如 f"{obj}"）中。

__repr__ 会在以下情况被调用：

    直接在交互式环境（如 Python Shell）中输入对象名并按回车时。

    使用 repr(obj) 时。

    当 __str__ 未定义时，会回退到 __repr__。

- 3. 返回值要求不同

__str__：返回的字符串可以是简化的、更人性化的形式。

__repr__：返回的字符串应尽量明确，理想情况下应是一个合法的 Python 表达式，能通过 eval(repr(obj)) 重新生成原对象。

- 4. 默认行为不同

如果未定义 __str__，Python 会默认调用 __repr__。

如果未定义 __repr__，Python 会使用基类（如 object）的默认实现（通常返回 <ClassName object at 0x...>）。

```
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def __str__(self):
        return f"{self.name} ({self.age} years old)"

    def __repr__(self):
        return f"Person('{self.name}', {self.age})"

# 实例化对象
p = Person("Alice", 30)

# 调用 __str__
print(p)           # 输出: Alice (30 years old)
print(str(p))      # 输出: Alice (30 years old)

# 调用 __repr__
print(repr(p))     # 输出: Person('Alice', 30)
p                  # 在交互式环境中直接输入 p 会输出: Person('Alice', 30)
```