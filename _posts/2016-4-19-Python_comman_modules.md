---
layout:     post
title:      "Python 常用模块记录"
subtitle:   ""
date:       2016-4-19
author:     "YuanBao"
header-img: "img/post-python.jpg"
header-mask: 0.25
catalog: true
tags:
 - programming
 - python
---

>什么是常用函数？经常使用又经常忘记的函数就是常用函数。

下图中展示了本文内容的脉络。本文仅仅作为 Python 的使用笔记，不涉及 python 虚拟机的内部实现。

![](/img/PythonPopularFunc.png){: width="500px" height="400px" }

## itertools -- 控制迭代的模块

有句话说，当你想针对迭代器做某些操作的时候，你就应该去看看 `itertools` 模块中是不是有你想用的功能。如果有也相应的功能，就不要重复造轮子了，因为你实现的百分之99.99没有 `itertools` 的效率高。`itertools` 模块内的函数（或者类）都非常常用，这里只是介绍几个作为代表：

* **`chain(iter1, iter2, ...)`**：将多个迭代器连接起来，生成一个大的迭代器（使用 `yield from` 也可以实现类似的功能）
* **`dropwhile(callable, iter)`**：该函数从迭代器 `iter` 头反复调用 callable 函数，丢弃掉所有满足条件的元素，直到遇到第一个不满足条件的元素。

<!--more-->

通过一个例子来更好地解释 `dropwhile` 的功能。假如你有一个文件，文件开头有几行使用 "##" 开始的注释，那么你可以使用下面的代码跳过这几行注释：

```python
from itertools import dropwhile

with open('/etc/passwd') as f:
    for line in dropwhile(lambda line: line.startswith('##'), f):
        print(line, end='')
```

你可能会觉得 `dropwhile` 的功能与列表推导和列表过滤比较相似。实际上它们逻辑并不相同，列表推导可以过滤到整个迭代器中的元素，而 `dropwhile` 则只考虑迭代器的头部。下面这个例子可以清楚地说明这个情况：

```python
patterns = ["xxb", "xxc", "cbc", "xxd", "xxe"]

res1 = [p for p in patterns if not p.startswith("xx")]
res2 = list(dropwhile(lambda p: p.startswith("xx"), patterns))

#########################
res1 = ['cbc']
res2 = ['cbc', 'xxd', 'xxe']
```

* **`islice`**：给迭代器赋予切片的功能。正常情况下，迭代器是不能进行切片操作的，但是通过使用 `islice` 可以实现迭代器的切片操作。例如：

```python
from itertools import islice

def range_iter(begin, end):
    if begin < end:
        yield begin
        begin += 1

items = range_iter(0, 10)

for x in islice(items, 3, None):
    print(x) 
```

注意 `islice` 函数和 `dropwhile` 函数的应用场景是不同的。`izip` 函数需要你清楚的知道，你应该从哪个位置开始操作迭代器，而 `dropwhile` 则以一种更加抽象的方式帮你筛选掉迭代器头部的某些元素。

* **`izip` 以及 `longest_izip`**：针对迭代器实现 `zip` 的功能和 `longest_zip` 的功能。这个很好理解，不多说。

## operator 模块

`operator` 模块中的函数和类可以轻松地返回一个 callable 对象供我们使用，这在某些需要排序的场景中应用非常方便。

* **`attrgetter`**：该函数将属性名封装成为一个 callable 对象，该 callable 对象可以在类的实例上直接调用，并返回相应属性的值。看下面的例子：

```python
from operator import attrgetter

class User:
    def __init__(self, name, mail):
        self.name = name
        self.mail = mail

user = User("wky", "abc@gmail.com")
f = attrgetter(("name", "mail"))

print f(user)  # (user.name, user.mail)

```

`attrgetter` 返回的 callable 对象在很多时候都会极大地方便我们编码。例如我们有一个 User 实例的列表，我们想以 User 实例中的 `name` 属性对列表进行排序，我们可以这么写：

```python
from operator import attrgetter
sorted(users, key=attrgetter('name'))
```

* **`itemgetter`**：同 `attrgetter` 类似，该函数可以将一个 key 封装成为一个 callable 对象，该对象可以在任何支持 `__getitem__` 的实例上（例如字典）调用。这两个函数都在排序不支持原生比较的对象的应用场景中使用非常广泛。


## collections 模块

`collections` 模块为我们定义了很多易用的数据结构，当我们需要某个复杂但是常见的数据结构时，最好可以去 collections 模块中看一下。

* **`defaultdict`**：有时候我们需要可以把一个键映射到多个值上的字典，这时候我么就可以使用 `defaultdict` 了（当然你也可以自己实现）。需要注意的是，`defaultdict` 会默认为每个 key 初始化值，所以你只需要关注元素添加操作，例如：


```python
from collections import defaultdict

d = defaultdict(list)
d['a'].append(1)  #自动创建键值'a'对应的列表
d['a'].append(2)
d['b'].append(4)

d = defaultdict(set)
d['a'].add(1)    #自动创建键值'a'对应的集合
d['a'].add(2)
d['b'].add(4)
```

* **`ChainMap`**：如果你想要把多个字典连接成为一个大的字典，那么 `ChainMap` 是你最好的选择：

