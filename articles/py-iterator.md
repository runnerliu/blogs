## 迭代器的实现原理

### 可迭代对象与迭代器

Python 一切皆对象，类型对象定义了哪些操作，决定了实例对象拥有哪些行为。

比如类型对象如果定义了 `__iter__`，那么其实例对象便被称为可迭代对象（iterable），像字符串、元组、列表、字典、集合等等都是可迭代对象。而整数、浮点数，由于其类型对象没有定义` __iter__`，所以它们不是可迭代对象。

```python
from typing import Iterable

print(
    isinstance("", Iterable),
    isinstance((), Iterable),
    isinstance([], Iterable),
    isinstance({}, Iterable),
    isinstance(set(), Iterable),
)  # True True True True True

print(
    isinstance(0, Iterable),
    isinstance(0.0, Iterable),
)  # False False
```

可迭代对象的一大特点就是它可以使用 for 循环进行遍历，但是能被 for 循环遍历的则不一定是可迭代对象。我们举个例子：

```python
class A:

    def __getitem__(self, item):
        return f"参数item: {item}"

a = A()
# 内部定义了 __getitem__
# 首先可以让实例对象像字典一样访问属性
print(a["name"])  # 参数item: name
print(a["satori"])  # 参数item: satori

# 此外还可以像可迭代对象一样被 for 循环
# 循环的时候会自动给 item 传值，0 1 2 3...
# 如果内部出现了 StopIteration，循环结束
# 否则会一直循环下去。这里我们手动 break
for idx, val in enumerate(a):
    print(val)
    if idx == 5:
        break
"""
参数item: 0
参数item: 1
参数item: 2
参数item: 3
参数item: 4
参数item: 5
"""
```

所以实现了`__getitem__`的类的实例，也是可以被for循环的，但它并不是可迭代对象。

```python
from typing import Iterable
print(isinstance(a, Iterable))  # False
```

总之判断一个对象是否是可迭代对象，就看它的类型对象有没有实现 `__iter__`。

可迭代对象我们知道了，那什么是迭代器呢？很简单，调用可迭代对象的 `__iter__` 方法，得到的就是迭代器。

### 迭代器的创建

不同类型的对象，都有自己的迭代器，举个例子。

```python
lst = [1, 2, 3]
# 底层调用的其实是 list.__iter__(lst)
# 从 C 的角度上看，就是 PyList_Type.tp_iter(lst)
it = lst.__iter__()
print(it)  # <list_iterator object at 0x000001DC6E898640>
print(
    str.__iter__("")
)  # <str_iterator object at 0x000001DC911B8070>
print(
    tuple.__iter__(())
)  # <tuple_iterator object at 0x000001DC911B8070>
```

迭代器也是可迭代对象，只不过迭代器内部的 `__iter__` 返回的还是它本身。当然啦，在创建迭代器的时候，我们更常用内置函数 iter。

```python
lst = [1, 2, 3]
# 等价于 type(lst).__iter__(lst)
it = iter(lst)
```

但是 iter 函数还有一个鲜为人知的用法，我们来看一下：

```python
val = 0

def foo():
    global val
    val += 1
    return val

# iter 可以接收一个参数: iter(可迭代对象)
# iter 也可以接收两个参数: iter(可调用对象, value)
for i in iter(foo, 5):
    print(i)
"""
1
2
3
4
"""
```

如果接收的是两个参数，那么第一个参数一定是 callable。进行迭代的时候，会不停地调用接收的可调用对象，每次迭代出来的值便是 callable 的返回值。当返回值等于传递第二个参数 value（在底层被称为哨兵）时，终止迭代。我们看一下 iter 函数的底层实现。

```c++
static PyObject *
builtin_iter(PyObject *self, 
    PyObject *const *args, Py_ssize_t nargs)
{
    PyObject *v;
  
    // iter 函数要么接收一个参数, 要么接收两个参数
    if (!_PyArg_CheckPositional("iter", nargs, 1, 2))
        return NULL;
    v = args[0];
    // 如果接收一个参数
    // 那么直接使用 PyObject_GetIter 获取对应的迭代器即可
    // 可迭代对象的类型不同，那么得到的迭代器也不同
    if (nargs == 1)
        return PyObject_GetIter(v);
    // 如果接收的不是一个参数, 那么一定是两个参数
    // 如果是两个参数, 那么第一个参数一定是可调用对象
    if (!PyCallable_Check(v)) {
        PyErr_SetString(PyExc_TypeError,
                        "iter(v, w): v must be callable");
        return NULL;
    }
    // 获取value(哨兵)
    PyObject *sentinel = args[1];
    //调用PyCallIter_New
    //得到 calliterobject 对象
    /*
    该对象位于 Objects/iterobject.c 中
    */
    return PyCallIter_New(v, sentinel);
}
```

以上就是 iter 函数的内部逻辑，既可以接收一个参数，也可以接收两个参数。这里我们只看接收一个可迭代对象的情况，所以核心就在于 PyObject_GetIter，它是根据可迭代对象生成迭代器的关键，我们来看一下它的逻辑是怎么样的？该函数定义在 Objects/abstract.c 中。

