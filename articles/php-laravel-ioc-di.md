## Laravel 系列 - 理解控制反转(IoC)和依赖注入(DI)

容器，字面上理解就是装东西的东西。常见的变量、对象属性等都可以算是容器。一个容器能够装什么，全部取决于你对该容器的定义。当然，有这样一种容器，它存放的不是文本、数值，而是对象、对象的描述（类、接口）或者是提供对象的回调，通过这种容器，我们得以实现许多高级的功能，其中最常提到的，就是 “解耦” 、“依赖注入(DI)”。本文就从这里开始。

### IoC 容器 - Laravel 的核心

Laravel 的核心就是一个 IoC 容器，根据文档，称其为“服务容器”，顾名思义，该容器提供了整个框架中需要的一系列服务。作为初学者，很多人会在这一个概念上犯难，因此，我打算从一些基础的内容开始讲解，通过理解面向对象开发中依赖的产生和解决方法，来逐渐揭开“依赖注入”的面纱，逐渐理解这一神奇的设计理念。

本文一大半内容都是通过举例来让读者去理解什么是 IoC（控制反转） 和 DI（依赖注入），通过理解这些概念，来更加深入。更多关于 Laravel 服务容器的用法建议阅读文档即可。

### IoC 容器诞生的故事

讲解 IoC 容器有很多的文章，我之前也写过。但现在我打算利用当下的灵感重新来过，那么开始吧。

#### 超人和超能力，依赖的产生

面向对象编程，有以下几样东西无时不刻的接触：接口、类还有对象。这其中，接口是类的原型，一个类必须要遵守其实现的接口；对象则是一个类实例化后的产物，我们称其为一个实例。当然这样说肯定不利于理解，我们就实际的写点中看不中用的代码辅助学习。

> 怪物横行的世界，总归需要点超级人物来摆平。

我们把一个“超人”作为一个类：

```
class Superman {}
```

我们可以想象，一个超人诞生的时候肯定拥有至少一个超能力，这个超能力也可以抽象为一个对象，为这个对象定义一个描述他的类吧。一个超能力肯定有多种属性、（操作）方法，这个尽情的想象，但是目前我们先大致定义一个只有属性的“超能力”，至于能干啥，我们以后再丰富：

```
class Power {
    /**
     * 能力值
     */
    protected $ability;

    /**
     * 能力范围或距离
     */
    protected $range;

    public function __construct($ability, $range)
    {
        $this->ability = $ability;
        $this->range = $range;
    }
}
```

这时候我们回过头，修改一下之前的“超人”类，让一个“超人”创建的时候被赋予一个超能力：

```
class Superman
{
    protected $power;

    public function __construct()
    {
        $this->power = new Power(999, 100);
    }
}
```

这样的话，当我们创建一个“超人”实例的时候，同时也创建了一个“超能力”的实例，但是，我们看到了一点，“超人”和“超能力”之间不可避免的产生了一个依赖。

所谓“依赖”，就是 “我若依赖你，我就不能离开你”。

在一个贯彻面向对象编程的项目中，这样的依赖随处可见。少量的依赖并不会有太过直观的影响，我们随着这个例子逐渐铺开，让大家慢慢意识到，当依赖达到一个量级时，是怎样一番噩梦般的体验。当然，我也会自然而然的讲述如何解决问题。

#### 一堆乱麻，可怕的依赖

之前的例子中，超能力类实例化后是一个具体的超能力，但是我们知道，超人的超能力是多元化的，每种超能力的方法、属性都有不小的差异，没法通过一种类描述完全。我们现在进行修改，我们假设超人可以有以下多种超能力：

- 飞行，属性有：飞行速度、持续飞行时间
- 蛮力，属性有：力量值
- 能量弹，属性有：伤害值、射击距离、同时射击个数

我们创建了如下类：

```
class Flight
{
    protected $speed;
    protected $holdtime;
    public function __construct($speed, $holdtime) {}
}

class Force
{
    protected $force;
    public function __construct($force) {}
}

class Shot
{
    protected $atk;
    protected $range;
    protected $limit;
    public function __construct($atk, $range, $limit) {}
}
```

为了省事儿我没有详细写出 ` __construct()`  这个构造函数的全部，只写了需要传递的参数。

好了，这下我们的超人有点“忙”了。在超人初始化的时候，我们会根据需要来实例化其拥有的超能力，大致如下：

