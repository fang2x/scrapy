.. _intro-tutorial:

===============
Scrapy Tutorial
===============

In this tutorial, we'll assume that Scrapy is already installed on your system.
If that's not the case, see :ref:`intro-install`.

我们将去掉 `quotes.toscrape.com <http://quotes.toscrape.com/>`_, 这是一个列出着名作家引用的网站。

本教程将引导您完成这些任务：

1. 创建一个新的Scrapy项目
2. 编写蜘蛛抓取网站并提取数据
3. 使用命令行导出刮取的数据
4. 更改蜘蛛递归跟随链接
5. 使用蜘蛛参数

Scrapy是用Python编写的。如果您对语言很陌生，您可能想先了解语言是什么样子，以充分利用Scrapy。

如果您已经熟悉其他语言，并希望快速学习Python，我们建议您阅读Dive Into Python 3。或者，您可以按照Python教程。

如果您是编程新手，想从Python开始，那么您可能会对联机丛书“ Learn Python The Hard Way”感兴趣。您还可以查看非程序员的Python资源列表。

.. _Python: https://www.python.org/
.. _this list of Python resources for non-programmers: https://wiki.python.org/moin/BeginnersGuide/NonProgrammers
.. _Dive Into Python 3: http://www.diveintopython3.net
.. _Python Tutorial: https://docs.python.org/3/tutorial
.. _Learn Python The Hard Way: https://learnpythonthehardway.org/book/


创建一个项目
==================

在开始抓取之前，您将不得不建立一个新的Scrapy项目。输入您想要存储代码并运行的目录：

    scrapy startproject tutorial

这将创建一个 ``tutorial`` 包含以下内容的目录:

    tutorial/
        scrapy.cfg            # 部署配置文件

        tutorial/             # project的Python模块，你将从这里导入你的代码
            __init__.py

            items.py          # 项目项目定义文件
            
            middlewares.py    # 项目中间件文件

            pipelines.py      # 项目管道文件

            settings.py       # 项目设置文件

            spiders/          # 一个你将在后面放置你的蜘蛛
                __init__.py


我们的第一个蜘蛛
================

蜘蛛是您定义的类，Scrapy用来从网站（或一组网站）上刮取信息。他们必须进行子类化 scrapy.Spider和定义最初的请求，可选择如何关注页面中的链接，以及如何解析下载的页面内容以提取数据.

这是我们第一个蜘蛛的代码。将其保存在项目目录quotes_spider.py下的一个文件 tutorial/spiders中：

    import scrapy


    class QuotesSpider(scrapy.Spider):
        name = "quotes"

        def start_requests(self):
            urls = [
                'http://quotes.toscrape.com/page/1/',
                'http://quotes.toscrape.com/page/2/',
            ]
            for url in urls:
                yield scrapy.Request(url=url, callback=self.parse)

        def parse(self, response):
            page = response.url.split("/")[-2]
            filename = 'quotes-%s.html' % page
            with open(filename, 'wb') as f:
                f.write(response.body)
            self.log('Saved file %s' % filename)


正如您所看到的，我们的Spider子类scrapy.Spider 定义了一些属性和方法：

* name：标识蜘蛛。它在项目中必须是唯一的，也就是说，不能为不同的蜘蛛设置相同的名称。

* start_requests()：必须返回Spider将开始抓取的请求的迭代（您可以返回请求列表或编写生成器函数）。随后的请求将从这些初始请求中连续生成。

* parse()：将被调用来处理为每个请求下载的响应的方法。响应参数是TextResponse保存页面内容的一个实例，并有更多有用的方法来处理它。

  该parse()方法通常解析响应，将提取的数据提取为字符串，并查找新的URL并Request根据它们创建新的请求（）。

如何运行我们的蜘蛛
---------------------

为了让我们的蜘蛛工作，请转到项目的顶层目录并运行：

   scrapy crawl quotes

