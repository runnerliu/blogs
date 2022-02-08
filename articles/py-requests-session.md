## Python 库 - requests 会话对象

### 问题背景

一个简单的需求是，向一个 host 地址+端口定时 post 一定量数据，程序使用 [requests](http://www.python-requests.org/en/master/) 的 post 方式，傻瓜式同步执行，因为没有时间限制。但是在执行后出现：

```
TimeoutError: [Errno 110] Connection timed out
```

显示连接超时了，程序每次运行 post 了大概500条数据，而这个错误只会出现一次，猜测可能是 HTTP 短连接的问题。

### 解决方案

其实对于	 向同一主机发送多个请求 这种场景，使用 HTTP 的长连接效率会更高，复用底层的 TCP 连接。Requests 库同样提供了这样的方式，即会话对象 - [Session](http://www.python-requests.org/en/master/user/advanced/#session-objects) ，使用方式如下：

```
s = requests.Session()
r = s.get('http://www.baidu.com')
print(r.text)
```

[源码](http://docs.python-requests.org/en/master/_modules/requests/adapters/?highlight=HTTPAdapter)如下：

```
class Session(SessionRedirectMixin):
 
    def __init__(self):
        ...
        self.max_redirects = DEFAULT_REDIRECT_LIMIT
        self.cookies = cookiejar_from_dict({})
        self.adapters = OrderedDict()
        self.mount('https://', HTTPAdapter())
        self.mount('http://', HTTPAdapter())
        self.redirect_cache = RecentlyUsedContainer(REDIRECT_CACHE_SIZE)
 
class HTTPAdapter(BaseAdapter):
 
    def __init__(self, pool_connections=DEFAULT_POOLSIZE,
                 pool_maxsize=DEFAULT_POOLSIZE, max_retries=DEFAULT_RETRIES,
                 pool_block=DEFAULT_POOLBLOCK):
        if max_retries == DEFAULT_RETRIES:
            self.max_retries = Retry(0, read=False)
        else:
            self.max_retries = Retry.from_int(max_retries)
        self.config = {}
        self.proxy_manager = {}
 
        super(HTTPAdapter, self).__init__()
 
        self._pool_connections = pool_connections
        self._pool_maxsize = pool_maxsize
        self._pool_block = pool_block
        self.init_poolmanager(pool_connections, pool_maxsize, block=pool_block) 
 
DEFAULT_POOLBLOCK = False  #是否阻塞连接池
DEFAULT_POOLSIZE = 10  # 默认连接池
DEFAULT_RETRIES = 0   # 默认重试次数
DEFAULT_POOL_TIMEOUT = None  # 超时时间
```

默认情况下，Session 连接池大小为10，请求失败重试次数为0，但是一般情况下我们会需要稍大容量的连接池，从源码中可以看到，session 绑定了 `HTTPAdapter` 对象，所以我们可以自定义一个符合预期参数的 `HTTPAdapter` 对象。

```
def get_http_session(self, pool_connections, pool_maxsize, max_retries):
    session = requests.Session()
    # 创建一个适配器，连接池的数量pool_connections, 最大数量pool_maxsize, 失败重试的次数max_retries
    adapter = requests.adapters.HTTPAdapter(pool_connections = pool_connections,
            pool_maxsize = pool_maxsize, max_retries = max_retries)
    # 告诉requests，http协议和https协议都使用这个适配器
    session.mount('http://', adapter)
    session.mount('https://', adapter)
    return session
```

然后就可以使用 session 的各个功能了。So Easy!



Read More:

> [Advanced Usage](http://www.python-requests.org/en/master/user/advanced/#session-objects) 