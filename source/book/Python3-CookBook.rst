Python3 CookBook
==================

数据结构和算法
----------------

1.3 保留最后N个元素
~~~~~~~~~~~~~~~~~~~~

在队列两端插入或删除元素时间复杂度都是 O(1) ，而在列表的开头插入或删除元素的时间复杂度为 O(N) 。

1.4 查找最大或最小的N个元素
~~~~~~~~~~~~~~~~~~~~~~~~~

heapq模块有两个函数：``nlargest()`` 和 ``nsmallest()``

如果你想在一个集合中查找最小或最大的N个元素，并且N小于集合元素数量，那么这些函数提供了很好的性能。 因为在底层实现里面，首先会先将集合数据进行堆排序后放入一个列表中：

堆数据结构最重要的特征是 ``heap[0]`` 永远是最小的元素。并且剩余的元素可以很容易的通过调用 ``heapq.heappop()`` 方法得到， 该方法会先将第一个元素弹出来，然后用下一个最小的元素来取代被弹出元素(这种操作时间复杂度仅仅是``O(log N)``，N是堆大小)。 

当要查找的元素个数相对比较小的时候，函数 ``nlargest()`` 和 ``nsmallest()`` 是很合适的。 如果你仅仅想查找唯一的最小或最大(N=1)的元素的话，那么使用 ``min()`` 和 ``max()`` 函数会更快些。 类似的，如果N的大小和集合大小接近的时候，通常先排序这个集合然后再使用切片操作会更快点 ( ``sorted(items)[:N]`` 或者是 ``sorted(items)[-N:]`` )。 需要在正确场合使用函数 ``nlargest()`` 和 ``nsmallest()`` 才能发挥它们的优势 (如果N快接近集合大小了，那么使用排序操作会更好些)。

1.6 字典中的键映射多个值
~~~~~~~~~~~~~~~~~~~~~~

需要注意的是， defaultdict 会自动为将要访问的键(就算目前字典中并不存在这样的键)创建映射实体。 如果你并不需要这样的特性，你可以在一个普通的字典上使用 setdefault() 方法来代替。比如：

.. code-block:: python

    d = {} # A regular dictionary
    d.setdefault('a', []).append(1)
    d.setdefault('a', []).append(2)
    d.setdefault('b', []).append(4)

一般来讲，创建一个多值映射字典是很简单的。但是，如果你选择自己实现的话，那么对于值的初始化可能会有点麻烦， 你可能会像下面这样来实现：

.. code-block:: python

    d = {}
    for key, value in pairs:
        if key not in d:
            d[key] = []
        d[key].append(value)

如果使用 defaultdict 的话代码就更加简洁了：

.. code-block:: python

    d = defaultdict(list)
    for key, value in pairs:
        d[key].append(value)

1.7 字典排序
~~~~~~~~~~~~~

OrderedDict 内部维护着一个根据键插入顺序排序的双向链表。每次当一个新的元素插入进来的时候， 它会被放到链表的尾部。对于一个已经存在的键的重复赋值不会改变键的顺序。

需要注意的是，一个 OrderedDict 的大小是一个普通字典的两倍，因为它内部维护着另外一个链表。 所以如果你要构建一个需要大量 OrderedDict 实例的数据结构的时候(比如读取100,000行CSV数据到一个 OrderedDict 列表中去)， 那么你就得仔细权衡一下是否使用 OrderedDict 带来的好处要大过额外内存消耗的影响。

1.9 查找两字典的相同点
~~~~~~~~~~~~~~~~~~~~~

这些操作也可以用于修改或者过滤字典元素。 比如，假如你想以现有字典构造一个排除几个指定键的新字典。 下面利用字典推导来实现这样的需求：

.. code-block:: python

    # Make a new dictionary with certain keys removed
    c = {key:a[key] for key in a.keys() - {'z', 'w'}}
    # c is {'x': 1, 'y': 2}

一个字典就是一个键集合与值集合的映射关系。 字典的 keys() 方法返回一个展现键集合的键视图对象。 键视图的一个很少被了解的特性就是它们也支持集合操作，比如集合并、交、差运算。 所以，如果你想对集合的键执行一些普通的集合操作，可以直接使用键视图对象而不用先将它们转换成一个set。

字典的 items() 方法返回一个包含(键，值)对的元素视图对象。 这个对象同样也支持集合操作，并且可以被用来查找两个字典有哪些相同的键值对。

尽管字典的 values() 方法也是类似，但是它并不支持这里介绍的集合操作。 某种程度上是因为值视图不能保证所有的值互不相同，这样会导致某些集合操作会出现问题。 不过，如果你硬要在值上面执行这些集合操作的话，你可以先将值集合转换成set，然后再执行集合运算就行了。


1.10 删除序列相同元素并保持顺序
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

如果序列上的值都是 ``hashable`` 类型，那么可以很简单的利用集合或者生成器来解决这个问题。比如：

.. code-block:: python

    def dedupe(items):
        seen = set()
        for item in items:
            if item not in seen:
                yield item
            seen.add(item)

下面是使用上述函数的例子：

.. code-block:: python

    >>> a = [1, 5, 2, 1, 9, 1, 5, 10]
    >>> list(dedupe(a))
    [1, 5, 2, 9, 10]
    >>>

这个方法仅仅在序列中元素为 ``hashable`` 的时候才管用。 如果你想消除元素不可哈希(比如 dict 类型)的序列中重复元素的话，你需要将上述代码稍微改变一下，就像这样：

.. code-block:: python

    def dedupe(items, key=None):
        seen = set()
        for item in items:
            val = item if key is None else key(item)
            if val not in seen:
                yield item
                seen.add(val)

这里的key参数指定了一个函数，将序列元素转换成 hashable 类型。下面是它的用法示例：

.. code-block:: python

    >>> a = [ {'x':1, 'y':2}, {'x':1, 'y':3}, {'x':1, 'y':2}, {'x':2, 'y':4}]
    >>> list(dedupe(a, key=lambda d: (d['x'],d['y'])))
    [{'x': 1, 'y': 2}, {'x': 1, 'y': 3}, {'x': 2, 'y': 4}]
    >>> list(dedupe(a, key=lambda d: d['x']))
    [{'x': 1, 'y': 2}, {'x': 2, 'y': 4}]
    >>>

1.12 序列中出现次数最多的元素
~~~~~~~~~~~~~~~~~~~~~~~~~~~

``collections.Counter``

``Counter`` 实例一个鲜为人知的特性是它们可以很容易的跟数学运算操作相结合。比如：

1.13 通过某个关键字排序一个字典列表
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

使用 ``operator`` 模块的 ``itemgetter`` 函数

1.14 排序不支持原生比较的对象
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

选择使用lambda函数或者是 attrgetter() 可能取决于个人喜好。 但是， attrgetter() 函数通常会运行的快点，并且还能同时允许多个字段进行比较。 这个跟 operator.itemgetter() 函数作用于字典类型很类似(参考1.13小节)。 例如，如果 User 实例还有一个 first_name 和 last_name 属性，那么可以向下面这样排序：

.. code-block:: python

    by_name = sorted(users, key=attrgetter('last_name', 'first_name'))

同样需要注意的是，这一小节用到的技术同样适用于像 min() 和 max() 之类的函数。比如：

.. code-block:: python

    >>> min(users, key=attrgetter('user_id'))
    User(3)
    >>> max(users, key=attrgetter('user_id'))
    User(99)
    >>>