这个命令以quotes我们刚刚添加的名称运行蜘蛛，它将发送一些quotes.toscrape.com域的请求。你会得到一个类似于这样的输出：

    ... (omitted for brevity)
    2016-12-16 21:24:05 [scrapy.core.engine] INFO: Spider opened
    2016-12-16 21:24:05 [scrapy.extensions.logstats] INFO: Crawled 0 pages (at 0 pages/min), scraped 0 items (at 0 items/min)
    2016-12-16 21:24:05 [scrapy.extensions.telnet] DEBUG: Telnet console listening on 127.0.0.1:6023
    2016-12-16 21:24:05 [scrapy.core.engine] DEBUG: Crawled (404) <GET http://quotes.toscrape.com/robots.txt> (referer: None)
    2016-12-16 21:24:05 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://quotes.toscrape.com/page/1/> (referer: None)
    2016-12-16 21:24:05 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://quotes.toscrape.com/page/2/> (referer: None)
    2016-12-16 21:24:05 [quotes] DEBUG: Saved file quotes-1.html
    2016-12-16 21:24:05 [quotes] DEBUG: Saved file quotes-2.html
    2016-12-16 21:24:05 [scrapy.core.engine] INFO: Closing spider (finished)
    ...

现在，检查当前目录中的文件。按照我们的方法指示，您应该注意到已经创建了两个新文件：quotes-1.html和quotes-2.html，以及各个URL的内容parse。



注意

如果您想知道为什么我们还没有解析HTML，请坚持下去，我们会很快回覆。


在引擎盖下发生了什么？
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Scrapy安排scrapy.Request由start_requestsSpider 的方法返回的对象。收到每个响应后，它会实例化Response对象并调用与请求相关的回调方法（在本例中为 parse方法），将响应作为参数传递。


start_requests方法的快捷方式
---------------------------------------
不用实现一个从URL start_requests()生成scrapy.Request对象的方法，你可以start_urls用一个URL列表来定义一个类属性。这个列表将被默认实现start_requests()用于为您的蜘蛛创建初始请求

    import scrapy


    class QuotesSpider(scrapy.Spider):
        name = "quotes"
        start_urls = [
            'http://quotes.toscrape.com/page/1/',
            'http://quotes.toscrape.com/page/2/',
        ]

        def parse(self, response):
            page = response.url.split("/")[-2]
            filename = 'quotes-%s.html' % page
            with open(filename, 'wb') as f:
                f.write(response.body)

该parse()方法将被调用来处理这些URL的每个请求，即使我们没有明确告诉Scrapy这样做。发生这种情况是因为parse()Scrapy的默认回调方法，该方法在没有显式分配回调的情况下被调用。


提取数据
---------------

学习如何使用Scrapy提取数据的最佳方法是尝试使用Shell Scrapy外壳的选择器。跑：

    scrapy shell 'http://quotes.toscrape.com/page/1/'

注意

请记住，从命令行运行Scrapy shell时，请始终将引号括起来，否则包含参数（例如。&字符）的url 将不起作用。

在Windows上，请使用双引号：

       scrapy shell "http://quotes.toscrape.com/page/1/"

你会看到类似于：

    [ ... Scrapy log here ... ]
    2016-09-19 12:09:27 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://quotes.toscrape.com/page/1/> (referer: None)
    [s] Available Scrapy objects:
    [s]   scrapy     scrapy module (contains scrapy.Request, scrapy.Selector, etc)
    [s]   crawler    <scrapy.crawler.Crawler object at 0x7fa91d888c90>
    [s]   item       {}
    [s]   request    <GET http://quotes.toscrape.com/page/1/>
    [s]   response   <200 http://quotes.toscrape.com/page/1/>
    [s]   settings   <scrapy.settings.Settings object at 0x7fa91d888c10>
    [s]   spider     <DefaultSpider 'default' at 0x7fa91c8af990>
    [s] Useful shortcuts:
    [s]   shelp()           Shell help (print this help)
    [s]   fetch(req_or_url) Fetch request (or URL) and update local objects
    [s]   view(response)    View response in a browser
    >>>

使用shell，你可以尝试用响应对象使用CSS选择元素：

    >>> response.css('title')
    [<Selector xpath='descendant-or-self::title' data='<title>Quotes to Scrape</title>'>]

运行的结果response.css('title')是一个名为的类似列表的对象 SelectorList，它表示一系列Selector围绕XML / HTML元素的对象列表， 并允许您运行更多查询来细化选择或提取数据。

