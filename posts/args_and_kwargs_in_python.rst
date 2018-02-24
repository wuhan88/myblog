.. title: Python : 什么是*args和**kwargs？
.. slug: args_and_kwargs_in_python
.. date: 2012-12-03 20:17:59 UTC+08:00
.. tags: python
.. category:
.. link:
.. description:
.. type: text


.. code:: python

    def foo(*args, **kwargs):
        print 'args = ', args
        print 'kwargs = ', kwargs
        print '---------------------------------------'
    if __name__ == '__main__':
        foo(1,2,3,4)
        foo(a=1,b=2,c=3)
        foo(1,2,3,4, a=1,b=2,c=3)
        foo('a', 1, None, a=1, b='2', c=3)

输出结果如下： ::

   | args = (1, 2, 3, 4) 
   | kwargs = {} 
   | --------------------------------------- 
   | args = () 
   | kwargs = {'a': 1, 'c': 3, 'b': 2} 
   | --------------------------------------- 
   | args = (1, 2, 3, 4) 
   | kwargs = {'a': 1, 'c': 3, 'b': 2} 
   | --------------------------------------- 
   | args = ('a', 1, None) 
   | kwargs = {'a': 1, 'c': 3, 'b': '2'} 
   | ---------------------------------------

可以看到，这两个是python中的可变参数。 ``*args`` 表示任何多个无名参数，它是一个tuple； ``**kwargs`` 表示关键字参数，它是一个dict。并且同时使用 ``*args`` 和 ``**kwargs`` 时，必须 ``*args`` 参数列要在 ``**kwargs`` 前，像foo(a=1, b='2', c=3, a', 1, None, )这样调用的话，会提示语法错误“SyntaxError: non-keyword arg after keyword arg”。


呵呵，知道 ``*args`` 和 ``**kwargs`` 是什么了吧。还有一个很漂亮的用法，就是创建字典：

.. code:: python

    def kw_dict(**kwargs):
        return kwargs
        print kw_dict(a=1,b=2,c=3) == {'a':1, 'b':2, 'c':3}

其实python中就带有dict类，使用dict(a=1,b=2,c=3)即可创建一个字典了。

另:连接两个字典的方法:

第一种:

>>> a={'a':'a','b':'b'}
>>> m=dict(c='c',**a)
>>> m
{'a': 'a', 'c': 'c', 'b': 'b'}
>>>

第二种:

>>> a={'a':'a','b':'b'}
>>> m={'c':'c'}
>>> m.update(a)
>>> m
{'a': 'a', 'c': 'c', 'b': 'b'}
