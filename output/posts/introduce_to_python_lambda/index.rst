python的lambda函数介绍
==========================
.. title: python的lambda函数介绍
.. slug: introduce_to_python_lambda
.. date: 2012-10-23 20:17:59 UTC+08:00
.. tags: python
.. category:
.. link:
.. description:
.. type: text


今天在论坛上看到有人问的一个关于如何从一个python的字典中取到value中最大的那个key值，里面用到了 ``lambda`` 函数，今天那就大致介绍下 ``lambda`` 是个什么东东。
python支持创建一种匿名的函数（一种没绑定名字的函数），这种函数叫做lambda，这个和fp(函数编程)里面的lambda的含义并不是完全一致，下面这段代码将展示 ``lambda`` 和普通函数之间的区别

 >>> def f (x): return x**2
 ...
 >>> print f(8)
 64
 >>>
 >>> g = lambda x: x**2
 >>>
 >>> print g(8)
 64

g()是一个 lambda 函数，完成同上面普通函数相同的事情。注意这里的简短的语法：在参数列表周围没有括号，而且忽略了 return 关键字 (隐含存在，因为整个函数只有一行)。而且，该函数没有函数名称，但是可以将它赋值给一个变量进行调用。
使用 lambda 函数时甚至不需要将它赋值给一个变量。这可能不是世上最有用的东西，它只是展示了 lambda 函数只是一个内联函数。

总的来说，lambda 函数可以接收任意多个参数 (包括可选参数) 并且返回单个表达式的值。lambda 函数不能包含命令，包含的表达式不能超过一个。不要试图向 lambda 函数中塞入太多的东西；如果你需要更复杂的东西，应该定义一个普通函数，然后想让它多长就多长。
这里只是大致介绍一下，想深入研究的可以看文章后面附的文档，这里回到开头的问题，如果返回一个字典中最大的value值的key，下面为代码：

>>> dict = {'a':1,'b':2,'c':3}
>>> max(dict.iterkeys(),key=lambda k:dict[k])
'c'
>>>

下面来大致解释这段代码，先定义了一个列表，通过使用key参数改变了max比较列表元素的方法，最终达到了取得value值最大的key的目的。

接下来讲一下，python字典的 ``iterkeys`` ``iteritems`` ``itervalues`` 这三个方法，字典对象也提供keys，items，values这三个方法，那前面的三种方法和后面的三种方法有什么不一样呢，我们大致运行一下就可以知道了，前面的三个方法返回迭代器对象，而后三种方法返回的为列表对象，使用前三种方法更高效一些，后三种方法对内存占用比较大，在python 3.0中取消了iterkeys，iteritems，itervalues这三个方法，将keys，items，values这三个方法功能改为原来iter*的功能。

**参考文档:**

* Python: `Lambda Functions`_ 
* Python: `Built-in Functions`_ 

.. _`Lambda Functions`: http://www.secnetix.de/olli/Python/lambda_functions.hawk
.. _`Built-in Functions`: http://docs.python.org/library/functions.html