要从上述标题中提取文本，您可以执行以下操作：

    >>> response.css('title::text').extract()
    ['Quotes to Scrape']

这里需要注意两点：其一是我们已经添加::text到CSS查询中，意思是我们只想选择元素内部的文本元素 <title>。如果我们没有指定::text，我们会得到完整的标题元素，包括它的标签：

    >>> response.css('title').extract()
    ['<title>Quotes to Scrape</title>']

另一件事是调用的结果.extract()是一个列表，因为我们正在处理一个实例SelectorList。当你知道你只是想要第一个结果，就像在这种情况下，你可以这样做：

    >>> response.css('title::text').extract_first()
    'Quotes to Scrape'

或者，你可以写下：

    >>> response.css('title::text')[0].extract()
    'Quotes to Scrape'

但是，如果找不到与选择相匹配的元素，则使用.extract_first()避免IndexError并返回 None。

这里有一个教训：对于大多数刮擦代码，您希望它能够灵活地处理由于在页面上未找到的东西而导致的错误，因此即使某些部分无法被刮取，您至少也可以获取一些数据。

除了extract()和 extract_first()方法之外，还可以使用该re()方法使用正则表达式进行提取：

    >>> response.css('title::text').re(r'Quotes.*')
    ['Quotes to Scrape']
    >>> response.css('title::text').re(r'Q\w+')
    ['Quotes']
    >>> response.css('title::text').re(r'(\w+) to (\w+)')
    ['Quotes', 'Scrape']

为了找到合适的CSS选择器来使用，你可能会发现在你的web浏览器中用shell打开响应页面很有用view(response)。您可以使用浏览器开发工具或Firebug等扩展（请参阅关于使用Firebug进行抓取和使用Firefox进行抓取的部分）。

Selector Gadget也是一个很好的工具，可以快速找到可供选择的元素的CSS选择器，这可以在许多浏览器中使用。

.. _regular expressions: https://docs.python.org/3/library/re.html
.. _Selector Gadget: http://selectorgadget.com/


XPath: a brief intro
^^^^^^^^^^^^^^^^^^^^

Besides `CSS`_, Scrapy selectors also support using `XPath`_ expressions::

    >>> response.xpath('//title')
    [<Selector xpath='//title' data='<title>Quotes to Scrape</title>'>]
    >>> response.xpath('//title/text()').extract_first()
    'Quotes to Scrape'

XPath expressions are very powerful, and are the foundation of Scrapy
Selectors. In fact, CSS selectors are converted to XPath under-the-hood. You
can see that if you read closely the text representation of the selector
objects in the shell.

While perhaps not as popular as CSS selectors, XPath expressions offer more
power because besides navigating the structure, it can also look at the
content. Using XPath, you're able to select things like: *select the link
that contains the text "Next Page"*. This makes XPath very fitting to the task
of scraping, and we encourage you to learn XPath even if you already know how to
construct CSS selectors, it will make scraping much easier.

We won't cover much of XPath here, but you can read more about :ref:`using XPath
with Scrapy Selectors here <topics-selectors>`. To learn more about XPath, we
recommend `this tutorial to learn XPath through examples
<http://zvon.org/comp/r/tut-XPath_1.html>`_, and `this tutorial to learn "how
to think in XPath" <http://plasmasturm.org/log/xpath101/>`_.

.. _XPath: https://www.w3.org/TR/xpath
.. _CSS: https://www.w3.org/TR/selectors

Extracting quotes and authors
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Now that you know a bit about selection and extraction, let's complete our
spider by writing the code to extract the quotes from the web page.

Each quote in http://quotes.toscrape.com is represented by HTML elements that look
like this:

.. code-block:: html

    <div class="quote">
        <span class="text">“The world as we have created it is a process of our
        thinking. It cannot be changed without changing our thinking.”</span>
        <span>
            by <small class="author">Albert Einstein</small>
            <a href="/author/Albert-Einstein">(about)</a>
        </span>
        <div class="tags">
            Tags:
            <a class="tag" href="/tag/change/page/1/">change</a>
            <a class="tag" href="/tag/deep-thoughts/page/1/">deep-thoughts</a>
            <a class="tag" href="/tag/thinking/page/1/">thinking</a>
            <a class="tag" href="/tag/world/page/1/">world</a>
        </div>
    </div>