```python
a = {'x': 1, 'z': 3 }
b = {'y': 2, 'z': 4 }

from collections import ChainMap
c = ChainMap(a,b)
print(c['x']) # Outputs 1 (from a)
print(c['y']) # Outputs 2 (from b)
print(c['z']) # Outputs 3 (from a)
```

需要注意的是，如果两个字典 `a` 和 `b` 有相同的键值，那么总是返回第一个字典中的键对应的值。同样地，如果你修改了 `c` 中的 `x` 对应的值，那么修改始终反应在第一个字典 `a` 上面。

事实上你可能觉得你可以通过字典的 update 操作来实现将两个字典合并的功能。比如：

```python
a = {'x': 1, 'z': 3 }
b = {'y': 2, 'z': 4 }
merged = dict(b)
merged.update(a)
```

然而，这种通过 update 实现的方案与 `ChainMap` 并不相同。`ChainMap` 并不创建新的字典，因此对于原字典的改变能够反映到 `ChainMap` 中。而 `update` 的实现方案创建了新的字典 `merged`，它并不跟原字典有任何关系。

* **`namedtuple`**：通过使用一个普通的元组对象来实现针对一个序列的命名，这将极大提高序列
操纵的可读性。

假设你从数据库中查询一个名为 `'wky'` 的用户信息，得到的结果是 `res=['wky', 'male', '123456']`。那么在编码中你要访问用户姓名只能通过 `res[0]`，访问用户性别只能使用 `res[1]`。如果这样的硬编码变得很多，你的整个代码将变得几不可读。

使用 `namedtuple` 将可以避免这种问题。我们可以这样来定义一个 `namedtuple`，然后通过属性访问操纵`res`：

```python
from collections import namedtuple

User = namedtuple('User', ["username", "gender", "phone"])

user = User(*res)

print user.username, user.gender, user.phone
```

这样是不是方便且易懂？

## callable 对象

前面很多次提到了 callable 对象，那么什么是一个 callable 对象呢？简而言之，**在 Python 中一个 callable 对象就是一个可以当做函数调用的对象**，我们在很多时候都需要使用这种 callable 对象。

#### lambda 表达式

当一些函数很简单，仅仅只是计算一个表达式的值的时候，就可以使用 lambda 表达式来代替了，例如：

```python
add = lambda x, y: x + y
add(2,3)  # 5
```

大多数情况下，lambda 表达式都是用在排序或者数据 reduce 的场景下，用以产生一个 callable 对象。尽管 lambda 表达式允许你定义简单函数，但是它的使用是有限制的。 **你只能指定单个表达式，它的值就是最后的返回值**。也就是说不能包含其他的语言特性了，包括多个语句、条件表达式、迭代以及异常处理等等。

需要注意的是 lambda 表达式是在**运行的时候绑定值**，而不是在定义的时候绑定值（这跟函数的默认参数正好相反）。为了理解这一点，可以看下面的代码：

```python
x = 10
a = lambda y: x+y
x = 20
b = lambda y: x+y

print a(10)  ## 结果是30
print b(10)  ## 结果也是30
```

如果想要让 lambda 表达式在定义时就捕获到值，可以这么实现:

```python
x = 10
a = lambda y, x=x: x+y
x = 20

print a(10)  ##结果是20
```

#### partial

`partial` 在 Python 中是一个非常实用的函数。有很多的场景下，看似很难完成的工作都可以使用 `partial` 来完成。这个函数能够将某个函数和它的一部分参数进行打包，然后返回一个 callable 对象。我们只需要提供剩余的参数，既可以成功完成函数调用。因此，`partial` 最直观的功能就是**减少函数的调用参数**。

举个例子，假如我们有一个记录二维坐标点的列表，和一个计算点与点之间坐标距离的函数：

```python
points = [ (1, 2), (3, 4), (5, 6), (7, 8) ]

import math
def distance(p1, p2):
    x1, y1 = p1
    x2, y2 = p2
    return math.hypot(x2 - x1, y2 - y1)
```

我们想要将列表 `points` 按照与点 (2,3) 的距离进行排序，那么我们可以轻松地通过 `partial` 来实现

```python
points.sort(key=partial(distance,(2,3)))
```
其中 `partial(distance,(2,3))` 返回了一个可迭代对象，其将 `distance` 函数和第一个参数 `p1` 打包在一起，然后我们只需要提供第二个参数 `p2`，就可以调用函数 `distance` 了。

很多情况下，当我们需要传递一个具有很多参数的函数时，我们就应该考虑能不能使用 `partial` 来帮助我们解决问题。

#### iter

最后要介绍的这个函数 `iter` 同样是非常易用，但却经常被忽视。这个函数的定义是这样的 `iter(callable, sentinel)`，其接受一个 callable 对象，然后迭代执行该对象，知道执行结果遇到 `sentinel` 为止。

例如我们在进行读文件或者网络操作时，经常会写出这样的代码：

```python
def reader(s):
    while True:
        data = s.recv(1024)
        if data == b'':
            break
        process_data(data)
```

如果借助于 `iter` 函数，我们可以直接：

```python
def reader2(s):
    for data in iter(lambda: s.recv(CHUNKSIZE), b''):
        process_data(data)
```

这样处理起来就会非常方便。