```
class Superman
{
    protected $power;

    public function __construct()
    {
        $this->power = new Fight(9, 100);
        // $this->power = new Force(45);
        // $this->power = new Shot(99, 50, 2);
        /*
        $this->power = array(
            new Force(45),
            new Shot(99, 50, 2)
        );
        */
    }
}
```

我们需要自己手动的在构造函数内（或者其他方法里）实例化一系列需要的类，这样并不好。可以想象，假如需求变更（不同的怪物横行地球），需要更多的有针对性的新的超能力，或者需要变更超能力的方法，我们必须 重新改造 超人。换句话说就是，改变超能力的同时，我还得重新制造个超人。效率太低了！新超人还没创造完成世界早已被毁灭。

这时，灵机一动的人想到：为什么不可以这样呢？超人的能力可以被随时更换，只需要添加或者更新一个芯片或者其他装置啥的（想到钢铁侠没）。这样的话就不要整个重新来过了。

对，就是这样的。

我们不应该手动在 “超人” 类中固化了他的 “超能力” 初始化的行为，而转由外部负责，由外部创造超能力模组、装置或者芯片等（我们后面统一称为 “模组”），植入超人体内的某一个接口，这个接口是一个既定的，只要这个 “模组” 满足这个接口的装置都可以被超人所利用，可以提升、增加超人的某一种能力。这种由外部负责其依赖需求的行为，我们可以称其为 “控制反转（IoC）”。

#### 工厂模式，转移依赖

当然，实现控制反转的方法有几种。在这之前，不如我们先了解一些好玩的东西。

> 我们可以想到，组件、工具（或者超人的模组），是一种可被生产的玩意儿，生产的地方当然是 “工厂（Factory）”，于是有人就提出了这样一种模式： 工厂模式。

工厂模式，顾名思义，就是一个类所依赖的外部事物的实例，都可以被一个或多个 “工厂” 创建的这样一种开发模式，就是 “工厂模式”。

我们为了给超人制造超能力模组，我们创建了一个工厂，它可以制造各种各样的模组，且仅需要通过一个方法：

```
class SuperModuleFactory
{
    public function makeModule($moduleName, $options)
    {
        switch ($moduleName) {
            case 'Fight': 
                return new Fight($options[0], $options[1]);
            case 'Force': 
                return new Force($options[0]);
            case 'Shot': 
                return new Shot($options[0], $options[1], $options[2]);
        }
    }
}
```

这时候，超人 创建之初就可以使用这个工厂：

```
class Superman
{
    protected $power;

    public function __construct()
    {
        // 初始化工厂
        $factory = new SuperModuleFactory;

        // 通过工厂提供的方法制造需要的模块
        $this->power = $factory->makeModule('Fight', [9, 100]);
        // $this->power = $factory->makeModule('Force', [45]);
        // $this->power = $factory->makeModule('Shot', [99, 50, 2]);
        /*
        $this->power = array(
            $factory->makeModule('Force', [45]),
            $factory->makeModule('Shot', [99, 50, 2])
        );
        */
    }
}
```

可以看得出，我们不再需要在超人初始化之初，去初始化许多第三方类，只需初始化一个工厂类，即可满足需求。但这样似乎和以前区别不大，只是没有那么多 new 关键字。其实我们稍微改造一下这个类，你就明白，工厂类的真正意义和价值了。

```
class Superman
{
    protected $power;

    public function __construct(array $modules)
    {
        // 初始化工厂
        $factory = new SuperModuleFactory;

        // 通过工厂提供的方法制造需要的模块
        foreach ($modules as $moduleName => $moduleOptions) {
            $this->power[] = $factory->makeModule($moduleName, $moduleOptions);
        }
    }
}

// 创建超人
$superman = new Superman([
    'Fight' => [9, 100],
    'Shot' => [99, 50, 2]
]);
```

现在修改的结果令人满意。现在，“超人” 的创建不再依赖任何一个 “超能力” 的类，我们如若修改了或者增加了新的超能力，只需要针对修改 SuperModuleFactory 即可。扩充超能力的同时不再需要重新编辑超人的类文件，使得我们变得很轻松。但是，这才刚刚开始。

### IoC 容器的重要组成 - 依赖注入

由 “超人” 对 “超能力” 的依赖变成 “超人” 对 “超能力模组工厂” 的依赖后，对付小怪兽们变得更加得心应手。但这也正如你所看到的，依赖并未解除，只是由原来对多个外部的依赖变成了对一个 “工厂” 的依赖。假如工厂出了点麻烦，问题变得就很棘手。