Let's open up scrapy shell and play a bit to find out how to extract the data
we want::

    $ scrapy shell 'http://quotes.toscrape.com'

We get a list of selectors for the quote HTML elements with::

    >>> response.css("div.quote")

Each of the selectors returned by the query above allows us to run further
queries over their sub-elements. Let's assign the first selector to a
variable, so that we can run our CSS selectors directly on a particular quote::

    >>> quote = response.css("div.quote")[0]

Now, let's extract ``title``, ``author`` and the ``tags`` from that quote
using the ``quote`` object we just created::

    >>> title = quote.css("span.text::text").extract_first()
    >>> title
    '“The world as we have created it is a process of our thinking. It cannot be changed without changing our thinking.”'
    >>> author = quote.css("small.author::text").extract_first()
    >>> author
    'Albert Einstein'

Given that the tags are a list of strings, we can use the ``.extract()`` method
to get all of them::

    >>> tags = quote.css("div.tags a.tag::text").extract()
    >>> tags
    ['change', 'deep-thoughts', 'thinking', 'world']

Having figured out how to extract each bit, we can now iterate over all the
quotes elements and put them together into a Python dictionary::

    >>> for quote in response.css("div.quote"):
    ...     text = quote.css("span.text::text").extract_first()
    ...     author = quote.css("small.author::text").extract_first()
    ...     tags = quote.css("div.tags a.tag::text").extract()
    ...     print(dict(text=text, author=author, tags=tags))
    {'tags': ['change', 'deep-thoughts', 'thinking', 'world'], 'author': 'Albert Einstein', 'text': '“The world as we have created it is a process of our thinking. It cannot be changed without changing our thinking.”'}
    {'tags': ['abilities', 'choices'], 'author': 'J.K. Rowling', 'text': '“It is our choices, Harry, that show what we truly are, far more than our abilities.”'}
        ... a few more of these, omitted for brevity
    >>>

Extracting data in our spider
-----------------------------

Let's get back to our spider. Until now, it doesn't extract any data in
particular, just saves the whole HTML page to a local file. Let's integrate the
extraction logic above into our spider.

A Scrapy spider typically generates many dictionaries containing the data
extracted from the page. To do that, we use the ``yield`` Python keyword
in the callback, as you can see below::

    import scrapy


    class QuotesSpider(scrapy.Spider):
        name = "quotes"
        start_urls = [
            'http://quotes.toscrape.com/page/1/',
            'http://quotes.toscrape.com/page/2/',
        ]

        def parse(self, response):
            for quote in response.css('div.quote'):
                yield {
                    'text': quote.css('span.text::text').extract_first(),
                    'author': quote.css('small.author::text').extract_first(),
                    'tags': quote.css('div.tags a.tag::text').extract(),
                }

If you run this spider, it will output the extracted data with the log::

    2016-09-19 18:57:19 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/page/1/>
    {'tags': ['life', 'love'], 'author': 'André Gide', 'text': '“It is better to be hated for what you are than to be loved for what you are not.”'}
    2016-09-19 18:57:19 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/page/1/>
    {'tags': ['edison', 'failure', 'inspirational', 'paraphrased'], 'author': 'Thomas A. Edison', 'text': "“I have not failed. I've just found 10,000 ways that won't work.”"}


.. _storing-data:

Storing the scraped data
========================

The simplest way to store the scraped data is by using :ref:`Feed exports
<topics-feed-exports>`, with the following command::

    scrapy crawl quotes -o quotes.json

That will generate an ``quotes.json`` file containing all scraped items,
serialized in `JSON`_.

For historic reasons, Scrapy appends to a given file instead of overwriting
its contents. If you run this command twice without removing the file
before the second time, you'll end up with a broken JSON file.

You can also use other formats, like `JSON Lines`_::

    scrapy crawl quotes -o quotes.jl

The `JSON Lines`_ format is useful because it's stream-like, you can easily
append new records to it. It doesn't have the same problem of JSON when you run
twice. Also, as each record is a separate line, you can process big files
without having to fit everything in memory, there are tools like `JQ`_ to help
doing that at the command-line.