```c++
PyObject *
PyObject_GetIter(PyObject *o)
{  
    // 获取可迭代对象的类型对象
    PyTypeObject *t = Py_TYPE(o);
    // 我们说类型对象定义的操作，决定了实例对象的行为
    // 实例对象调用的那些方法都是定义在类型对象里面的
    // 所以obj.func()本质上就是type(obj).func(obj)的语法糖
    getiterfunc f;
    // 所以这里是获取类型对象的 tp_iter 成员
    // 也就是 Python 中的 __iter__
    f = t->tp_iter;
    // 如果 f 为 NULL
    // 说明该类型对象内部的tp_iter成员被初始化为NULL
    // 即内部没有定义 __iter__ 
    // 像str、tuple、list等类型对象，它们的tp_iter成员都是不为NULL的
    if (f == NULL) {
       // 如果 tp_iter 为 NULL，那么解释器会退而求其次
       // 检测该类型对象中是否定义了 __getitem__
       // 如果定义了，那么直接调用PySeqIter_New
       // 得到一个seqiterobject对象
       // 下面的PySequence_Check负责检测类型对象是否实现了__getitem__
        if (PySequence_Check(o))
            return PySeqIter_New(o);
        // 走到这里说明该类型对象既没有__iter__、也没有__getitem__
        // 因此它的实例对象不具备可迭代的性质，于是抛出异常
        return type_error("'%.200s' object is not iterable", o);
    }
    else {
        // 否则说明定义了__iter__，于是直接进行调用
        // Py_TYPE(o)->tp_iter(o) 返回对应的迭代器
        PyObject *res = (*f)(o);
        // 但如果返回值res不为NULL、并且还不是迭代器
        // 证明 __iter__ 的返回值有问题，于是抛出异常
        if (res != NULL && !PyIter_Check(res)) {
            PyErr_Format(PyExc_TypeError,
                         "iter() returned non-iterator "
                         "of type '%.100s'",
                         Py_TYPE(res)->tp_name);
            Py_DECREF(res);
            res = NULL;
        }
        // 返回 res
        return res;
    }
}
```

所以我们看到这便是 iter 函数的底层实现，并且当类型对象内部没有定义 `__iter__` 时，解释器会退而求其次检测内部是否定义了 `__getitem__`。

因此以上就是迭代器的创建过程，每个可迭代对象都有自己的迭代器，而迭代器本质上只是对原始数据的一层封装罢了。

### 迭代器的底层结构

由于迭代器的种类非常多，字符串、元组、列表等等，都有自己的迭代器，这里就不一一介绍了。我们就以列表的迭代器为例，看看迭代器在底层的结构是怎么样的。

```c++
typedef struct {
    PyObject_HEAD
    Py_ssize_t it_index;
    //指向创建该迭代器的列表
    PyListObject *it_seq;
} listiterobject;
```

显然对于列表而言，迭代器就是在其之上进行了一层简单的封装，所谓元素迭代本质上还是基于索引，并且我们每迭代一次，索引就自增 1。一旦出现索引越界，就将 it_seq 设置为 NULL，表示迭代器迭代完毕。

我们实际演示一下：

```python
from ctypes import *

class PyObject(Structure):
    _fields_ = [
        ("ob_refcnt", c_ssize_t),
        ("ob_size", c_void_p)
    ]

class ListIterObject(PyObject):
    _fields_ = [
        ("it_index", c_ssize_t),
        ("it_seq", POINTER(PyObject))
    ]

it = iter([1, 2, 3])
it_obj = ListIterObject.from_address(id(it))

# 初始的时候，索引为0
print(it_obj.it_index)  # 0
# 进行迭代
next(it)
# 索引自增1，此时it_index等于1
print(it_obj.it_index)  # 1
# 再次迭代
next(it)
# 此时it_index等于2
print(it_obj.it_index)  # 2
# 再次迭代
next(it)
# 此时it_index等于3
print(it_obj.it_index)  # 3
```

当 it_index 为 3 的时候，如果再次迭代，那么底层发现 it_index 已超过最大索引，就知道迭代器已经迭代完毕了。然后会将 it_seq 设置为 NULL，并抛出 StopIteration。如果是 for 循环，那么会自动捕获此异常，然后停止循环。

所以这就是迭代器，真的没有想象中的那么神秘，甚至在知道它的实现原理之后，还觉得有点 low。

就是将原始的数据包了一层，加了一个索引而已。所谓的迭代仍然是基于索引来做的，并且每迭代一次，索引自增 1。当索引超出范围时，证明迭代完毕了，于是将 it_seq 设置为 NULL，抛出 StopIteration。

### 元素迭代的具体过程

我们知道在迭代元素的时候，可以通过 next 内置函数，当然它本质上也是调用了对象的 `__next__` 方法。