> 其实大多数情况下，工厂模式已经足够了。工厂模式的缺点就是：接口未知（即没有一个很好的契约模型，关于这个我马上会有解释）、产生对象类型单一。总之就是，还是不够灵活。虽然如此，工厂模式依旧十分优秀，并且适用于绝大多数情况。不过我们为了讲解后面的依赖注入 ，这里就先夸大一下工厂模式的缺陷咯。

我们知道，超人依赖的模组，我们要求有统一的接口，这样才能和超人身上的注入接口对接，最终起到提升超能力的效果。

事实上，我之前说谎了，不仅仅只有一堆小怪兽，还有更多的大怪兽。嘿嘿。额，这时候似乎工厂的生产能力显得有些不足 —— 由于工厂模式下，所有的模组都已经在工厂类中安排好了，如果有新的、高级的模组加入，我们必须修改工厂类（好比增加新的生产线）：

```
class SuperModuleFactory
{
    public function makeModule($moduleName, $options)
    {
        switch ($moduleName) {
            case 'Fight': 
                return new Fight($options[0], $options[1]);
            case 'Force': 
                return new Force($options[0]);
            case 'Shot': 
                return new Shot($options[0], $options[1], $options[2]);
            // case 'more': .......
            // case 'and more': .......
            // case 'and more': .......
            // case 'oh no! its too many!': .......
        }
    }
}
```

看到没。。。噩梦般的感受！

> 其实灵感就差一步！你可能会想到更为灵活的办法！对，下一步就是我们今天的主要配角 —— DI （依赖注入）

由于对超能力模组的需求不断增大，我们需要集合整个世界的高智商人才，一起解决问题，不应该仅仅只有几个工厂垄断负责。不过高智商人才们都非常自负，认为自己的想法是对的，创造出的超能力模组没有统一的接口，自然而然无法被正常使用。这时我们需要提出一种契约，这样无论是谁创造出的模组，都符合这样的接口，自然就可被正常使用。

```
interface SuperModuleInterface
{
    /**
     * 超能力激活方法
     *
     * 任何一个超能力都得有该方法，并拥有一个参数
     *@param array $target 针对目标，可以是一个或多个，自己或他人
     */
    public function activate(array $target);
}
```

上文中，我们定下了一个接口 （超能力模组的规范、契约），所有被创造的模组必须遵守该规范，才能被生产。

其实，这就是 php 中接口（ interface ）的用处和意义！很多人觉得，为什么 php 需要接口这种东西？难道不是 java 、 C# 之类的语言才有的吗？这么说，只要是一个正常的面向对象编程语言（虽然 php 可以面向过程），都应该具备这一特性。因为一个 对象（object） 本身是由他的模板或者原型 —— 类 （class） ，经过实例化后产生的一个具体事物，而有时候，实现统一种方法且不同功能（或特性）的时候，会存在很多的类（class），这时候就需要有一个契约，让大家编写出可以被随时替换却不会产生影响的接口。这种由编程语言本身提出的硬性规范，会增加更多优秀的特性。

虽然有些绕，但通过我们接下来的实例，大家会慢慢领会接口带来的好处。

这时候，那些提出更好的超能力模组的高智商人才，遵循这个接口，创建了下述（模组）类：

```
/**
 * X-超能量
 */
class XPower implements SuperModuleInterface
{
    public function activate(array $target)
    {
        // 这只是个例子。。具体自行脑补
    }
}

/**
 * 终极炸弹
 */
class UltraBomb implements SuperModuleInterface
{
    public function activate(array $target)
    {
        // 这只是个例子。。具体自行脑补
    }
}
```

同时，为了防止有些 “砖家” 自作聪明，或者一些叛徒恶意捣蛋，不遵守契约胡乱制造模组，影响超人，我们对超人初始化的方法进行改造：

```
class Superman
{
    protected $module;

    public function __construct(SuperModuleInterface $module)
    {
        $this->module = $module;
    }
}
```

改造完毕！现在，当我们初始化 “超人” 类的时候，提供的模组实例必须是一个` SuperModuleInterface` 接口的实现。否则就会提示错误。

正是由于超人的创造变得容易，一个超人也就不需要太多的超能力，我们可以创造多个超人，并分别注入需要的超能力模组即可。这样的话，虽然一个超人只有一个超能力，但超人更容易变多，我们也不怕怪兽啦！