In small projects (like the one in this tutorial), that should be enough.
However, if you want to perform more complex things with the scraped items, you
can write an :ref:`Item Pipeline <topics-item-pipeline>`. A placeholder file
for Item Pipelines has been set up for you when the project is created, in
``tutorial/pipelines.py``. Though you don't need to implement any item
pipelines if you just want to store the scraped items.

.. _JSON Lines: http://jsonlines.org
.. _JQ: https://stedolan.github.io/jq


Following links
===============

Let's say, instead of just scraping the stuff from the first two pages
from http://quotes.toscrape.com, you want quotes from all the pages in the website.

Now that you know how to extract data from pages, let's see how to follow links
from them.

First thing is to extract the link to the page we want to follow.  Examining
our page, we can see there is a link to the next page with the following
markup:

.. code-block:: html

    <ul class="pager">
        <li class="next">
            <a href="/page/2/">Next <span aria-hidden="true">&rarr;</span></a>
        </li>
    </ul>

We can try extracting it in the shell::

    >>> response.css('li.next a').extract_first()
    '<a href="/page/2/">Next <span aria-hidden="true">→</span></a>'

This gets the anchor element, but we want the attribute ``href``. For that,
Scrapy supports a CSS extension that let's you select the attribute contents,
like this::

    >>> response.css('li.next a::attr(href)').extract_first()
    '/page/2/'

Let's see now our spider modified to recursively follow the link to the next
page, extracting data from it::

    import scrapy


    class QuotesSpider(scrapy.Spider):
        name = "quotes"
        start_urls = [
            'http://quotes.toscrape.com/page/1/',
        ]

        def parse(self, response):
            for quote in response.css('div.quote'):
                yield {
                    'text': quote.css('span.text::text').extract_first(),
                    'author': quote.css('small.author::text').extract_first(),
                    'tags': quote.css('div.tags a.tag::text').extract(),
                }

            next_page = response.css('li.next a::attr(href)').extract_first()
            if next_page is not None:
                next_page = response.urljoin(next_page)
                yield scrapy.Request(next_page, callback=self.parse)


Now, after extracting the data, the ``parse()`` method looks for the link to
the next page, builds a full absolute URL using the
:meth:`~scrapy.http.Response.urljoin` method (since the links can be
relative) and yields a new request to the next page, registering itself as
callback to handle the data extraction for the next page and to keep the
crawling going through all the pages.

What you see here is Scrapy's mechanism of following links: when you yield
a Request in a callback method, Scrapy will schedule that request to be sent
and register a callback method to be executed when that request finishes.

Using this, you can build complex crawlers that follow links according to rules
you define, and extract different kinds of data depending on the page it's
visiting.

In our example, it creates a sort of loop, following all the links to the next page
until it doesn't find one -- handy for crawling blogs, forums and other sites with
pagination.


.. _response-follow-example:

A shortcut for creating Requests
--------------------------------

As a shortcut for creating Request objects you can use
:meth:`response.follow <scrapy.http.TextResponse.follow>`::

    import scrapy


    class QuotesSpider(scrapy.Spider):
        name = "quotes"
        start_urls = [
            'http://quotes.toscrape.com/page/1/',
        ]

        def parse(self, response):
            for quote in response.css('div.quote'):
                yield {
                    'text': quote.css('span.text::text').extract_first(),
                    'author': quote.css('span small::text').extract_first(),
                    'tags': quote.css('div.tags a.tag::text').extract(),
                }

            next_page = response.css('li.next a::attr(href)').extract_first()
            if next_page is not None:
                yield response.follow(next_page, callback=self.parse)

Unlike scrapy.Request, ``response.follow`` supports relative URLs directly - no
need to call urljoin. Note that ``response.follow`` just returns a Request
instance; you still have to yield this Request.

You can also pass a selector to ``response.follow`` instead of a string;
this selector should extract necessary attributes::

    for href in response.css('li.next a::attr(href)'):
        yield response.follow(href, callback=self.parse)

For ``<a>`` elements there is a shortcut: ``response.follow`` uses their href
attribute automatically. So the code can be shortened further::

    for a in response.css('li.next a'):
        yield response.follow(a, callback=self.parse)

