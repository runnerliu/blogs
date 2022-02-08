## Python 系列 - 上下文管理器

### 何为上下文管理器

上下文管理器是管理上下文的，负责冲锋和垫后，而让开发人员专心完成自己的事情。我们在编写程序的时候，通常会将一系列操作放到一个语句块中，当某一条件为真时执行该语句快。有时候，我们需要再执行一个语句块时保持某种状态，并且在离开语句块后结束这种状态。

例如对文件的操作，我们在打开一个文件进行读写操作时需要保持文件处于打开状态，而等操作完成之后要将文件关闭。所以，上下文管理器的任务是：代码块执行前准备，代码块执行后收拾。上下文管理器是在Python2.5加入的功能，它能够让你的代码可读性更强并且错误更少。

### 需求的产生

在正常的管理各种系统资源（文件、锁定和连接），在涉及到异常时通常是个棘手的问题。异常很可能导致控制流跳过负责释放关键资源的语句。例如打开一个文件进行操作时，如果意外情况发生（磁盘已满、特殊的终端信号让其终止等），就会抛出异常，这样可能最后的文件关闭操作就不会执行。如果这样的问题频繁出现，则可能耗尽系统资源。

在没有接触到上下文管理器之前，我们可以用"try/finally"语句来解决这样的问题。或许在有些人看来，"try/finally"语句显得有些繁琐。上下文管理器就是被设计用来简化"try/finally"语句的，这样可以让程序更加简洁。

### 上下文管理协议

那么在Python中怎么实现一个上下文管理器呢？主要依靠`__enter__`、`__exit__`这两个"魔术方法"。

> `__enter__(self)` 
>
> Defines what the context manager should do at the beginning of the block created by the with statement. Note that the return value of _\_enter__ is bound to the target of the with statement, or the name after the as.
>
> `__exit__(self, exception_type, exception_value, traceback)` 
>
> Defines what the context manager should do after its block has been executed (or terminates). It can be used to handle exceptions, perform cleanup, or do something always done immediately after the action in the block. If the block executes successfully, exception_type, exception_value, and traceback will be None. Otherwise, you can choose to handle the exception or let the user handle it; if you want to handle it, make sure _\_exit__ returns True after all is said and done. If you don't want the exception to be handled by the context manager, just let it happen.

也就是说，当我们需要创建一个上下文管理器类型的时候，就需要实现`__enter__`和`__exit__`方法，这对方法就称为上下文管理协议（Context Manager Protocol），定义了一种运行时上下文环境。

#### with语句

在Python中，可以通过with语句来方便的使用上下文管理器，with语句可以在代码块运行前进入一个运行时上下文（执行\_\_enter\_\_方法），并在代码块结束后退出该上下文（执行\_\_exit\_\_方法）。

with语句的语法如下：

```
with context_expr [as var]:
    with_suite
```

- context_expr: 支持上下文管理协议的对象，也就是上下文管理器对象，负责维护上下文环境
- as var: 可选部分，通过变量方式保存上下文管理器对象
- with_suite: 需要放在上下文环境中执行的语句块

在Python的内置类型中，很多类型都是支持上下文管理协议的，例如file、thread.LockType、threading.Lock等等。这里我们就以file类型为例，看看with语句的使用。

```
with File('demo.txt', 'w') as opened_file:
    opened_file.write('Hola!')
```

 with语句先暂存了File类的`__exit__()`方法，然后它调用File类的`__enter__()`方方法，`__enter__`方法打开文件并返回给with语句，打开的文件句柄被传递给`opened_file`参数，然后使用`.write()`来写文件，with语句调用之前暂存的`__exit__`方法关闭了文件。

#### 自定义上下文管理器

了解上下文管理器的执行流程和方法后，我们可以通过实现`__enter__()`、`__exit__()`函数来自定义上下文管理器，如下：

```
filename = 'my_file.txt'
mode = 'w' # Mode that allows to write to the file
writer = open(filename, mode)

class PypixOpen(object):
    def __init__(self, filename, mode):
        self.filename = filename
        self.mode = mode

    def __enter__(self):
        self.openedFile = open(self.filename, self.mode)
        return self.openedFile

    def __exit__(self, *unused):
        self.openedFile.close()

# Script starts from here

with PypixOpen(filename, mode) as writer:
    writer.write("Hello World from our new Context Manager!")
```

#### 异常处理

上下文管理器根据`__exit__()` 方法的返回值来决定是否抛出异常，如果没有返回值或者返回值为 False ，则异常由上下文管理器处理，如果为 True 则由用户自己处理。

`__exit__ ()`接受三个参数，exception_type、exception_value、traceback，我们可以根据这些值来决定是否处理异常

下面这个例子，捕捉了 AttributeError 的异常，并打印出警告: 

```
class Open:
    def __init__(self, file, mode):
        self.open_file = open(file, mode)
    def __enter__(self):
        return self.open_file
    def __exit__(self, type, value, tb):
        self.open_file.close()
        if type is AttributeError:
            print('handing some exception')
            return True
            
with Open('aaa', 'w') as f:
    f.writeee('aaaa')
    
    
>>> handing some exception
```

#### contextmanager

由于上下文管理非常有用，Python 中有一个专门用于实现上下文管理的标准库，这就是 contextlib。

有了 contextlib 创建上下文管理的最好方式就是使用 contextmanager 装饰器，通过 contextmanager 装饰一个生成器函数，`yield` 语句前面的部分被认为是` __enter__()` 方法的代码，后面的部分被认为是 `__exit__() `方法的代码。

```
from contextlib import contextmanager
@contextmanager
def file(path, mode):
    open_file = open(path, mode)
    yield open_file
    open_file.close()
```

Python解释器遇到了`yield`关键字。因为这个缘故它创建了一个生成器而不是一个普通的函数。因为这个装饰器，contextmanager会被调用并传入函数名（`file`）作为参数。contextmanager函数返回一个以GeneratorContextManager对象封装过的生成器。这个GeneratorContextManager被赋值给`file`函数，我们实际上是在调用GeneratorContextManager对象。



Read More: 

> [上下文管理器(Context managers)](https://eastlakeside.gitbooks.io/interpy-zh/content/context_managers/) 
>
> [Python上下文管理器与with语句](http://kuanghy.github.io/2015/08/08/python-with) 
>
> [Python上下文管理器](http://www.cnblogs.com/wilber2013/p/4638967.html) 
>
> [Python - 上下文管理](https://anyisalin.github.io/anyisalin.github.io/2017/03/07/python-context-manager/) 