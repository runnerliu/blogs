## PHP 系列 - 变量的内部存储（值和类型）

在翻看  [PHP手册](http://php.net/docs.php)  的时候，看到了  [垃圾回收机制](http://php.net/manual/zh/features.gc.php)  ，介绍了PHP 5.3新的垃圾回收机制（GC）的特点，里面涉及到了PHP变量内部存储的知识，如果你想对PHP变量存储结构有一个了解或想对PHP变量加深理解的话，本文是适合你的，比较深入的去看源代码吧。

PHP是一种**弱类型**的脚本语言，弱类型不表示PHP的变量没有类型区分，PHP变量有8种原始类型。

四种标量类型：

- Boolean（布尔型）
- Integer（整型）
- Float（浮点型）
- String（字符串）

两种复合类型：

- Array（数组）
- Object（对象）

两种特殊类型：

- Resource（资源）
- NULL

我们都知道，在PHP程序运行中，可以将变量从一种类型转换为另一种类型，那么PHP是怎么实现这个过程的呢？

在PHP引擎（Zend）内部，变量都是用如下的结构体来表示的：

```
struct _zval_struct {  
    /* Variable information */  
    zvalue_value value;     /* value */  
    zend_uint refcount__gc;  
    zend_uchar type;    	/* active type */  
    zend_uchar is_ref__gc;  
};
```

其中：

value：存储变量的值；

refcount__gc：引用计数；

type：变量的动态类型；

is_ref__gc：是否为引用。

type的各种类型都被定义成了宏：

```
#define IS_NULL     0  
#define IS_LONG     1  
#define IS_DOUBLE   2  
#define IS_BOOL     3  
#define IS_ARRAY    4  
#define IS_OBJECT   5  
#define IS_STRING   6  
#define IS_RESOURCE 7  
#define IS_CONSTANT 8  
#define IS_CONSTANT_ARRAY   9  
```

zvalue_value 是真正保存数据的关键部分，定义为一个联合：

```
typedef union _zvalue_value {  
    long lval;                  /* long value */  
    double dval;                /* double value */  
    struct {  
        char *val;  
        int len;  
    } str;  
    HashTable *ht;              /* hash table value */  
    zend_object_value obj;  
} zvalue_value;  
```

PHP根据 zval 中的 type 字段来储存一个变量的真正类型，然后根据 type 来选择如何获取 zvalue_value 的值，比如对于整型和bool值：

```
zval.type = IS_LONG;	//整型
zval.type = IS_BOOL;	//布尔型
```

就去取 zval.value.lval，对于 bool 值来说 lval∈(0|1)；如果是双精度，或者 float 则会去取 zval.value 的 dval。而如果是字符串，那么：

```
zval.type = IS_STRING	//字符串
```

这个时候，就会取 zval.value.str 而这个也是个结构，存有C类型的字符串和字符串的长度。

而对于数组和对象，则type分别对应IS_ARRAY、 IS_OBJECT,，相对应的则分别取 zval.value.ht 和 obj。

比较特别的是资源，在PHP中，资源是个很特别的变量，任何不属于PHP内建的变量类型的变量，都会被看作成资源来进行保存，比如，数据库句柄、打开的文件句柄等等。 对于资源：

```
 type = IS_RESOURCE		//资源
```

这个时候，会去取 zval.value.lval， 此时的 lval 是个整型的指示器， 然后 PHP 会再根据这个指示器在 PHP 内建的一个资源列表中查询相对应的资源，此时的 lval 就好像是对应于资源链表的偏移值。

```
ZEND_FETCH_RESOURCE(con, type, zval *, default, resource_name, resource_type);
```

借用这样的机制，PHP就实现了弱类型，因为对于ZE（Zend）的来说，它所面对的永远都是同一种类型，那就是 zval。

在了解了 PHP 变量的内部存储后，新的问题就来了，ZE是如何把用户自定义变量和内部结构 zval 联系起来的呢？

```
<?php
  $var = "laruence";
  echo $var;
?>
```

PHP内部都是使用 zval 来表示变量的，但是对于上面的脚本，我们的变量是有名字的\$var。而 zval 中并没有相应的字段来体现变量名。在PHP中，所有的变量都会存储在一个数组中（确切的说是hash table）。

当你创建一个变量的时候，PHP会为这个变量分配一个 zval，填入相应的变量值，然后将这个变量的名字、和指向这个zval的指针填入一个数组中。然后，当你获取这个变量的时候，PHP会通过查找这个数组，获得对应的 zval。

查看_zend_executor_globals结构（这个结构在PHP的执行器保存一些执行相关的上下文信息）：

```
struct _zend_executor_globals {
     ....
    HashTable *active_symbol_table;	/*活动符号表*/
    HashTable symbol_table;     	/*全局符号表*/
 
    HashTable included_files;   
 
    jmp_buf *bailout;
    int error_reporting;
     .....
}
```

以上只是对PHP变量内部存储的简单介绍，如果想深入了解，建议研究PHP的源码。



Read More:

> [深入PHP变量存储结构](http://blog.csdn.net/wenzhou1219/article/details/16832067)  [变量的内部存储：值和类型](http://blog.csdn.net/phpkernel/article/details/5718003)