现在有人疑惑了，你要讲的依赖注入呢？

其实，上面讲的内容，正是依赖注入。

#### 什么叫做依赖注入？

本文从开头到现在提到的一系列依赖，只要不是由内部生产（比如初始化、构造函数 __construct 中通过工厂方法、自行手动 new 的），而是由外部以参数或其他形式注入的，都属于依赖注入（DI） 。是不是豁然开朗？事实上，就是这么简单。下面就是一个典型的依赖注入：

```
// 超能力模组
$superModule = new XPower;
// 初始化一个超人，并注入一个超能力模组依赖
$superMan = new Superman($superModule);
```

关于依赖注入这个本文的主要配角，也就这么多需要讲的。理解了依赖注入，我们就可以继续深入问题。

### 更为先进的工厂 - IoC容器

刚刚列了一段代码：

```
$superModule = new XPower;
$superMan = new Superman($superModule);
```

读者应该看出来了，手动的创建了一个超能力模组、手动的创建超人并注入了刚刚创建超能力模组。呵呵，手动。

> 现代社会，应该是高效率的生产，干净的车间，完美的自动化装配。

一群怪兽来了，如此低效率产出超人是不现实，我们需要自动化 —— 最多一条指令，千军万马来相见。我们需要一种高级的生产车间，我们只需要向生产车间提交一个脚本，工厂便能够通过指令自动化生产。这种更为高级的工厂，就是工厂模式的升华 —— IoC 容器。

```
class Container
{
    protected $binds;

    protected $instances;

    public function bind($abstract, $concrete)
    {
    	// Todo: 向 container 添加一种对象的的生产方式
 
    	// $abstract: 第一个参数 $abstract, 一般为一个字符串(有时候也会是一个接口), 
    	// 当你需要 make 这个类的对象的时候, 传入这个字符串(或者接口), 这样make 就知道制造什么样的对象了
    	// $concrete: 第二个参数 $concrete, 一般为一个 Closure 或者 一个单例对象, 用于说明制造这个对象的方式
        
        if ($concrete instanceof Closure) {
            $this->binds[$abstract] = $concrete;
        } else {
            $this->instances[$abstract] = $concrete;
        }
    }

    public function make($abstract, $parameters = [])
    {
    	// Todo: 生产一种对象
 
    	// $abstract: 在bind方法中已经介绍过
    	// $parameters: 生产这种对象所需要的参数
    	
        if (isset($this->instances[$abstract])) {
            return $this->instances[$abstract];
        }

        array_unshift($parameters, $this);

        return call_user_func_array($this->binds[$abstract], $parameters);
    }
}
```

这时候，一个十分粗糙的容器就诞生了。现在的确很简陋，但不妨碍我们进一步提升他。先着眼现在，看看这个容器如何使用吧：

```
// 创建一个容器（后面称作超级工厂）
$container = new Container;

// 向该超级工厂添加超人的生产脚本
$container->bind('superman', function($container, $moduleName) {
    return new Superman($container->make($moduleName));
});

// 向该超级工厂添加超能力模组的生产脚本
$container->bind('xpower', function($container) {
    return new XPower;
});

// 同上
$container->bind('ultrabomb', function($container) {
    return new UltraBomb;
});

// ****************** 华丽丽的分割线 **********************
// 开始启动生产
$superman_1 = $container->make('superman', ['xpower']);
$superman_2 = $container->make('superman', ['ultrabomb']);
$superman_3 = $container->make('superman', ['xpower']);
// ......
```

看到没？通过最初的 绑定（`bind`） 操作，我们向 超级工厂 注册了一些生产脚本，这些生产脚本在生产指令下达之时便会执行。发现没有？我们彻底的解除了 超人 与 超能力模组 的依赖关系，更重要的是，容器类也丝毫没有和他们产生任何依赖！我们通过注册、绑定的方式向容器中添加一段可以被执行的回调（可以是匿名函数、非匿名函数、类的方法）作为生产一个类的实例的 脚本 ，只有在真正的 生产（`make`） 操作被调用执行时，才会触发。

这样一种方式，使得我们更容易在创建一个实例的同时解决其依赖关系，并且更加灵活。当有新的需求，只需另外绑定一个“生产脚本”即可。

实际上，真正的 IoC 容器更为高级。我们现在的例子中，还是需要手动提供超人所需要的模组参数，但真正的 IoC 容器会根据类的依赖需求，自动在注册、绑定的一堆实例中搜寻符合的依赖需求，并自动注入到构造函数参数中去。Laravel 框架的服务容器正是这么做的。

