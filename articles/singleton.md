## 单例模式

单例模式属于创建型模式，它提供了一种创建对象的最佳方式。这种模式涉及到一个单一的类，该类负责创建自己的对象，同时确保只有单个对象被创建。这个类提供了一种访问其唯一的对象的方式，可以直接访问，不需要实例化该类的对象。
特点：

- 单例类只能有一个实例；
- 单例类必须自己创建自己的唯一实例；
- 单例类必须给所有其他对象提供这一实例。

使用场景：

- Python 的 logger 就是一个单例模式，用以日志记录；
- Windows 的资源管理器是一个单例模式；
- 线程池，数据库连接池等资源池一般也用单例模式；
- 网站计数器。

优点：

- 在内存中只有一个对象，节省内存空间；
- 避免频繁地创建销毁对象，可以提高性能；
- 避免对共享资源的多重占用；
- 可以全局访问。

### 非线程安全

维护类中的共享变量的方式：

```
class Singleton(object):
    _instance = None
   
    # 这里不能使用__init__，因为__init__是在instance已经生成以后才去调用的
    # __new__负责创建新的实例并返回新的实例，__init__负责新创建实例的初始化工作无返回
    
    def __new__(cls, *args, **kwargs):  
        if cls._instance is None:
            cls._instance = object.__new__(cls, *args, **kwargs)
        return cls._instance
        
class MyClass(Singleton):
	pass
```

创建实例时把所有实例的`__dict__`指向同一个字典，这样它们具有相同的属性和方法：

```
class Singleton(object):
    _state = {}
    def __new__(cls, *args, **kw):
        ob = super(Singleton, cls).__new__(cls, *args, **kw)
        ob.__dict__ = cls._state
        return ob
```

Python的装饰器方式，每次执行类中方法时，都会先执行装饰器方法，获取类实例：

```
def singleton(cls, *args, **kw):
    instances = {}
    def getinstance():
        if cls not in instances:
            instances[cls] = cls(*args, **kw)
        return instances[cls]
    return getinstance

@singleton
class MyClass:
	pass
```

使用`metaclass`：

```
class Singleton(type):
    _instances = {}
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super(Singleton, cls).__call__(*args, **kwargs)
        return cls._instances[cls]

#Python2
class MyClass(BaseClass):
    __metaclass__ = Singleton

#Python3
class MyClass(metaclass=Singleton):
    pass
```

使用模块引入的方式，可以将类单独定义在一个模块中，因为在python中，一个模块只能被导入一次，所以类似一个单例。

以上都是非线程安全的方法，一般我们在使用时已经足够。但是在多线程程序中，可能出现多个线程同时访问造成单例失败，我们可以引入锁机制来解决这个问题。

### 线程安全

方式一：

```
import threading

class Singleton(object):

    _instance = None

    lock = threading.RLock()

    def __new__(cls):
        cls.lock.acquire()
        if cls._instance is None:
            cls._instance = super(Singleton, cls).__new__(cls)
        cls.lock.release()
        return cls._instance
```

方式二：

```
import threading
try:
    from synchronize import make_synchronized
except ImportError:
    def make_synchronized(func):
        import threading
        func.__lock__ = threading.Lock()

        def synced_func(*args, **kws):
            with func.__lock__:
                return func(*args, **kws)

        return synced_func

class Singleton(object):
    instance = None

    @make_synchronized
    def __new__(cls, *args, **kwargs):
        if cls.instance is None:
            cls.instance = object.__new__(cls, *args, **kwargs)
        return cls.instance

    def __init__(self):
        pass
```

### 设计模式的分类

| 类型   | 设计模式                                     |
| ---- | ---------------------------------------- |
| 创建型  | 工厂方法模式、抽象工厂模式、单例模式、建造者模式、原型模式            |
| 结构型  | 适配器模式、装饰器模式、代理模式、外观模式、桥接模式、组合模式、享元模式     |
| 行为型  | 策略模式、模板方法模式、观察者模式、迭代子模式、责任链模式、命令模式、备忘录模式、状态模式、访问者模式、中介者模式、解释器模式 |

### 设计模式的六大原则：

**总原则**：开闭原则

开闭原则就是说对扩展开放，对修改关闭。在程序需要进行拓展的时候，不能去修改原有的代码，而是要扩展原有代码，实现一个热插拔的效果。所以一句话概括就是：为了使程序的扩展性好，易于维护和升级。想要达到这样的效果，我们需要使用接口和抽象类等，后面的具体设计中我们会提到这点。

**单一职责原则**

不要存在多于一个导致类变更的原因，也就是说每个类应该实现单一的职责，如若不然，就应该把类拆分。 

**里氏替换原则**

里氏代换原则面向对象设计的基本原则之一。 里氏代换原则中说，任何基类可以出现的地方，子类一定可以出现。 LSP是继承复用的基石，只有当衍生类可以替换掉基类，软件单位的功能不受到影响时，基类才能真正被复用，而衍生类也能够在基类的基础上增加新的行为。里氏代换原则是对“开-闭”原则的补充。实现“开-闭”原则的关键步骤就是抽象化。而基类与子类的继承关系就是抽象化的具体实现，所以里氏代换原则是对实现抽象化的具体步骤的规范。

里氏替换原则中，子类对父类的方法尽量不要重写和重载。因为父类代表了定义好的结构，通过这个规范的接口与外界交互，子类不应该随便破坏它。

**依赖倒转原则**

这个是开闭原则的基础，具体内容：面向接口编程，依赖于抽象而不依赖于具体。写代码时用到具体类时，不与具体类交互，而与具体类的上层接口交互。

**接口隔离原则**

这个原则的意思是：每个接口中不存在子类用不到却必须实现的方法，如果不然，就要将接口拆分。使用多个隔离的接口，比使用单个接口（多个接口方法集合到一个的接口）要好。

**迪米特法则（最少知道原则）**

就是说：一个类对自己依赖的类知道的越少越好。也就是说无论被依赖的类多么复杂，都应该将逻辑封装在方法的内部，通过public方法提供给外部。这样当被依赖的类变化时，才能最小的影响该类。

最少知道原则的另一个表达方式是：只与直接的朋友通信。类之间只要有耦合关系，就叫朋友关系。耦合分为依赖、关联、聚合、组合等。我们称出现为成员变量、方法参数、方法返回值中的类为直接朋友。局部变量、临时变量则不是直接的朋友。我们要求陌生的类不要作为局部变量出现在类中。

**合成复用原则**

原则是尽量首先使用合成/聚合的方式，而不是使用继承。