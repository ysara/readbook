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
    # 如果你只想实现两个特殊方法中的一个, __repr__ 会是更好等等选择, 因为如果一个对象没有 __str__ 函数, 而Python需要调用它的时候, 解释器会用 __repr__ 代替
```

#### 算数运算符

通过 `__add__`, `__mul__`为向量带来 `+`, `*`两个算数运算符