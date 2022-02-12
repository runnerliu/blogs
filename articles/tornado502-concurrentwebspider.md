## Tornado5.0.2 翻译文档 - Queue示例:一个并发网络爬虫

Tornado的 [`tornado.queues`](http://tornado-zh.readthedocs.io/zh/latest/queues.html#module-tornado.queues) 模块实现了异步生产者/消费者模式的协程，类似于通过Python 标准库的 [`queue`](https://docs.python.org/3.4/library/queue.html#module-queue) 实现线程模式。

一个yields [`Queue.get`](http://tornado-zh.readthedocs.io/zh/latest/queues.html#tornado.queues.Queue.get) 的协程直到队列中有值的时候才会暂停。如果队列设置了最大长度，yields [`Queue.put`](http://tornado-zh.readthedocs.io/zh/latest/queues.html#tornado.queues.Queue.put) 的协程直到队列中有空间才会暂停。

一个 [`Queue`](http://tornado-zh.readthedocs.io/zh/latest/queues.html#tornado.queues.Queue) 从0开始对未完成的任务进行计数， [`put`](http://tornado-zh.readthedocs.io/zh/latest/queues.html#tornado.queues.Queue.put) 加计数； [`task_done`](http://tornado-zh.readthedocs.io/zh/latest/queues.html#tornado.queues.Queue.task_done) 减少计数。

这里的网络爬虫的例子，队列开始的时候只包含 base_url。当一个 worker 抓取到一个页面它会解析链接并把它添加到队列中，然后调用 [`task_done`](http://tornado-zh.readthedocs.io/zh/latest/queues.html#tornado.queues.Queue.task_done) 减少计数一次。最后，当一个 worker 抓取到的页面 URL 都是之前抓取到过的并且队列中没有任务了。于是 worker 调用 [`task_done`](http://tornado-zh.readthedocs.io/zh/latest/queues.html#tornado.queues.Queue.task_done) 把计数减到0。等待 [`join`](http://tornado-zh.readthedocs.io/zh/latest/queues.html#tornado.queues.Queue.join) 的主协程取消暂停并且完成。

```
#!/usr/bin/env python

import time
from datetime import timedelta

try:
    from HTMLParser import HTMLParser
    from urlparse import urljoin, urldefrag
except ImportError:
    from html.parser import HTMLParser
    from urllib.parse import urljoin, urldefrag

from tornado import httpclient, gen, ioloop, queues

base_url = 'http://www.tornadoweb.org/en/stable/'
concurrency = 10


@gen.coroutine
def get_links_from_url(url):
    """Download the page at `url` and parse it for links.

    Returned links have had the fragment after `#` removed, and have been made
    absolute so, e.g. the URL 'gen.html#tornado.gen.coroutine' becomes
    'http://www.tornadoweb.org/en/stable/gen.html'.
    """
    try:
        response = yield httpclient.AsyncHTTPClient().fetch(url)
        print('fetched %s' % url)

        html = response.body if isinstance(response.body, str) \
            else response.body.decode(errors='ignore')
        urls = [urljoin(url, remove_fragment(new_url))
                for new_url in get_links(html)]
    except Exception as e:
        print('Exception: %s %s' % (e, url))
        raise gen.Return([])

    raise gen.Return(urls)


def remove_fragment(url):
    pure_url, frag = urldefrag(url)
    return pure_url


def get_links(html):
    class URLSeeker(HTMLParser):
        def __init__(self):
            HTMLParser.__init__(self)
            self.urls = []

        def handle_starttag(self, tag, attrs):
            href = dict(attrs).get('href')
            if href and tag == 'a':
                self.urls.append(href)

    url_seeker = URLSeeker()
    url_seeker.feed(html)
    return url_seeker.urls


@gen.coroutine
def main():
    q = queues.Queue()
    start = time.time()
    fetching, fetched = set(), set()

    @gen.coroutine
    def fetch_url():
        current_url = yield q.get()
        try:
            if current_url in fetching:
                return

            print('fetching %s' % current_url)
            fetching.add(current_url)
            urls = yield get_links_from_url(current_url)
            fetched.add(current_url)

            for new_url in urls:
                # Only follow links beneath the base URL
                if new_url.startswith(base_url):
                    yield q.put(new_url)

        finally:
            q.task_done()

    @gen.coroutine
    def worker():
        while True:
            yield fetch_url()

    q.put(base_url)

    # Start workers, then wait for the work queue to be empty.
    for _ in range(concurrency):
        worker()
    yield q.join(timeout=timedelta(seconds=300))
    assert fetching == fetched
    print('Done in %d seconds, fetched %s URLs.' % (
        time.time() - start, len(fetched)))


if __name__ == '__main__':
    io_loop = ioloop.IOLoop.current()
    io_loop.run_sync(main)
```



Read More:

> [`Queue` example - a concurrent web spider](http://www.tornadoweb.org/en/stable/guide/queues.html) 





