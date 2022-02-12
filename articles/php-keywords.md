## PHP 系列 - self,final,static,this,parent

最近在看PHP手册，看到类和对象的时候，有一个“后期静态绑定”的内容，self，static，parent这几个关键字弄的我云里雾里，索性查了一些资料，看看self，final，static，this，parent这些关键字到底是什么意思。

[官方文档解释：后期静态绑定](http://php.net/manual/zh/language.oop5.late-static-bindings.php)

> 自 PHP 5.3.0 起，PHP 增加了一个叫做后期静态绑定的功能，用于在继承范围内引用静态调用的类。
>
> 准确说，后期静态绑定工作原理是存储了在上一个“非转发调用”（non-forwarding call）的类名。当进行静态方法调用时，该类名即为明确指定的那个（通常在 :: 运算符左侧部分）；当进行非静态方法调用时，即为该对象所属的类。所谓的“转发调用”（forwarding call）指的是通过以下几种方式进行的静态调用：self::，parent::，static:: 以及 forward_static_call()。可用 get_called_class() 函数来得到被调用的方法所在的类名，static:: 则指出了其范围。
>
> 该功能从语言内部角度考虑被命名为“后期静态绑定”。“后期绑定”的意思是说，static:: 不再被解析为定义当前方法所在的类，而是在实际运行时计算的。也可以称之为“静态绑定”，因为它可以用于（但不限于）静态方法的调用。

因为存在继承和不存在继承的情况下，self，static，parent关键字的作用不同，而this和final是不受继承限制的，我们先来看看this，final。

### 关键字this

```
class name                              //建立了一个名为name的类
{
    private $name;                      //定义属性，私有

    //定义构造函数，用于初始化赋值
    function __construct($name)
    {
        $this->name = $name;            //这里已经使用了this指针语句(1)
    }

    //析构函数
    function __destruct()
    {
    }

    //打印用户名成员函数
    function print_name()
    {
        print($this->name);             //再次使用了this指针语句(2)
    }
}

$obj1 = new name("PHP_Home");            //实例化对象 语句(3)

//执行打印
$obj1->print_name();                    //输出:PBPHome

//第二次实例化对象
$obj2 = new name("PHP");

//执行打印
$obj2->print_name();                    //输出：PHP
```

**Note**：上面的类分别在语句(1)和语句(2)使用了 this 指针，那么当时 this 是指向谁呢？其实 **this 是在实例化的时候来确定指向谁**，比如第一次实例化对象的时候（语句(3)），那么当时 this 就是指向 \$obj1 对象，那么执行语句(2)的打印时就把

```
print($this->name) 
```

变成了 

```
print($obj1->name)
```

那么当然就输出了"PHP_Home"。

第二个实例的时候，

```
print($this->name )
```

变成了

```
print( $obj2->name)
```

于是就输出了"PHP"。所以说，**this 是指向当前对象实例的指针，不指向任何其他对象或类**。

### 关键字final

**如果父类中的方法被声明为 final，则子类无法覆盖该方法。如果一个类被声明为 final，则不能被继承**。

final 方法示例：

```
class BaseClass {
   public function test() {
       echo "BaseClass::test() called\n";
   }
   
   final public function moreTesting() {
       echo "BaseClass::moreTesting() called\n";
   }
}

class ChildClass extends BaseClass {
   public function moreTesting() {
       echo "ChildClass::moreTesting() called\n";
   }
}

// 产生Fatal error: Cannot override final method BaseClass::moreTesting()
```

final 类示例：

```
final class BaseClass {
   public function test() {
       echo "BaseClass::test() called\n";
   }
   
   // 这里无论你是否将方法声明为final，都没有关系
   final public function moreTesting() {
       echo "BaseClass::moreTesting() called\n";
   }
}

class ChildClass extends BaseClass {
}

// 产生 Fatal error: Class ChildClass may not inherit from final class (BaseClass)
```

**Note**：属性不能被定义为 final，只有类和方法才能被定义为 final。

下面我们在存在继承和不存在继承两种情况下看看self，static，parent关键字的作用。

### 不存在继承的时候

顾名思义，不存在继承就是书写一个单独的类来使用。self 和 static 在范围解析操作符 （::） 的使用上，并无区别。

在静态函数中，self 和 static 可以调用静态属性和静态函数（没有实例化类，因此不能调用非静态的属性和函数）。
在非静态函数中，self 和 static 可以调用静态属性和静态函数以及非静态函数。
不存在继承的情况下，self 和 static 的作用是一样的，同时可替换为 **类名::** 的方式调用。

```
class Demo {
    public static $static;
    public $Nostatic;

    public function __construct() {
        self::$static = "static";
        $this->Nostatic = "Nostatic";
    }

    public static function get() {
        return __CLASS__;
    }

    public function show() {
        return "this is function show with " . $this->Nostatic;
    }

    public function test() {
        echo Demo::$static . "<br/>";   //使用类名调用静态属性
        echo Demo::get() . "<br/>";     //使用类名调用静态属性
        echo Demo::show() . "<br/>";    //使用类名调用静态属性
        echo self::$static . "<br/>";   //self调用静态属性
        echo self::show() . "<br/>";    //self调用非静态方法
        echo self::get() . "<br/>";     //self调用静态方法
        echo static::$static . "<br/>"; //static调用静态属性
        echo static::show() . "<br/>";  //static调用非静态方法
        echo static::get() . "<br/>";   //static调用静态方法
    }
}

$obj = new Demo();
$obj->test();
```

输出结果为：

```
static
Demo
this is function show with Nostatic
static
this is function show with Nostatic
Demo
static
this is function show with Nostatic
Demo
```

### 存在继承的时候

示例1：

```
class A {
    static function getClassName() {
        return "this is class A";
    }

    static function testSelf() {
        echo self::getClassName();
    }

    static function testStatic() {
        echo static::getClassName();
    }
}

class B extends A {
    static function getClassName() {
        return "this is class B";
    }
}

B::testSelf();
B::testStatic();
```

输出结果为：

```
this is class A
this is class B
```

Note：self 调用的静态方法或属性始终表示其在使用的时候的当前类（A）的方法或属性，可以替换为其类名，static 调用的静态方法或属性会在继承中被其子类重写覆盖，应该替换为对应的子类名（B）。

对于parent关键字，用于调用父类的方法和属性。在静态方法中，可以调用父类的静态方法和属性；在非静态方法中，可以调用父类的方法和属性。

```
class A {
    public static $static;
    public $Nostatic;

    public function __construct() {
        self::$static = "static";
        $this->Nostatic = "Nostatic";
    }

    public static function staticFun() {
        return self::$static;
    }

    public function noStaticFun() {
        return "this is function show with " . $this->Nostatic;
    }
}

class B extends A {
    static function testS() {
        echo parent::staticFun();
    }

    function testNoS() {
        echo parent::noStaticFun();
    }
}

$obj = new B();
$obj->testS();
$obj->testNoS();
```

输出结果为：

```
static
this is function show with Nostatic
```

### PHP手册中的例子

```
class A {
    public static function foo() {
        static::who();
    }

    public static function who() {
        echo __CLASS__ . "\n";
    }
}

class B extends A {
    public static function test() {
        A::foo();
        parent::foo();
        self::foo();
    }

    public static function who() {
        echo __CLASS__ . "\n";
    }
}

class C extends B {
    public static function who() {
        echo __CLASS__ . "\n";
    }
}

C::test();
```

输出结果为：

```
A
C
C
```

我们分析 test() 方法：

```
public static function test() {
        A::foo();
        parent::foo();
        self::foo();
}
```

A::foo()；这个语句是可以在任何地方执行的，它表示使用A去调用静态方法 foo() 得到“A”。 

parent::foo()；C 的 parent 是 B ， B 的parent是A，回溯找到了A 的 foo 方法；static::who()；语句中的 static:: 调用的方法会被子类覆盖，所以优先调用 C 的 who() 方法，如果 C 的 who 方法不存在会调用 B 的 who 方法，如果 B 的 who 方法不存在会调用 A 的 who 方法。所以，输出结果是"C"。

self::foo()；这个 self:: 是在 B 中使用的，所以 self:: 等价于 B:: ，但是 B 没有实现 foo() 方法，B 又继承自 A，所以我们实际上调用了 A::foo() 这个方法。foo() 方法使用了static::who() 语句，导致我们又调用了 C 的 who 函数。



Read More:

> [PHP中this,self,parent的区别](http://www.cnblogs.com/myjavawork/articles/1793664.html)  
>
> [Final 关键字](http://php.net/manual/zh/language.oop5.final.php)   
>
> [后期静态绑定](http://php.net/manual/zh/language.oop5.late-static-bindings.php)  
>
> [PHP中的self、static、parent关键字](http://blog.csdn.net/koastal/article/details/52166246)