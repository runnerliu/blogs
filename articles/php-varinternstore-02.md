## PHP 系列 - 变量的内部存储（引用和计数）

在  [PHP内核探索：变量的内部存储（值和类型）](https://runnerliu.github.io/2017/04/15/phpvarinternstore1/#more)  中介绍了PHP变量在引擎中是如何存储以及PHP如何实现其弱类型功能的。本文将从引用和计数的角度继续介绍变量的内部存储。

### 引用计数基本知识

每个php变量存在一个叫"zval"的变量容器中。一个zval变量容器，除了包含变量的类型和值，还包括两个字节的额外信息。第一个是"is\_ref"(此字段在5.3.2版本中是is\_ref\_\_gc)，是个bool值，用来标识这个变量是否是属于引用集合。通过这个字节，php引擎才能把普通变量和引用变量区分开来，由于php允许用户通过使用&来使用自定义引用，zval变量容器中还有一个内部引用计数机制，来优化内存使用。第二个额外字节是"refcount"(此字段在5.3.2版本中是refcount\_\_gc)，用以表示指向这个zval变量容器的变量(也称符号即symbol)个数。所有的符号存在一个符号表中，其中每个符号都有作用域，那些主脚本(比如：通过浏览器请求的的脚本)和每个函数或者方法也都有作用域。

当一个变量被赋常量值时，就会生成一个zval变量容器，如下例这样：

**Example \#1** 生成一个新的zval容器

```
<?php
$a = "new string";
?>
```

在上例中，新的变量a，是在当前作用域中生成的。并且生成了类型为 string 和值为new string的变量容器。在额外的两个字节信息中，"is\_ref"被默认设置为 FALSE，因为没有任何自定义的引用生成。"refcount" 被设定为 1，因为这里只有一个变量使用这个变量容器， 注意到当"refcount"的值是1时，"is\_ref"的值总是FALSE。如果你已经安装了Xdebug，你能通过调用函数 xdebug\_debug\_zval()显示"refcount"和"is\_ref"的值。

**Example \#2 **显示zval信息

```
<?php
xdebug_debug_zval('a');
?>
```

以上例程会输出：

```
a: (refcount=1, is_ref=0)='new string'
```

把一个变量赋值给另一变量将增加引用次数(refcount)。

**Example \#3 **增加一个zval的引用计数

```
<?php
$a = "new string";
$b = $a;
xdebug_debug_zval( 'a' );
?>
```

以上例程会输出：

```
a: (refcount=2, is_ref=0)='new string'
```

这时，引用次数是2，因为同一个变量容器被变量a 和变量 b关联。当没必要时，php不会去复制已生成的变量容器。变量容器在”refcount“变成0时就被销毁. 当任何关联到某个变量容器的变量离开它的作用域(比如：函数执行结束)，或者对变量调用了函数 unset()时，”refcount“就会减1，下面的例子就能说明：

**Example \#4 **减少引用计数

```
<?php
$a = "new string";
$c = $b = $a;
xdebug_debug_zval( 'a' );
unset( $b, $c );
xdebug_debug_zval( 'a' );
?>
```

以上例程会输出：

```
a: (refcount=3, is_ref=0)='new string'
a: (refcount=1, is_ref=0)='new string'
```

如果我们现在执行 unset($a)，包含类型和值的这个变量容器就会从内存中删除。

### 复合类型

当考虑像 array和object这样的复合类型时，事情就稍微有点复杂。与 标量类型的值不同，array和 object类型的变量把它们的成员或属性存在自己的符号表中。这意味着下面的例子将生成三个zval变量容器。

**Example \#5** Creating a array zval

```
<?php
$a = array( 'meaning' => 'life', 'number' => 42 );
xdebug_debug_zval( 'a' );
?>
```

以上例程的输出类似于：

```
a: (refcount=1, is_ref=0)=array (
   'meaning' => (refcount=1, is_ref=0)='life',
   'number' => (refcount=1, is_ref=0)=42
)
```

图示：

![2017-4-15 135317](../images/2017-4-15 135317.png)

这三个zval变量容器是: a，meaning和 number。增加和减少”refcount”的规则和上面提到的一样。下面， 我们在数组中再添加一个元素，并且把它的值设为数组中已存在元素的值：

**Example \#6 **添加一个已经存在的元素到数组中

```
<?php
$a = array( 'meaning' => 'life', 'number' => 42 );
$a['life'] = $a['meaning'];
xdebug_debug_zval( 'a' );
?>
```

以上例程的输出类似于：

```
a: (refcount=1, is_ref=0)=array (
   'meaning' => (refcount=2, is_ref=0)='life',
   'number' => (refcount=1, is_ref=0)=42,
   'life' => (refcount=2, is_ref=0)='life'
)
```

图示：

![2017-4-15 135507](../images/2017-4-15 135507.png)

从以上的Xdebug输出信息，我们看到原有的数组元素和新添加的数组元素关联到同一个"refcount"为2的zval变量容器. 尽管 Xdebug的输出显示两个值为'life'的 zval 变量容器，其实是同一个。 函数xdebug\_debug\_zval()不显示这个信息，但是你能通过显示内存指针信息来看到。

删除数组中的一个元素，就是类似于从作用域中删除一个变量. 删除后,数组中的这个元素所在的容器的“refcount”值减少，同样，当“refcount”为0时，这个变量容器就从内存中被删除，下面又一个例子可以说明：

**Example \#7 **从数组中删除一个元素

```
<?php
$a = array( 'meaning' => 'life', 'number' => 42 );
$a['life'] = $a['meaning'];
unset( $a['meaning'], $a['number'] );
xdebug_debug_zval( 'a' );
?>
```

以上例程的输出类似于：

```
a: (refcount=1, is_ref=0)=array (
   'life' => (refcount=1, is_ref=0)='life'
)
```

现在，当我们添加一个数组本身作为这个数组的元素时，事情就变得有趣，下个例子将说明这个。例中我们加入了引用操作符，否则php将生成一个复制。

**Example \#8 **把数组作为一个元素添加到自己

```
<?php
$a = array( 'one' );
$a[] =& $a;
xdebug_debug_zval( 'a' );
?>
```

以上例程的输出类似于：

```
a: (refcount=2, is_ref=1)=array (
   0 => (refcount=1, is_ref=0)='one',
   1 => (refcount=2, is_ref=1)=...
)
```

图示：

![2017-4-15 135755](../images/2017-4-15 135755.png)

能看到数组变量 (a) 同时也是这个数组的第二个元素(1) 指向的变量容器中“refcount”为 2。上面的输出结果中的"..."说明发生了递归操作, 显然在这种情况下意味着"..."指向原始数组。

跟刚刚一样，对一个变量调用unset，将删除这个符号，且它指向的变量容器中的引用次数也减1。所以，如果我们在执行完上面的代码后，对变量a调用unset，那么变量 a 和数组元素 "1" 所指向的变量容器的引用次数减1，从"2"变成"1"。下例可以说明：

**Example \#9 **Unsetting a

```
(refcount=1, is_ref=1)=array (
   0 => (refcount=1, is_ref=0)='one',
   1 => (refcount=1, is_ref=1)=...
)
```

图示：

![2017-4-15 140001](/images/2017-4-15 140001.png)

### 清理变量容器的问题

尽管不再有某个作用域中的任何符号指向这个结构(就是变量容器)，由于数组元素“1”仍然指向数组本身，所以这个容器不能被清除 。因为没有另外的符号指向它，用户没有办法清除这个结构，结果就会导致内存泄漏。庆幸的是，php将在脚本执行结束时清除这个数据结构，但是在php清除之前，将耗费不少内存。如果你要实现分析算法，或者要做其他像一个子元素指向它的父元素这样的事情，这种情况就会经常发生。当然，同样的情况也会发生在对象上，实际上对象更有可能出现这种情况，因为对象总是隐式的被引用。

如果上面的情况发生仅仅一两次倒没什么，但是如果出现几千次，甚至几十万次的内存泄漏，这显然是个大问题。这样的问题往往发生在长时间运行的脚本中，比如请求基本上不会结束的守护进程或者单元测试中的大的套件中。后者的例子：在给巨大的eZ(一个知名的PHP Library) 组件库的模板组件做单元测试时，就可能会出现问题。有时测试可能需要耗用2GB的内存，而测试服务器很可能没有这么大的内存。

**这就涉及到了PHP中的垃圾回收机制**，将在下一篇博客中介绍。

### 写时复制机制（Copy on write）

考虑以下示例：

```
<?php  
$a = "Hello world";  
$b = $a;  
$b = "new string";  
echo $a;  
echo $b;  
?>  
```

a和b明明是指向同一个zval，为什么修改了b，a还能保持不变呢，这就是copy on write（写时复制）技术，简单的说，当重新给b赋值的时候，会将b从之前的zval中分离出来。分离之后，a和b分别是指向不同的zval了。

> 写时复制技术的一个比较有名的应用是在unix类操作系统内核中，当一个进程调用fork函数生成一个子进程的时候，父子进程拥有相同的地址空间内容，在老版本的系统中，子进程是在fork的时候就将父进程的地址空间中的内容都拷贝一份，对于规模较大的程序这个过程可能会有着很大的开销，更崩溃的是，很多进程在fork之后，直接在子进程中调用exec执行另外一个程序，这样原来花了大量时间从父进程复制的地址空间都还没来得及碰一下就被新的进程地址空间代替，这显然是对资源的极大浪费，所以在后来的系统中，就使用了写时复制技术，fork之后，子进程的地址空间还是简单的指向父进程的地址空间，只有当子进程需要写地址空间中的内容的时候，才会单独分离一份（一般以内存页为单位）给子进程，这样就算子进程马上调用exec函数也没关系，因为根本就不需要从父进程的地址空间中拷贝内容，这样节约了内存同时又提高了速度。

当b从a指向的zval分离出来之后，zval的refcount就要减1，这样由之前的2变成了1，表示这个zval还有一个变量指向它，就是a。b变量指向了一个新的zval，新的zval的refcount为1，值为字符串"new string"，大概过程如下：

```
$a = "Hello world"	//a：(refcount=1, is_ref=0)="Hello world"
$b = $a			//a，b： (refcount=2, is_ref=0)="Hello world"
$b = "new string"	//a： (refcount=1, is_ref=0)="Hello world"   b: (refcount=1, is_ref=0)="new string"(发生分离操作)
```

这个分离逻辑可以表叙为：
对一个一般变量a（is\_ref=0）进行一般赋值操作，如果a所指向的zval的计数refcount大于1,那么需要为a重新分配一个新的zval，并且把之前的zval的计数refcount减少1。

以上为普通赋值的情况，如果是引用赋值，我们看看这个变化过程：

```
$a = "Hello world"	//a： (refcount=1, is_ref=0)="Hello world"
$b = &$a		//a，b： (refcount=2, is_ref=1)="Hello world"
$b = "new string"	//a，b： (refcount=2, is_ref=1)="new string"
```

可以看出来，对一个引用类型的zval进行赋值是不会进行分离操作的，实际上我们再产生一个引用变量的时候是可能出现一个分离操作的，只是时机有些不同：

1. 在普通赋值的情况下，分离操作发生在`$b="new string"`这一步，也就是在对变量赋新的值的时候，才会进行zval分离操作；
2. 在引用赋值的情况下，分离操作有可能发生在`$b = &$a`这一步，也就是在生成引用变量的时候。

情况1就不多解释了，情况2中强调是有可能发生分离，以前面的这代码为例子，是否进行分离与a当前指向的zval的refcount有关系，代码中`$b = &$a` 的时候，a指向的zval的refcount=1，这个时候不需要进行分离操作，但是如果refcount=2，那么就需要分离一个zval出来。比如如下代码：

```
<?php  
$a = "Hello world";  
$c = $a;  
$b = &$a;  
$b = "new string";  
?>  
```

在执行引用赋值的时候，a指向的zval的refcount=2，因为a和c同时指向了这个zval，所以在`$b=&$a`的时候，就需要进行一个分离操作，这个分离操作生成了一个ref=1的zval，并且计数为2，因为a，b两个变量指向分离出来的zval，原来的zval的refcount减少1，所以最终只有c指向一个值为"Hello world"，ref=0的zval1，a和b指向一个值为"Hello world"，ref=1的zval2。 这样我们对c的修改时在操作zval1，对a和b的修改都是在操作zval2，这样就符合引用的特性了。

此过程大致如下：

```
$a = "Hello world";	//a: (refcount=1, is_ref=0)="Hello world"
$c = $a;		// a,c: (refcount=2, is_ref=0)="Hello world"
$b = &$a;		// c: (refcount=1, is_ref=0)="Hello world" a,b: (refcount=2, is_ref=1)="Hello world" (发生分离操作)
$b = "new string";	// c: (refcount=1, is_ref=0)="Hello world" a,b: (refcount=2, is_ref=1)="new string"
```

试想一下如果不进行这个分离会有什么后果？
如果不进行分离，a，b，c都指向了同一个zval，对b的修改也会影响到c，这显然是不符合PHP语言特性的。

这个分离逻辑可以表述为：
将一个一般变量a(is\_ref=0)的引用赋给另外一个变量b的时候，如果a的refcount大于1，那么需要对a进行一次分离操作，分离之后的 zval 的is\_ref等于1，refcount等于2。

### unset的作用

unset()并非一个函数，而是一种语言结构，主要的操作时从当前符号表中删除参数中的符号，比如在全局代码中执行`unset($a)`，那么将会在全局符号表中删除a这个符号。全局符号表是一张哈希表，建立这张表的时候会提供一个表中的项的析构函数，当我们从符号表中删除a的时候，会对符号a指向的项（这里是zval的指针）调用这个析构函数，这个析构函数的主要功能是将a对应的zval的refcount减1，如果refcount变成了0，那么释放这个zval。所以当我们调用unset的时候，不一定能释放变量所占的内存空间，只有当这个变量对应的zval没有别的变量指向它的时候，才会释放掉zval，否则只是对refcount进行减1操作。

### 继续Example

```
如果将代码：  
<?php    
$a = "Hello world";    
$c = $a;    
$b = &$a;    
$b = "new string";    
?>   
  
改为：  
  
<?php    
$a = "Hello world";    
$c = &$a;         
$b = &$a;    
$b = "new string";    
?>   
```

执行以上示例，会发现，输出的a，b，c都是“new string”：

此过程大致如下：

```
$a = "Hello world";		//a：(refcount=1, is_ref=0)="Hello world"
$c = &$a;			//a，c:(refcount=2, is_ref=1)="hello world"
$b = &$a;			//a，b，c:(refcount=3, is_ref=1)="hello world"
$b = "new string";		//a，b，c:(refcount=3, is_ref=1)="new string"
```

我们发现，如果都是引用赋值的话，PHP是不会进行分离的，这种情况与上述情况2还是有区别的。



Read More:

> [引用计数基本知识](http://php.net/manual/zh/features.gc.refcounting-basics.php)  
>
> [变量的内部存储：引用和计数](http://blog.csdn.net/phpkernel/article/details/5732784)  