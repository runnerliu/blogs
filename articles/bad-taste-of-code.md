## 重构改善既有代码的设计 - 代码的坏味道

《重构，改善既有代码的设计》一书中，将重构时机用味道来形容，“代码的坏味道”顾名思义就是我们的程序中存在一些不合理、难以理解、难以扩展的设计，本文参考书中的“坏味道条款”列举程序设计中可能遇到的可重构代码。

### Alternative Class with Different Interfaces

说明：异曲同工的类。如果两个函数做同一件事儿，却有着不同的签名。

拟可重构方法：Rename Method(函数重命名)，Move Method(搬移函数)

### Comments

说明：过多的注释。并不是说不该写注释，但是过多无谓的注释，不如几行简洁的代码，毕竟代码说明了一切。

拟可重构方法：Extract Method(提炼函数)，Introduce Assertion(引入断言)

### Data Class

说明：纯稚的类。某个类拥有一些字段，已经用于访问这些字段的函数。

拟可重构方法：Move Method(搬移函数)，Encapsulate Filed(封装字段)，Encapsulate Collection(封装集合)

### Data Clumps

说明：数据泥团。比如多个类中有相同的字段，几个函数签名中有相同的参数。

拟可重构方法：Extract Class(提炼类)，Introduce Parameter Object(引入参数对象)，Preserve Whole Object(保持对象完整)

### Divergent Change

说明：发散式变化。比如我们在一个类中，如果换一种数据库，需要修改这三个函数，如果出现一种新的解决办法，需要修改那四个函数。

拟可重构方法：Extract Class(提炼类)

### Duplicated Code

说明：重复代码。有重复就会有重构。

拟可重构方法：Extract Method(提炼函数)，Extract Class(提炼类)，Pull Up Method(函数上移)，Form Template Method(塑造模板函数)

### Feature Envy

说明：依赖情结。某个函数对另一个类的依赖高于其所处的类。

拟可重构方法：Move Method(搬移函数)，Move Filed(搬移字段)，Extract Method(提炼函数)

### Inappropriate Intimacy

说明：狎昵关系。如果某一个类过渡访问另一个类的private字段，这就需要将这种关系拆散。

拟可重构方法：Move Method(搬移函数)，Move Filed(搬移字段)，Change Bidirectional Association to Unidirectional(将双向关联改为单向关联)，Replace Inheritance with Delegation(以委托取代继承)，Hide Delegation(隐藏委托关系)

### Incomplete Library

说明：不完美的库类。类库中提供的函数不足以我们的程序设计。

拟可重构方法：Introduce Foreign Method(引入外加函数)，Introduce Local Extension(引入本地扩展)

### Large Class

说明：过大的类。

拟可重构方法：Extract Class(提炼类)，Extract Subclass(提炼子类)，Extract Interface(提炼接口)，Replace Data Value with Object(以对象取代数值)

### Lazy Class

说明：冗赘类。如果某一个类的功能过于简单或者根本没有必要，我们去除这个类可以使代码更加简洁明了。

拟可重构方法：Inline Class(将类内联化)，Collapse Hierarchy(折叠继承关系)

### Long Method

说明：过长的函数。程序越长越难理解，并且不易于维护。

拟可重构方法：Move Method(搬移函数)，Replace Temp with Query(以查询取代临时变量)，Replace Method with Method Object(以函数对象取代函数)，Decompose Conditional(分解条件表达式)

### Long Parameter List

说明：过长参数列。太长的参数列难以理解，可能造成前后不一致，不易使用。

拟可重构方法：Replace Parameter with Method(以函数取代参数)，Introduce Parameter Object(引入参数对象)，Preserve Whole Object(保持对象完整)

### Message Chains

说明：过度耦合的消息链。对象A请求对象B，对象B请求对象C......无穷无尽。

拟可重构方法：Hide Delegate(隐藏委托关系)

### Middle Man

说明：中间人。滥用委托的结果，某个类接口有一半的函数都委托给其他类。

拟可重构方法：Remove Middle Man(移除中间人)，Inline Method(内联函数)，Replace Delegation with Inheritance(以继承取代委托)

### Parallel Inheritance Hierarchies

说明：平行继承体系。为某一个类添加一个子类时，必须也为另一个类增加相应的子类。

拟可重构方法：Move Method(搬移函数)，Move Filed(搬移字段)

### Primitive Obsession

说明：基本类型偏执 。一个很大的类中可能含有许多基本类型数据，或者一个函数参数列表由数个基本类型构成。完全可以用对象来代替这些基本类型。比如结合数值和币种的money类。

拟可重构方法：Replace Data Value with Object(以对象取代数据值)，Extract Class(提炼类)，Introduce Parameter Object(引入参数对象)，Replace Array with Object(以对象取代数组)，Replace Type Code with Class(以类取代类型码)，Replace Type Code with Subclass(以子类取代类型码)，Replace Type Code with State/Strategy(以State/Strategy取代类型码)

### Refuse Bequest

说明：被拒绝的遗赠。子类不想继承父类的函数和数据。

拟可重构方法：Replace Interface with Delegation(以委托取代继承)

### Shotgun Surgery

说明：散弹式修改。我们希望软件发生变化时，只修改程序中的一个地方，而不是各个角落。

拟可重构方法：Move Method(搬移函数)，Move Filed(搬移字段)，Inline Class(将类内联化)

### Speculative Generality

说明：夸夸其谈未来性。我们在项目设计的时候总想着以后会有什么新的需要，所以企图以各种各式各样的钩子和特殊情况处理一些给必要的事情，这就导致了系统更加难以理解和维护。

拟可重构方法：Collapse Hierarchy(折叠继承体系)，Inline Class(将类内联化)，Remove Parameter(移除参数)，Rename Method(重命名函数)

### Switch Statements

说明：switch惊悚现身。switch语句的问题就在于重复，有重复就可以重构。

拟可重构方法：Replace Conditional with Polymorphism(以多态取代条件表达式)，Replace Type Code with Subclass(以子类取代类型码)，Replace Type Code with State/Strategy(以State/Strategy取代类型码)，Replace Parameter with Explicit Methods(以明确函数取代参数)，Introduce Null Object(引入Null对象)

### Temporary Field

说明：令人迷惑的暂时字段。某个类的实例变量只是服务于某种特殊情况，这会给阅读者造成理解障碍。

拟可重构方法：Extract Class(提炼类)，Introduce Null Object(引入Null对象)