不过我告诉大家，这种自动搜寻依赖需求的功能，是通过反射（Reflection）实现的，恰好的，php 完美的支持反射机制！关于反射，php 官方文档有详细的资料，并且中文翻译基本覆盖，足够学习和研究：

现在，到目前为止，我们已经不再惧怕怪兽们了。高智商人才集思广益，井井有条，根据接口契约创造规范的超能力模组。超人开始批量产出。最终，人人都是超人，你也可以是哦！

### 重新审视 Laravel 的核心

现在，我们开始慢慢解读 Laravel 的核心。其实，Laravel 的核心就是一个 IoC 容器，也恰好是我之前所说的高级的 IoC 容器。

可以说，Laravel 的核心本身十分轻量，并没有什么很神奇很实质性的应用功能。很多人用到的各种功能模块比如 Route（路由）、Eloquent ORM（数据库 ORM 组件）、Request（请求）以及 Response（响应）等等等等，实际上都是与核心无关的类模块提供的，这些类从注册到实例化，最终被你所使用，其实都是 Laravel 的服务容器负责的。

我们以大家最常见的 Route 类作为例子。大家可能经常见到路由定义是这样的：

```
Route::get('/', function() {
    // ......
});
```

实际上， Route 类被定义在这个命名空间：`Illuminate\Routing\Router`，文件 `vendor/laravel/framework/src/Illuminate/Routing/Router.php`。

我们通过打开发现，这个类的这一系列方法，如 `get`，`post`，`any` 等都不是静态（`static`）方法，这是怎么一回事儿？不要急，我们继续。

#### 服务提供者

我们在前文介绍 IoC 容器的部分中，提到了，一个类需要绑定、注册至容器中，才能被“制造”。

对，一个类要被容器所能够提取，必须要先注册至这个容器。既然 Laravel 称这个容器叫做服务容器，那么我们需要某个服务，就得先注册、绑定这个服务到容器，那么提供服务并绑定服务至容器的东西，就是服务提供者（Service Provider）。

虽然，绑定一个类到容器不一定非要通过服务提供者。

但是，我们知道，有时候我们的类、模块会有需要其他类和组件的情况，为了保证初始化阶段不会出现所需要的模块和组件没有注册的情况，Laravel 将注册和初始化行为进行拆分，注册的时候就只能注册，初始化的时候就是初始化。拆分后的产物就是现在的服务提供者。

服务提供者主要分为两个部分，`register`（注册） 和` boot`（引导、初始化），具体参考文档。`register` 负责进行向容器注册“脚本”，但要注意注册部分不要有对未知事物的依赖，如果有，就要移步至 `boot` 部分。

#### 门面（Facade）

我们现在解答之前关于` Route` 的方法为何能以静态方法访问的问题。实际上这个问题文档上有写，简单说来就是模拟一个类，提供一个静态魔术方法`__callStatic`，并将该静态方法映射到真正的方法上。

我们使用的 `Route` 类实际上是 `Illuminate\Support\Facades\Route` 通过 `class_alias()` 函数创造的别名而已，这个类被定义在文件 `vendor/laravel/framework/src/Illuminate/Support/Facades/Route.php` 。

```
<?php 
    namespace Illuminate\Support\Facades;

    /**
     * @see \Illuminate\Routing\Router
     */
    class Route extends Facade {

        /**
         * Get the registered name of the component.
         *
         * @return string
         */
        protected static function getFacadeAccessor()
        {
            return 'router';
        }

}
```

看到这个类之后，并没有找到之前 Route 调用的 `get` 方法，此时我们再看里面的这行注释， `@see\Illuminate\Routing\Router` ，他提示我们去找这个位置，那我们就去找一下，我们又发现了一个 Router 类，而这个 Router 类中，是有 `get` 方法的，看起来这里似乎就是 Route 的真实身份了。

```
namespace Illuminate\Routing;
class Router implements RegistrarContract {
    ...
 
    /**
     * Register a new GET route with the router.
     *
     * @param  string  $uri
     * @param  \Closure|array|string|null  $action
     * @return \Illuminate\Routing\Route
     */
    public function get($uri, $action = null)
    {
        return $this->addRoute(['GET', 'HEAD'], $uri, $action);
    }
 
    ...
}
```

