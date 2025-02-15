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

```
def add(x: int, y: int) -> int:
    return x + y

```

But it is still possible to pass a string into the add() function as far as Python is concerned. 
However, static analysis tools (such as **MyPy**) can be applied to the code to check for such misuse.

### Class Slots
问题：python可以在运行时动态定义类成员，导致无法提前预知和管理类成员

解决方案：使用__slots__

```
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

```
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
```
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

```
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