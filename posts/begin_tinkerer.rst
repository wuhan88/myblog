.. title: 博客迁移到tinkerer
.. slug: begin_tinkerer
.. date: 2013-01-29 20:17:59 UTC+08:00
.. tags: sphinx,blog,tinkerer,staic site
.. category:
.. link:
.. description:
.. type: text


在11年时，将我的博客从wordpress+mysql换成了django+sqlite，具体可见 `将博客从wordpress迁移到django`_ 这篇文章， 在折腾精神的感召下，在12年底我将博客程序又换掉了，这次换成了一个基于 sphinx_ 静态博客生成器 tinkerer_ ， ``tinkerer`` 使用 reStructuredText_  作为标记语言，可以生成文章的分类，tag，支持评论和代码高亮，文章评论使用 ``disqus`` , 高亮代码使用 ``Pygments`` 。

说说为什么使用 ``tinkerer`` ，从打算把博客弄成静态站点开始我尝试了很多项目，如 ``octpress`` 、 ``jekyll`` 、 ``pelican`` ，一次偶然的机会在 ``bitbucket`` 上发现了 ``tinkerer`` ，试用后感觉比较顺手，并且可以使用sphinx的扩展，我比较熟悉 ``sphinx`` 和 ``reStructuredText`` 的使用，平时就用这两个东西写文档，因此选择了基于 ``sphinx`` 的 ``tinkerer`` 。我最开始使用的是0.4 beta版还存在一些bug，然后我自己做了一些修改pull给作者，作者很快就接受了我的pull request，并表示感谢，现在1.0版本已经发布，基本上没有比较明显的bug了，只是有一些细节可能不是那么完美，还有一个就是主题很少，目前默认的几款主题，都不是很美观，目前我在移植一款主题。

tinkerer的一些使用经验:

＊ 自定义博客的侧边栏

这里以创建友情链接的plugin作为例子，复制boilerplate主题目录下recent.html为一个新的文件，如friendslinks.html，然后改成如下这样就可以了: 

.. sourcecode:: html

   {#-
       boilerplate/friendslinks.html
       ~~~~~~~~~~~~~~~~~~~~~
   
       Sidebar list of all tags.
   
       :copyright: Copyright 2012 by Iñigo Serna
       :license: FreeBSD, see LICENSE file
   -#}
   
   <div class="widget">
       <h1>Friends links</h1>
       <ul>
           <li><a href="http://www.qdyongai.cn/?from=jason">龙哥-网站设计</a></li>
           <li><a href="http://www.zipeng.info/?from=jason">子鹏-kun的记事本</a></li>
           <li><a href="http://www.aaronw.me/?from=jason">王炜-我的技术生活</a></li>
           <li><a href="http://cuikai-wh.com/?from=jason">小轰-时光立方</a></li>
       </ul>
   </div>

**这里要注意的是默认的modern5主题是继承于boilerplate这个主题，因此只要修改boilerplate这个主题就可以了**

然后在conf.py中加入这个文件的配置

.. sourcecode:: python
   
   # Add templates to be rendered in sidebar here
   html_sidebars = {
       "**": ["searchbox.html", "categories.html", "recent.html", "friendslinks.html", "weibo.html"]
       }

* 创建一个makefile，使用过 ``sphinx`` 的人应该都知道 ``sphinx`` 会生成一个 makefile，这样可以直接使用make html就可以生成文档了，tinker没有提供，那么我们可以自己写一个，让我么操作的更加自动化，如下是我 ``Makefile`` 示例，大家可以参照下：

.. sourcecode:: bash

   all: build commit update
   
   clean:
   		rm -rf blog/html/
   
   build:
   		tinker -b
   
   serve:
   		cd blog/html/ && python -m SimpleHTTPServer
   
   commit:
   		hg commit ./ -m 'add new post'&&hg push
   			@echo "Done..."
   
   update:
   		ssh root@jasonwu.me 'cd /home/admin/jasonwu.me/&&hg pull&&hg update'

这个makefile主要实现了生成文档，在本地起一个http服务器来查看生成的文章效果，提交到bitbucket和更新vps上的博客内容等功能，将这些操作完全的自动化，是不是很方便，大家玩的开心。

.. _reStructuredText: http://docutils.sourceforge.net/rst.html
.. _sphinx: http://sphinx.pocoo.org/
.. _tinkerer: http://tinkerer.me/pages/documentation.html
.. _`将博客从wordpress迁移到django`: http://jasonwu.me/2011/05/29/The-blog-from-wordpress-migrate-django.html