那 Laravel 是如何为 `Illuminate\Support\Facades\Route::class` 这个类找到他的真实身份的呢？

首先， `class Route extends Facade `， `Route` 继承自 `Facade` 类，`Route` 类又调用了静态的 `get` 方法，我们在 `Route` 类，或者是他的父类 `Facade` 中都是无法找到这个 `get` 方法的。 但是在 `Facade` 类中，我们可以发现有一个 `__callStatic()` 魔术方法 ，这个方法的作用就是：如果你想要调用的静态方法在类的定义中并没有声明，那么就会执行 `__callStatic()` 。在我们当前的情景中，静态方法 `get` 并没有被声明，那么当然，我们的类就会转而调用 `__callStatic() `。

```
public static function __callStatic($method, $args)
{
    $instance = static::getFacadeRoot();
 
    if (! $instance) {
        throw new RuntimeException('A facade root has not been set.');
    }
 
    switch (count($args)) {
        case 0:
            return $instance->$method();
        case 1:
            return $instance->$method($args[0]);
        case 2:
            return $instance->$method($args[0], $args[1]);
        case 3:
            return $instance->$method($args[0], $args[1], $args[2]);
        case 4:
            return $instance->$method($args[0], $args[1], $args[2], $args[3]);
        default:
            return call_user_func_array([$instance, $method], $args);
    }
}
```

然后我们看 `__callStatic()` 的执行过程。首先看 `getFacadeRoot()` 是如何执行的：

```
public static function getFacadeRoot()  
{
    return static::resolveFacadeInstance(static::getFacadeAccessor());
}
```

在最初，定义 Route 类时，我们只实现了一个方法 `getFacadeAccessor()` ，这时我们当初定义的字符串，就会在此处用到了，所以上面这个函数，实际上返回的内容就是 `static::resolveFacadeInstance("router");`

我们继续看 `resolveFacadeInstance` 这个函数的执行过程：

```
protected static function resolveFacadeInstance($name)
{  
    //判断是否为对象，当然不是了，$name 是字符串
    if (is_object($name)) {
        return $name;
    }
 
    //判断 resolvedInstance 这个数组中是否存了 $name 相关的信息，当然也没有，因为我们假设程序是第一次执行这里
    if (isset(static::$resolvedInstance[$name])) {
        return static::$resolvedInstance[$name];
    }
 
    // 返回 static::$app[$name]，同时把得到的结果保存到上面验证的数组中
    return static::$resolvedInstance[$name] = static::$app[$name];
}
```

所以我们的程序执行了最后的一个 return， 返回了 `static::$app[‘router’]` 这个值。

$app 就是前面说过的 Laravel Application 类的实例化对象，这个类是一个 IOC Container，实例化过程发生在 Laravel 最开始的时候。

在 Facade 初始化的时候，也让自己有了一个 `static::$app` 这个就是 Application 类的实例化对象。

而 $app 其实并没有 ‘router’ 这个属性，那为什么可以这样调用呢？ 是因为 Application 继承了 Container， 而 Container 又继承了 ArrayAccess 这个类。正是由于 ArrayAccess 的存在，以及 Container 实现了 ArrayAccess 的下面这个方法：

```
public function offsetGet($key)
{
  return $this->make($key);
}
```

所以，当我们使用 `$app['router']` 时，实际上是执行了 `$app->make(‘router’)`，到这里已经比较明显了，这里就是从 $app 这个 IOC Container 中， `make` 了一个 router 的实例。

已经绕的有点远了，不过还好，我们终于要回去了。到最初的 `__callStatic()` 中的：

`$instance = static::getFacadeRoot();`

也就相当于

`$instance = $app->make(‘router’)`

`__callStatic()` 继续往下执行，想一下我们最初的那条代码

```
Route::get('/', function() {
    // ......
});
```

有2个参数，所以会执行到 case 2 这条语句：

```
return $instance->$method($args[0], $args[1]);
```

到这里，我们的 $instance 就是我们的 IOC Container make 出的具有实际功能的实例，这个实例将会执行这个实例的类所声明过的 get 方法。



Read More:

> [Laravel 服务容器实例教程 —— 深入理解控制反转（IoC）和依赖注入（DI）](http://laravelacademy.org/post/769.html/comment-page-5#comment-17821)  [Laravel 从 1 行代码开始，带你系统性的理解 Laravel Service Container 的核心概念](http://www.tuicool.com/articles/jy6RviE)