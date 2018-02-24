.. title: 如何判断一个值在lua的table里
.. slug: search_for_an_item_in_a_lua_list
.. date: 2013-05-23 20:17:59 UTC+08:00
.. tags: lua
.. category:
.. link:
.. description:
.. type: text


最近在用lua写一个nginx模块时，遇到一个需要判断一个值是否在一个table里的问题，我有一个类似如下的一个table:

.. sourcecode:: python
    
    local items = { "apple", "orange", "pear", "banana" }

我怎样去判断orange是否在这个table里，在python里我可以使用如下的方法：

.. sourcecode:: python

    if "orange" in items:
        # do something

在lua里如何实现类似的方法呢？

`Programming in Lua`_ 中提供了一种方法：

.. sourcecode:: python

    function Set (list)
      local set = {}
      for _, l in ipairs(list) do set[l] = true end
      return set
    end

你可以使用一个类似set的结构，具体使用的方法如下：

.. sourcecode:: python

    local items = Set { "apple", "orange", "pear", "banana" }

    if items["orange"] then
      -- do something
    end

上面的这个方法和下面的这个方法等同：

.. sourcecode:: python

    local items = { apple=true, orange=true, pear=true, banana=true }
    if items.apple then
        -- do something 
    end

这个方法足够简单吧，并且效率上要比遍历这个table要高。

-- eof --

.. _Programming in Lua: http://www.lua.org/pil/11.5.html