.. note::

    ``response.follow(response.css('li.next a'))`` is not valid because
    ``response.css`` returns a list-like object with selectors for all results,
    not a single selector. A ``for`` loop like in the example above, or
    ``response.follow(response.css('li.next a')[0])`` is fine.

More examples and patterns
--------------------------

Here is another spider that illustrates callbacks and following links,
this time for scraping author information::

    import scrapy


    class AuthorSpider(scrapy.Spider):
        name = 'author'

        start_urls = ['http://quotes.toscrape.com/']

        def parse(self, response):
            # follow links to author pages
            for href in response.css('.author + a::attr(href)'):
                yield response.follow(href, self.parse_author)

            # follow pagination links
            for href in response.css('li.next a::attr(href)'):
                yield response.follow(href, self.parse)

        def parse_author(self, response):
            def extract_with_css(query):
                return response.css(query).extract_first().strip()

            yield {
                'name': extract_with_css('h3.author-title::text'),
                'birthdate': extract_with_css('.author-born-date::text'),
                'bio': extract_with_css('.author-description::text'),
            }

This spider will start from the main page, it will follow all the links to the
authors pages calling the ``parse_author`` callback for each of them, and also
the pagination links with the ``parse`` callback as we saw before.

Here we're passing callbacks to ``response.follow`` as positional arguments
to make the code shorter; it also works for ``scrapy.Request``.

The ``parse_author`` callback defines a helper function to extract and cleanup the
data from a CSS query and yields the Python dict with the author data.

Another interesting thing this spider demonstrates is that, even if there are
many quotes from the same author, we don't need to worry about visiting the
same author page multiple times. By default, Scrapy filters out duplicated
requests to URLs already visited, avoiding the problem of hitting servers too
much because of a programming mistake. This can be configured by the setting
:setting:`DUPEFILTER_CLASS`.

Hopefully by now you have a good understanding of how to use the mechanism
of following links and callbacks with Scrapy.

As yet another example spider that leverages the mechanism of following links,
check out the :class:`~scrapy.spiders.CrawlSpider` class for a generic
spider that implements a small rules engine that you can use to write your
crawlers on top of it.

Also, a common pattern is to build an item with data from more than one page,
using a :ref:`trick to pass additional data to the callbacks
<topics-request-response-ref-request-callback-arguments>`.


Using spider arguments
======================

You can provide command line arguments to your spiders by using the ``-a``
option when running them::

    scrapy crawl quotes -o quotes-humor.json -a tag=humor

These arguments are passed to the Spider's ``__init__`` method and become
spider attributes by default.

In this example, the value provided for the ``tag`` argument will be available
via ``self.tag``. You can use this to make your spider fetch only quotes
with a specific tag, building the URL based on the argument::

    import scrapy


    class QuotesSpider(scrapy.Spider):
        name = "quotes"

        def start_requests(self):
            url = 'http://quotes.toscrape.com/'
            tag = getattr(self, 'tag', None)
            if tag is not None:
                url = url + 'tag/' + tag
            yield scrapy.Request(url, self.parse)

        def parse(self, response):
            for quote in response.css('div.quote'):
                yield {
                    'text': quote.css('span.text::text').extract_first(),
                    'author': quote.css('small.author::text').extract_first(),
                }

            next_page = response.css('li.next a::attr(href)').extract_first()
            if next_page is not None:
                yield response.follow(next_page, self.parse)


If you pass the ``tag=humor`` argument to this spider, you'll notice that it
will only visit URLs from the ``humor`` tag, such as
``http://quotes.toscrape.com/tag/humor``.

You can :ref:`learn more about handling spider arguments here <spiderargs>`.

Next steps
==========

This tutorial covered only the basics of Scrapy, but there's a lot of other
features not mentioned here. Check the :ref:`topics-whatelse` section in
:ref:`intro-overview` chapter for a quick overview of the most important ones.

You can continue from the section :ref:`section-basics` to know more about the
command-line tool, spiders, selectors and other things the tutorial hasn't covered like
modeling the scraped data. If you prefer to play with an example project, check
the :ref:`intro-examples` section.

.. _JSON: https://en.wikipedia.org/wiki/JSON
.. _dirbot: https://github.com/scrapy/dirbot
