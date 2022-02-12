## PHP 系列 - $_SERVER

| 参数                               | 说明                                       |
| -------------------------------- | ---------------------------------------- |
| $_SERVER['PHP_SELF']             | 当前执行脚本的文件名，与 document root 有关            |
| $_SERVER['argv']                 | 传递给该脚本的参数的数组                             |
| $_SERVER['argc']                 | 包含命令行模式下传递给该脚本的参数的数目(如果运行在命令行模式下)        |
| $_SERVER[' GATEWAY_INTERFACE     | 服务器使用的 CGI 规范的版本                         |
| $_SERVER[' SERVER_ADDR']         | 当前运行脚本所在的服务器的 IP 地址                      |
| $_SERVER[' SERVER_NAME']         | 当前运行脚本所在的服务器的主机名                         |
| $_SERVER['SERVER_SOFTWARE']      | 服务器标识字符串，在响应请求时的头信息中给出                   |
| $_SERVER['SERVER_PROTOCOL']      | 请求页面时通信协议的名称和版本                          |
| $_SERVER['REQUEST_METHOD']       | 访问页面使用的请求方法。 例如，“*GET*”、“*HEAD*”、“*POST*”、“*PUT*” |
| $_SERVER['REQUEST_TIME']         | 请求开始时的时间戳                                |
| $_SERVER['REQUEST_TIME_FLOAT']   | 请求开始时的时间戳，微秒级别的精准度                       |
| $_SERVER['QUERY_STRING']         | 查询字符串，如果有的话，通过它进行页面访问                    |
| $_SERVER['DOCUMENT_ROOT']        | 当前运行脚本所在的文档根目录                           |
| $_SERVER['HTTP_ACCEPT']          | 当前请求头中 *Accept:* 项的内容                    |
| $_SERVER['HTTP_ACCEPT_CHARSET']  | 当前请求头中 *Accept-Charset:* 项的内容            |
| $_SERVER['HTTP_ACCEPT_ENCODING'] | 当前请求头中 *Accept-Encoding:* 项的内容           |
| $_SERVER['HTTP_ACCEPT_LANGUAGE'] | 当前请求头中 *Accept-Language:* 项的内容           |
| $_SERVER['HTTP_CONNECTION']      | 当前请求头中 *Connection:* 项的内容                |
| $_SERVER['HTTP_HOST']            | 当前请求头中 *Host:* 项的内容                      |
| $_SERVER['HTTP_REFERER']         | 引导用户代理到当前页的前一页的地址                        |
| $_SERVER['HTTP_USER_AGENT']      | 当前请求头中 *User-Agent:* 项的内容， 该字符串表明了访问该页面的用户代理的信息 |
| $_SERVER['HTTPS']                | 如果脚本是通过 HTTPS 协议被访问，则被设为一个非空的值           |
| $_SERVER['REMOTE_ADDR']          | 浏览当前页面的用户的 IP 地址                         |
| $_SERVER['REMOTE_HOST']          | 浏览当前页面的用户的主机名                            |
| $_SERVER['REMOTE_PORT']          | 用户机器上连接到 Web 服务器所使用的端口号                  |
| $_SERVER['REMOTE_USER']          | 经验证的用户                                   |
| $_SERVER['REDIRECT_REMOTE_USER'] | 验证的用户，如果请求已在内部重定向                        |
| $_SERVER['SCRIPT_FILENAME']      | 当前执行脚本的绝对路径                              |
| $_SERVER['SERVER_ADMIN']         | 该值指明了 Apache 服务器配置文件中的 SERVER_ADMIN 参数   |
| $_SERVER['SERVER_PORT']          | Web 服务器使用的端口                             |
| $_SERVER['SERVER_SIGNATURE']     | 包含了服务器版本和虚拟主机名的字符串                       |
| $_SERVER['PATH_TRANSLATED']      | 当前脚本所在文件系统（非文档根目录）的基本路径                  |
| $_SERVER['SCRIPT_NAME']          | 包含当前脚本的路径                                |
| $_SERVER['REQUEST_URI']          | URI 用来指定要访问的页面                           |
| $_SERVER['PHP_AUTH_DIGEST']      | 当作为 Apache 模块运行时，进行 HTTP Digest 认证的过程中，此变量被设置成客户端发送的“Authorization” HTTP 头内容 |
| $_SERVER['PHP_AUTH_USER']        | 当 PHP 运行在 Apache 或 IIS（PHP 5 是 ISAPI）模块方式下，并且正在使用 HTTP 认证功能，这个变量便是用户输入的用户名 |
| $_SERVER['PHP_AUTH_PW']          | 当 PHP 运行在 Apache 或 IIS（PHP 5 是 ISAPI）模块方式下，并且正在使用 HTTP 认证功能，这个变量便是用户输入的密码 |
| $_SERVER['AUTH_TYPE']            | 当 PHP 运行在 Apache 模块方式下，并且正在使用 HTTP 认证功能，这个变量便是认证的类型 |
| $_SERVER[' PATH_INFO']           | 包含由客户端提供的、跟在真实脚本名称之后并且在查询语句（query string）之前的路径信息 |
| $_SERVER['ORIG_PATH_INFO']       | 在被 PHP 处理之前，“PATH_INFO” 的原始版本            |

