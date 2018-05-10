reStructuredText 入门
=====================

-  `reStructuredText
   Primer <http://www.sphinx-doc.org/en/1.4.8/rest.html#>`__

-  `使用Sphinx +
   reST编写文档 <https://www.cnblogs.com/zzqcn/p/5096876.html#_label7_0>`__

toc
------

`Sphinx目录树 <http://www.sphinx-doc.org/en/stable/markup/toctree.html>`_ 

常用格式
---------

标题
~~~~~~

.. code-block:: 

    ===================
    这就是一个标题
    ===================
    
    ----------------
    这也是一个章节标题
    ----------------

reStructuredText 推荐使用这些字符: ``= - ` : . ' " ~ ^ _ * + #``

标题支持: ``! " # $ % & ' ( ) * + , - . / : ; < = > ? @ [ \ ] ^ _ ` { | } ~``

标题可嵌套, 可以只写下半部分

.. code-block:: 

    这个标题和上面的一样
    ===================


段落
~~~~~~~

段落一般隶属于某个章节中，是一块左对齐并且没有其他元素体标记的块。在``.rst``文件中，段落和其他内容的分割是靠空行来完成，如果段落相较于其他的段落有缩进，reStructuredText会解析为引用段落，样式上有些不同。

.. code-block:: 

    这里是一段reStructuredText的内容，它可以很长很长。。。。最后，末尾留出空行表示是本段落的结束即可。

    这里是另外一段reStructuredText的内容，这段内容和上一段之间，乃至后面的其他内容之间都要留出空行进行分割。
    
        这个也是段落，但是由于缩进了，会变成引用段落。显示和直接的段落有点不同

列表
~~~~~~~~~~~~

列表在HTML中被分为两种

- 有序列表（Enumerated Lists）
- 无序列表（Bullet Lists）

在reStructuredText中，我们也能找到这两种列表，还有一种称为定义列表（Definition Lists），这和HTML中的DL一样，在``.rst``文件中可以支持嵌套列表。

无序列表要求文本块是以下面这些字符开始，并且后面紧跟空格，而后跟列表项的内容，其中列表项比趋势左对齐并且有与列表对应的缩进。

常用字符: ``* + - • ‣ ⁃``

.. code-block:: 

    - 这里是列表的第一个列表项
    - 这是第二个列表项
    - 这是第三个列表项
        - 这是缩进的第一个列表项
            注意，这里的缩进要和当前列表项的缩进同步。
    - 第一级的第四个列表项
    
    - 列表项之间要用个空格来分割。

有序列表在格式上和无序列表差不多，但是在使用的前缀修饰符上，使用的不是无序列表那种字符，而是可排序的字符，可以识别的有下面这些：

.. code-block:: 

    arabic numerals: 1, 2, 3, ... (no upper limit).
    uppercase alphabet characters: A, B, C, ..., Z.
    lower-case alphabet characters: a, b, c, ..., z.
    uppercase Roman numerals: I, II, III, IV, ..., MMMMCMXCIX (4999).
    lowercase Roman numerals: i, ii, iii, iv, ..., mmmmcmxcix (4999).

如果你不想使用这些，在你标明第一个条目的序号字符后，第二个开始你还可以使用"#"号来让reStructuredText自动生成需要的序号（Docutils >= 0.3.8）。

.. code-block:: 

    1. 第一项
        巴拉巴拉好多内容在这里。。。
    
    #. 第二项
    
        a. 第二项的第一小项
    
        #. 第二项的第二小项
    
    #. 第三项

定义列表：每个定义列表项里面包含术语（term），分类器（classifiers，可选）， 定义（definition）。术语是一行文字或者短语，分类器跟在术语后面，用“ ： ”(空格，冒号，空格）分隔。定义是相对于术语缩进后的一个块。定义中可以包含多个段落或者其他的内容元素。术语和定义之间可以没有空行，但是在定义列表前后必须要有空行的存在。下面是示例：

.. code-block:: 

    术语1
        术语1的定义
    
    术语 2
        术语2的定义,这是第一段
    
        术语2的定义，第二段
    
    术语 3 : 分类器
        术语3的定义
    
    
    术语 4 : 分类器1 : 分类器2
        术语4的定义

TIPS：在reStructuredText中，还有两种列表，一种是字段列表（Field Lists），一种是选项列表（Option Lists）。由于是rst的语法入门教程，这里不做深入介绍

表格(Table)
~~~~~~~~~~~~~~~

reStructuredText提供两种表格：网格表格（Grid Tables）， 简单表格（Simple Tables）。

 网格表中，共使用的符号有：

- = | +
“-” 用来分隔行， “=“ 用来分隔表头和表体行，"|" 用来分隔列，而"+"用来表示行和列相交的节点，如下面的例子：

.. code-block:: rst

    +------------------------+------------+----------+----------+
    | Header row, column 1   | Header 2   | Header 3 | Header 4 |
    | (header rows optional) |            |          |          |
    +========================+============+==========+==========+
    | body row 1, column 1   | column 2   | column 3 | column 4 |
    +------------------------+------------+----------+----------+
    | body row 2             | Cells may span columns.          |
    +------------------------+------------+---------------------+
    | body row 3             | Cells may  | - Table cells       |
    +------------------------+ span rows. | - contain           |
    | body row 4             |            | - body elements.    |
    +------------------------+------------+---------------------+

