# 流畅的Python

> 英文原版由O'Reilly Media, Inc.出版, 2015. 中文版2017年5月第一版

## 前言

避免倾向于寻求自己熟悉的东西, 而失去了使用Python独有的特性的机会(比如元组拆包, 描述符)

先熟悉 doctest

书中代码都可以尝试使用 `python3 -m doctest example_script.py` 来验证

获取书籍代码

[https://github.com/fluentpython/example-code](https://github.com/fluentpython/example-code)

## Python数据模型

Python最好的品质之一是一致性.

数据模型其实是对Python框架的描述, 它规范了这门语言自身构建模块的接口, 这些模块包括但不限于序列, 迭代器, 函数, 类和上下文管理器.

不管在哪种框架下写程序, 都会花费大量时间去实现那些会被框架本身调用的方法.

Python解释器碰到特殊的句法时, 会使用特殊方法去激活一些基本的对象操作, 这些特殊方法的名字以两个下划线开头, 以两个下划线结尾(例如 `__getitem__`), 比如 `obj[key]` 的背后就是`__getitem__` 方法, 为了能求得 `my_collection[key]` 的值, 解释器实际上会调用 `my_collection.__getitem__(key)`

这些特殊的方法名能让你自己的对象实现和支持以下的语言架构, 并与之交互:

- 迭代
- 集合类
- 属性访问
- 运算符重载
- 函数和方法的调用
- 对象的创建和销毁
- 字符串表示形式和格式化
- 管理上下文(即with块)

### 一摞Python风格的纸牌

看代码

Python内置了从一个序列中随机选出一个元素的函数 `random.choice`, 我们可以直接把它用在这一摞纸牌的实例上.

```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-

import collections

Card = collections.namedtuple('Card', ['rank', 'suit'])


class FrenchDeck:
    ranks = [str(n) for n in range(2, 11)] + list('JQKA')
    suits = 'spades diamonds clubs hearts'.split()

    def __init__(self):
        self._card = [Card(rank, suit) for suit in self.suits
                      for rank in self.ranks]

    def __len__(self):
        return len(self._card)

    def __getitem__(self, item):
        return self._card[item]

beer_card = Card('7', 'diamonds')
print(beer_card)
# Card(rank='7', suit='diamonds')

deck = FrenchDeck()
print(len(deck))
# 52

print(deck[0])
print(deck[-1])


# 随机抽取一张纸牌
from random import choice
print(choice(deck))


# __getitem__ 方法把 [] 操作交给了 self._card 列表, 所以我们的deck类自动支持切片(slicing)操作

print(deck[:3])

print(deck[12::13])

# 实现了 __getitem__ 方法, 这一摞牌就变成可迭代的了

for card in deck:
    print(card)

# 反向迭代

for card in reversed(deck):
    print(card)
```

迭代通常是隐式的, 譬如说一个集合类型没有实现 `__contains__` 方法, 那么 `in` 运算符就会按顺序做一次迭代搜索. 于是, `in` 可以用在我们的 `FrenchDeck` 类上, 因为它可迭代

```python
# in

print(Card('Q', 'hearts') in deck)
# True

print(Card('7', 'beasts') in deck)
# False
```

> 排序

按照常规, 用点数判定扑克大小, 2最小, A最大, 同时判定花色. 黑桃最大, 红桃次之, 方块再次, 梅花最小. 按照这个规则给扑克排序

```python
# 排序
suit_values = dict(spades=3, hearts=2, diamonds=1, clubs=0)


def spades_high(card):
    rank_value = FrenchDeck.ranks.index(card.rank)
    return rank_value * len(suit_values) + suit_values[card.suit]

for card in sorted(deck, key=spades_high):
    print(card)
```

> 洗牌

FrenchDeck 是不能洗牌的, 因为这摞牌是不可变的, 卡牌和它的位置都是固定的, 除非破坏这个类的封装性, 直接对 `_cards` 进行操作. 其实可以使用 `__setitem__` 方法, 洗牌功能就不是问题了

### 如何使用特殊方法

特殊方法的存在是为了被Python解释器调用, 我们并不需要调用它.

没有 `my_object.__len__()` 这种写法, 而是直接使用 `len(my_object)`. 在执行 `len(my_object)` 的时候, 如果 `my_object` 是一个自定义类的对象, 那么Python会自己调用其中由我们实现的 `__len__` 方法

如果是Python内置类型, CPython会抄近路, 直接返回 `PyVarObject` 里的 `ob_size`属性, `PyVarObject` 是表示内存中长度可变的内置对象的C语言结构体. 直接读取这个值比调用一个方法快很多.

通过内置的函数,(len, iter, str等)来使用特殊方法是最好的选择. 这些内置函数不仅会调用特殊方法, 通常还会提供额外的好处, 而对于内置的类来说, 它们的速度更快

### 模拟数值类型

```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-

from math import hypot


class Vector:
    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y

    def __repr__(self):
        return 'Vector(%r, %r)' % (self.x, self.y)

    def __abs__(self):
        return hypot(self.x, self.y)

    def __bool__(self):
        return bool(abs(self))

    def __add__(self, other):
        x = self.x + other.x
        y = self.y + other.y
        return Vector(x, y)

    def __mul__(self, scalar):
        return Vector(self.x * scalar, self.y * scalar)

v1 = Vector(2, 4)
v2 = Vector(2, 1)
print(v1 + v2)

v = Vector(3, 4)
print(abs(v))

print(v * 3)
print(abs(v*3))
```

```python
Vector(4, 5)
5.0
Vector(9, 12)
15.0
```

#### 字符串表示形式

```python
    # 字符串表示形式, 把一个对象用字符串的形式表达出来以便辨认
    # repr 通过 __repr__ 这个特殊方法来得到一个对象的字符串表示形式
    # 如果没有实现 __repr__ , 我们在控制台打印一个向量的实例时, 得到的可能就是地址

    # __repr__ 和 __str__ 的区别在于, 后者是在str()调用的时候被使用, 或是在用print函数打印一个对象的时候才被调用
    # 如果你只想实现两个特殊方法中的一个, __repr__ 会是更好的选择, 因为如果一个对象没有 __str__ 函数, 而Python需要调用它的时候, 解释器会用 __repr__ 代替
```

#### 算数运算符

通过 `__add__`, `__mul__`为向量带来 `+`, `*`两个算数运算符

#### 自定义布尔值

如果要让 `Vector.__bool__`更高效, 可以采用下面的实现方式

```python
def __bool__(self):
    return bool(self.x or self.y)
```

### 特殊方法

[https://docs.python.org/3/reference/datamodel.html](https://docs.python.org/3/reference/datamodel.html)

### 为什么len不是普通方法

CPython会直接从C结构体中读取对象的长度, 完全不会调用任何方法. 获取一个集合中元素的数量是一个很常见的操作, 在str, list, memoryview等类型上, 这个操作必须高效

len之所以不是一个普通方法, 是为了让Python自带的数据结构可以走后门, abs也是同理.

### 小结

通过特殊方法, 自定义数据类型可以表现得跟内置类型一样, 从而让我们写出更具表现力的代码--更具Python风格的代码.

- `__repr__` 方便我们调试和记录日志
- `__str__` 则是给终端用户看的

## 序列构成的数组

### 内置序列类型概览

Python标准库用C实现了丰富的序列类型

- 容器序列: list, tuple和collections.deque, 这些序列能存放不同类型的数据
- 扁平序列: str, bytes, bytearray, memoryview, array.arrya, 这些序列只能容纳一种类型

容器序列存放的是它们所包含的任意类型的对象的引用, 扁平序列里存放的是值, 热不是引用.

换句话说, 扁平序列其实是一段连续的内存空间,. 扁平序列其实更加紧凑, 但是它里面只能存放诸如字符, 字节和数值这种基础类型.

序列还能按照能否被修改来分类

- 可变序列: list, bytearray, array.array, collections.deque 和 memoryview
- 不可变序列: tuple, str, bytes

### 列表推导和生成器表达式

#### 列表推导和可读性

不要滥用列表推导, 通常的原则是, 只用列表推导式来创建新的列表, 并且尽量保持简短.

> Python会忽略`[]`,`{}`,`()`中的换行, 因此如果你的代码里有多行的列表, 列表推导, 等等, 可以省略不好看的续行符
> python2.x中, 列表推导会有变量泄露的问题, 不过python3.x已经解决了

#### 笛卡尔积

#### 生成器表达式

生成器表达式的语法和列表推导差不多, 只不过把方括号换成圆括号而已

### 元组不仅仅是不可变的列表

元组除了用作不可变列表, 还可以用于没有字段名的记录.

#### 元组和记录

元组其实是对数据的记录: 元组中的每个元素都存放了记录中一个字段的数据, 外加这个字段的位置.

```python
>>> lax_coordinates = (33.9425, -118.408056)
>>> city, year, pop, chg, area = ('Tokyo', 2003, 32450, 0.66, 8014)
>>> traveler_ids = [('USA', '31195855'), ('BRA', 'CE342567')]
>>> for passport in sorted(traveler_ids):
...   print('%s/%s' % passport)
...
BRA/CE342567
USA/31195855
>>> for country, _ in traveler_ids:
...   print(country)
...
USA
BRA
```

#### 元组拆包

我们可以把元组 `('Tokyo', 2003, 32450, 0.66, 8014)` 里的元素分别赋值给变量 city, year, pop, chg和area, 而这所有的赋值我们只用一行声明就完成了, 同样在后面, 一个 `%` 运算符就把passport元组的元素对应到了print函数的格式字符串空档中, 这两个都是对元组拆包的应用.

元组拆包可以应用到任何可迭代对象上, 唯一要求是, 被可迭代对象中的元素数量必须要跟接受这些元素的元组的空档数一致. 除非我们用 `*` 来表示忽略多余的元素.

可以用 `*` 运算符把一个可迭代对象拆开作为函数的参数:

```python
>>> divmod(20, 8)
(2, 4)
>>> t = (20, 8)
>>> divmod(*t)
(2, 4)
```

用 `*` 处理剩下的元素

```python
>>> a, b, *rest = range(5)
>>> a, b, rest
(0, 1, [2, 3, 4])
>>> a, b, *rest = range(2)
>>> a, b, rest
(0, 1, [])
>>>
```

平行赋值中, `*` 前缀只能用在一个变量名前面, 但是这个变量可以出现在赋值表达式的任意位置

```python
>>> a, *body, c, d = range(5)
>>> a, body, c, d
(0, [1, 2], 3, 4)
>>> *body, b, c, d = range(5)
>>> body, b, c, d
([0, 1], 2, 3, 4)
```

#### 嵌套元组拆包

看代码

#### 具名元组

collections.namedtuple 是一个工厂函数, 它可以用来构建一个带字段名的元组和一个有名字的类

> 用namedtuple构建的类的实例说消耗的内存跟元组是一样的, 字段名都被存在对应的类里面. 这个实例跟普通对象实例比起来也要小一些. python不会用 `__dict__` 来存放这些实例的属性

```python
>>> from collections import namedtuple
>>> City = namedtuple('City', 'name country population coordinates')
>>> tokyo = City('Tokyo', 'JP', 36.933, (35.689772, 139.691667))
>>> tokyo
City(name='Tokyo', country='JP', population=36.933, coordinates=(35.689772, 139.691667))
>>> tokyo.population
36.933
>>> tokyo.coordinates
(35.689772, 139.691667)
>>> tokyo[1]
'JP'
>>> tokyo.name
'Tokyo'
>>> tokyo.country
'JP'
```

创建一个具名元组需要两个参数, 一个是类名, 另一个是类的各个字段的名字. 后者是可以是由数个字符串组成的可迭代对象, 或者是由空格分隔开的字段名组成的字符串.

#### 作为不可变列表的元组

### 切片

在Python里, 列表, 元组和字符串这类序列类型都支持切片操作.

#### 为什么切片和区间会忽略最后一个元素

在切片和区间操作里不包括区间范围的最后一个元素是Python的风格, 这个习惯符合Python, C和其他语言里以0作为起始下标的传统. 这样有以下好处

- 当只有最后一个位置信息时, 我们可以很快看出切片和区间里有几个元素: `range(3)`, `my_list[:3]`都返回3个元素
- 当起止位置信息都可见时, 我们可以快速计算出切片和区间的长度, 用后一个数减取第一个下标即可
- 这样做也让我们可以利用任意一个下标来把序列分隔成不重叠的两部分, 只要写成 `my_list[:x]` 和 `my_list[x:]` 就可以了

#### 对对象进行切片

对 `seq[start:stop:step]` 进行求值的时候, Python会调用 `seq.__getitem__(slice(start, stop, step))`.

#### 多维切片和省略

#### 给切片赋值

如果把切片放在赋值语句的左边, 或把它作为del操作的对象, 我们就可以对序列进行嫁接, 切除或就地修改操作.

```python
>>> l = list(range(10))
>>> l
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
>>> l[2:5]=[20,30]
>>> l
[0, 1, 20, 30, 5, 6, 7, 8, 9]
>>> l[2:5] = 100
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: can only assign an iterable
>>> l[2:5] = [100]
>>> l
[0, 1, 100, 6, 7, 8, 9]
```

如果赋值的对象是一个切片, 那么赋值语句右边必须是个可迭代对象. 即便只有单独的一个值

### 对序列使用`+`和`*`

```python
>>> l = [1, 2, 3]
>>> l*5
[1, 2, 3, 1, 2, 3, 1, 2, 3, 1, 2, 3, 1, 2, 3]

>>> 5 * 'abcd'
'abcdabcdabcdabcdabcd'
```

`+`, `*` 不修改原有的操作对象, 而是构建一个全新的序列

> `a*n`这个语句中, 序列a里的元素是对其他可变对象的**引用**的话, 需要注意, 结果可能会出乎意料

建立由列表组成的列表

看代码

其实就是关于引用的问题

### 序列的增量赋值

`+=`, `*=` 的表现取决于第一个操作对象.

`+=` 背后的特殊方法是 `__iadd__` (就地加法), 如果类没有实现这个方法, Python会退一步调用 `__add__`.

如果没有实现 `__iadd__`, `a+=b`这个表达式的效果和 `a = a + b` 一样, 首先计算 `a+b`, 得到一个新对象, 然后赋值给 `a`. 这个表达式中, 变量名会不会被关联到新的对象, 完全取决于这个类型有没有实现 `__iadd__` 这个方法.

总体来说, 可变序列一般都实现了 `__iadd__` 方法, 因此 `+=` 是就地加法, 而不可变序列根本就不支持这个操作.

`*=` 对应的是 `__imul__`

对不可变序列进行重复拼接操作的话, 效率很低, 因为每次都有一个新对象, 而解释器需要把原来对象中的元素先复制到新的对象里, 然后再追加新的元素.

```python
t = (1, 2, [30, 40])
t[2] += [50, 60]
>>> t[2] += [50, 60]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'tuple' object does not support item assignment
>>> t
(1, 2, [30, 40, 50, 60])

# 如果写成 t[2].extend([50, 60]) 可以避免异常
```

> Python Tutor

对Python运行原理进行可视化分析

```python
>>> import dis
>>> dis.dis('s[a] +=b')
  1           0 LOAD_NAME                0 (s)
              2 LOAD_NAME                1 (a)
              4 DUP_TOP_TWO
              6 BINARY_SUBSCR
              8 LOAD_NAME                2 (b)
             10 INPLACE_ADD
             12 ROT_THREE
             14 STORE_SUBSCR
             16 LOAD_CONST               0 (None)
             18 RETURN_VALUE
```

- 不要把可变对象放在元组里面.
- 增量赋值不是一个原子操作. 虽然刚刚抛出了异常, 但是还是完成了操作.
- 查看Python的字节码并不难, 而且它对我们了解代码背后的运行机制很有帮助.

### list.sort方法和内置函数sorted

list.sort 会就地排序列表, 不会把原列表复制一份. 方法返回值为 None. 提醒方法不会新建一个列表. 这种情况返回None是Python的一个惯例: 如果一个函数或者方法对对象进行的是就地改动, 那它就返回一个None, 好让调用者知道传入的参数发生了变动, 而且并未产生新的对象.

内置函数sorted, 会新建一个列表作为返回值. 可以接受任何形式的可迭代对象作为参数, 甚至包括不可变序列或生成器. 不管sorted接受的是怎样的参数, 最后都会返回一个列表.

已排序的序列可以用来进行快速搜索, 标准库的**bisect**模块给我们提供了**二分查找算法**, 还有bisect.insort可以让已排序的序列保持有序.

### 用bisect来管理已排序的序列

#### 用bisect来搜索

另一个排序集合模块[http://code.activestate.com/recipes/577197-sortedcollection/](http://code.activestate.com/recipes/577197-sortedcollection/), 模块里集成了 bisect 功能, 比独立的bisect更易用.

### 当列表不是首选时

面对各类需求, 我们可能会有更好的选择, 比如, 要存放1000万个浮点数的话, 数组(array)的效率要高得多, 因为数组在背后存的并不是float对象, 而是数字的机器翻译, 也就是字节表述.

再比如, 如果要频繁对序列做先进先出的操作, deque(双端列表)的速度可能会更快.

#### 数组

如果我们需要一个只包含数字的列表, array.array 比 list 更高效. 数组支持所有跟可变序列有关的操作, 同时海通共文件读取和存入文件的更快的方法, 如, `.frombytes` 和 `.tofile`.

查看代码

#### 内存视图

memoryview是一个内置类, 它能让用户在不复制内容的情况下操作同一个数组的不同切片.

> 内存视图其实是泛化和去数学化的NumPy数组.

#### NumPy和SciPy

#### 双向队列和其他形式的队列

双向队列 append 和 popleft 都是原子操作, 就是说deque可以在多线程程序中安全地当做先进先出的栈使用, 而使用者不用担心资源锁的问题.

除了deque之外, 还有些其他的Python标准库也有对队列的实现

可以查阅书籍, 看相关应用, 或特点

- queue
- multiprocessing
- asyncio
- heapq

### 2. 小结