```c++
static PyObject *
builtin_next(PyObject *self, 
    PyObject *const *args, Py_ssize_t nargs)
{
    PyObject *it, *res;
  
    // 同样接收一个参数或者两个参数
    // 因为调用next函数时，可以传入一个默认值
    // 当迭代器没有元素可以迭代的时候，会返回指定的默认值
    if (!_PyArg_CheckPositional("next", nargs, 1, 2))
        return NULL;

    it = args[0];
    // 第一个参数必须是一个迭代器
    if (!PyIter_Check(it)) {
        // 否则的话, 抛出TypeError
        // 表示第一个参数传递的不是一个迭代器
        PyErr_Format(PyExc_TypeError,
            "'%.200s' object is not an iterator",
            it->ob_type->tp_name);
        return NULL;
    } 
    // it->ob_type 表示获取类型对象，也就是该迭代器的类型
    // 可能是列表的迭代器、元组的迭代器、字符串的迭代器等等
    // 具体是哪一种不重要，因为实现了多态
    // 然后再获取 tp_iternext 成员，相当于__next__
    // 拿到函数指针之后，传入迭代器进行调用
    res = (*it->ob_type->tp_iternext)(it);
    
    // 如果 res 不为 NULL, 那么证明迭代到值了, 直接返回
    if (res != NULL) {
        return res;
    } else if (nargs > 1) {
        // 否则的话，说明 res == NULL，
        // 这意味着迭代完毕了，或者程序出错了
        // 然后看 nargs 是否大于1, 如果大于1, 说明设置了默认值
        PyObject *def = args[1];
        // 如果出现异常
        if (PyErr_Occurred()) {
           // 那么就看该异常的种类
           // 能否匹配 StopIteration
            if(!PyErr_ExceptionMatches(PyExc_StopIteration))
            // 如果不是，说明程序的逻辑有问题
            // 于是直接return NULL，结束执行
            // 然后在 Python 里面我们会看到打印到stderr中的异常信息
                return NULL;
            // 如果是 StopIteration，证明迭代完毕了
            // 但我们设置了默认值，那么就应该返回默认值
            // 而不应该抛出 StopIteration，于是将异常回溯栈给清空
            PyErr_Clear();
        }
        // 然后增加默认值的引用计数, 并返回
        Py_INCREF(def);
        return def;
    } else if (PyErr_Occurred()) {
        //走到这里说明程序出异常了，并且没有指定默认值
        //那么这种情况，不管什么异常都直接抛出
        return NULL;
    } else {
        // 都不是的话，仍是直接抛出 StopIteration
        PyErr_SetNone(PyExc_StopIteration);
        return NULL;
    }
}
```

以上就是next函数的背后逻辑，实际上还是调用了迭代器的`__next__`方法。

```python
lst = [1, 2, 3]
it = iter(lst)
# 然后迭代，等价于next(it)
print(type(it).__next__(it))  # 1
print(type(it).__next__(it))  # 2
print(type(it).__next__(it))  # 3
# 但是next可以指定默认值
# 如果不指定默认值，或者还是type(it).__next__(it)
# 那么就会报错，会抛出StopIteration
print(next(it, 666))  # 666
```

以上就是元素的迭代，但是我们知道内置函数next要更强大一些，因为它还可以指定一个默认值。当然在不指定默认值的情况下，`next(it)` 和 `type(it).__next__(it)` 是等价的。

我们仍以列表的迭代器为例，看看 `__next__` 的具体实现。

```c++
static PyObject *
listiter_next(listiterobject *it)
{
    PyListObject *seq;  //列表
    PyObject *item;     //元素
    
    assert(it != NULL);
    //拿到具体对应的列表
    seq = it->it_seq;
    //如果seq为NULL，证明迭代器已经迭代完毕
    //否则它不会为NULL
    if (seq == NULL)
        return NULL;
    assert(PyList_Check(seq));
    //如果索引小于列表的长度，证明尚未迭代完毕
    if (it->it_index < PyList_GET_SIZE(seq)) {
        //通过索引获取指定元素
        item = PyList_GET_ITEM(seq, it->it_index);
        //it_index自增1
        ++it->it_index;
        //增加引用计数后返回
        Py_INCREF(item);
        return item;
    }
    //否则的话，说明此次索引正好已经超出最大范围
    //意味着迭代完毕了，将it_seq设置为NULL
    //并减少它的引用计数，然后返回
    it->it_seq = NULL;
    Py_DECREF(seq);
    return NULL;
}
```

显然这和我们之前分析的是一样的，以上我们就以列表为例，考察了迭代器的实现原理和元素迭代的具体过程。当然其它对象也有自己的迭代器，有兴趣可以自己看一看。

### 总结

到此，我们再次体会到了 Python 的设计哲学，通过 PyObject * 和 ob_type 实现了多态。原因就在于它们接收的不是对象本身，而是对象的 PyObject * 泛型指针。

不管变量 obj 指向什么样的可迭代对象，都可以交给 iter 函数，会调用类型对象内部的 `__iter__`，底层是 tp_iter，得到对应的迭代器。不管变量 it 指向什么样的迭代器，都可以交给 next 函数进行迭代，会调用迭代器的类型对象的 `__next__`，底层是 tp_iternext，将值迭代出来。

至于`__iter__`和`__next__`本身，每个迭代器都会有，我们这里只以列表的迭代器为例。

回顾整个过程，会发现这是不是实现了多态呢？