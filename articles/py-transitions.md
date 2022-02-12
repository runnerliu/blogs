## Python 库 - Transitions

由于项目需要，使用到了Python的状态机库 [Transitions](https://github.com/tyarkoni/transitions) ，但GitHub上是英文文档，使用过后觉得还不错，遂手工翻译一下，加深理解。

### Transitions

Python 实现的轻量级、面向对象状态机，兼容Python 2.7+ 和 Python 3.0+。

#### 安装方法

方法一

```
pip install transitions
```

方法二

直接在 [GitHub](https://github.com/tyarkoni/transitions) 上clone仓库到本地，然后

```
python setup.py install
```

#### 快速入门

俗话说，接触一个新的工具库，100页的API文档、一千字的描述文档或更多的解释说明都不如一个简单的Demo，虽然这个说法我们无法验证真伪，但下面的例子会让你快速了解 `Transitions` 的基本用法

```
from transitions import Machine
import random

class NarcolepticSuperhero(object):

    # Define some states. Most of the time, narcoleptic superheroes are just like
    # everyone else. Except for...
    states = ['asleep', 'hanging out', 'hungry', 'sweaty', 'saving the world']

    def __init__(self, name):

        # No anonymous superheroes on my watch! Every narcoleptic superhero gets
        # a name. Any name at all. SleepyMan. SlumberGirl. You get the idea.
        self.name = name

        # What have we accomplished today?
        self.kittens_rescued = 0

        # Initialize the state machine
        self.machine = Machine(model=self, states=NarcolepticSuperhero.states, initial='asleep')

        # Add some transitions. We could also define these using a static list of
        # dictionaries, as we did with states above, and then pass the list to
        # the Machine initializer as the transitions= argument.

        # At some point, every superhero must rise and shine.
        self.machine.add_transition(trigger='wake_up', source='asleep', dest='hanging out')

        # Superheroes need to keep in shape.
        self.machine.add_transition('work_out', 'hanging out', 'hungry')

        # Those calories won't replenish themselves!
        self.machine.add_transition('eat', 'hungry', 'hanging out')

        # Superheroes are always on call. ALWAYS. But they're not always
        # dressed in work-appropriate clothing.
        self.machine.add_transition('distress_call', '*', 'saving the world',
                         before='change_into_super_secret_costume')

        # When they get off work, they're all sweaty and disgusting. But before
        # they do anything else, they have to meticulously log their latest
        # escapades. Because the legal department says so.
        self.machine.add_transition('complete_mission', 'saving the world', 'sweaty',
                         after='update_journal')

        # Sweat is a disorder that can be remedied with water.
        # Unless you've had a particularly long day, in which case... bed time!
        self.machine.add_transition('clean_up', 'sweaty', 'asleep', conditions=['is_exhausted'])
        self.machine.add_transition('clean_up', 'sweaty', 'hanging out')

        # Our NarcolepticSuperhero can fall asleep at pretty much any time.
        self.machine.add_transition('nap', '*', 'asleep')

    def update_journal(self):
        """ Dear Diary, today I saved Mr. Whiskers. Again. """
        self.kittens_rescued += 1

    def is_exhausted(self):
        """ Basically a coin toss. """
        return random.random() < 0.5

    def change_into_super_secret_costume(self):
        print("Beauty, eh?")
```

现在我们已经建立了一个 `NarcolepticSuperhero` 状态机，来看看具体怎么使用吧

```
>>> batman = NarcolepticSuperhero("Batman")
>>> batman.state
'asleep'

>>> batman.wake_up()
>>> batman.state
'hanging out'

>>> batman.nap()
>>> batman.state
'asleep'

>>> batman.clean_up()
MachineError: "Can't trigger event clean_up from state asleep!"

>>> batman.wake_up()
>>> batman.work_out()
>>> batman.state
'hungry'

# Batman still hasn't done anything useful...
>>> batman.kittens_rescued
0

# We now take you live to the scene of a horrific kitten entreement...
>>> batman.distress_call()
'Beauty, eh?'
>>> batman.state
'saving the world'

# Back to the crib.
>>> batman.complete_mission()
>>> batman.state
'sweaty'

>>> batman.clean_up()
>>> batman.state
'asleep'   # Too tired to shower!

# Another productive day, Alfred.
>>> batman.kittens_rescued
1
```

#### 详细文档

##### 基本初始化方式

使状态机正常运行非常简单。 假设你有一个对象 `lump` ( `Matter` 类的一个实例) ，并且你想要管理它的状态：

```
class Matter(object):
    pass

lump = Matter()
```

我们将 `lump` 绑定到状态机上来初始化一个**最小状态机**

```
from transitions import Machine
machine = Machine(model=lump, states=['solid', 'liquid', 'gas', 'plasma'], initial='solid')

# Lump now has state!
lump.state
>>> 'solid'
```

 之所以称之为**最小状态机**，因为这种状态机在技术上是可操作的，状态机初试状态为 `solid` ，但因为我们没有增加任何的转移动作(transitions：这应该是个动词，我在这翻译为转移动作)， 所以它实际上什么也没做。

下面我们来定义更多的状态和转移动作

```
# The states
states=['solid', 'liquid', 'gas', 'plasma']

# And some transitions between states. We're lazy, so we'll leave out
# the inverse phase transitions (freezing, condensation, etc.).
transitions = [
    { 'trigger': 'melt', 'source': 'solid', 'dest': 'liquid' },
    { 'trigger': 'evaporate', 'source': 'liquid', 'dest': 'gas' },
    { 'trigger': 'sublimate', 'source': 'solid', 'dest': 'gas' },
    { 'trigger': 'ionize', 'source': 'gas', 'dest': 'plasma' }
]

# Initialize
machine = Machine(lump, states=states, transitions=transitions, initial='liquid')

# Now lump maintains state...
lump.state
>>> 'liquid'

# And that state can change...
lump.evaporate()
lump.state
>>> 'gas'
lump.trigger('ionize')
lump.state
>>> 'plasma'
```

**注意**：我们为 `Matter`  实例 `lump` 绑定了 `evaporate()`  `ionize()` 等方法，每个方法触发相应的状态转换，你不必在任何地方明确定义这些方法，每个转换的名称通过绑定到模型上来传递给状态机初始化函数(`lump`)。`evaporate()`  `ionize()`另等方法都是静态来触发状态转移，如果我们想实现动态转移，可以使用`trigger` 方法。

##### 状态集

毫无疑问，状态机的核心是状态集，上述介绍中我们通过传递给 `Machine` 初始化函数字符串列表来定义有效的模型状态，但是在 `Machine` 内部，状态以 `State` 对象的形式存储。

我们可以通过多种方式初始化和跟新状态，比如：

-  传递一个给定状态名的字符串给 `Machine` 初始化函数；
-  直接初始化每个新的 `State` 对象；
-  传递具有初始化参数的字典。

以下代码片段说明了实现相同目标的几种方法

```
# Create a list of 3 states to pass to the Machine
# initializer. We can mix types; in this case, we
# pass one State, one string, and one dict.
states = [
    State(name='solid'),
    'liquid',
    { 'name': 'gas'}
    ]
machine = Machine(lump, states)

# This alternative example illustrates more explicit
# addition of states and state callbacks, but the net
# result is identical to the above.
machine = Machine(lump)
solid = State('solid')
liquid = State('liquid')
gas = State('gas')
machine.add_states([solid, liquid, gas])
```

##### 回调方法

我们可以给一个状态添加一系列的”进入“和”退出“的回调函数，还可以在初始化期间指定回调，或者稍后再添加回调。

为方便起见，每当新的状态被添加到 `Machine` 中时，方法 `on_enter_«state name»` 和 `on_exit_«state name»` 都是在 `Machine` 上（而不是 `Model` 上）上动态创建的，并且允许我们动态地添加新的进入和退出回调函数。

```
# Our old Matter class, now with  a couple of new methods we
# can trigger when entering or exit states.
class Matter(object):
    def say_hello(self): print("hello, new state!")
    def say_goodbye(self): print("goodbye, old state!")

lump = Matter()

# Same states as above, but now we give StateA an exit callback
states = [
    State(name='solid', on_exit=['say_goodbye']),
    'liquid',
    { 'name': 'gas' }
    ]

machine = Machine(lump, states=states)
machine.add_transition('sublimate', 'solid', 'gas')

# Callbacks can also be added after initialization using
# the dynamically added on_enter_ and on_exit_ methods.
# Note that the initial call to add the callback is made
# on the Machine and not on the model.
machine.on_enter_gas('say_hello')

# Test out the callbacks...
machine.set_state('solid')
lump.sublimate()
>>> 'goodbye, old state!'
>>> 'hello, new state!'
```

**注意**：首次初始化状态机时， `on_enter_«state name»` 回调函数是不会触发的。例如，我们定义了一个回调函数 `on_enter_A()` ，并给状态机初始化状态为 `initial='A'` ，`on_enter_A()` 会在下次进入状态A的时候出发，首次并不会触发。如果我们想在初始化的时候就触发 `on_enter_A()` 方法，可以创建一个虚拟的初始状态，然后调用 `to_A()` 方法，而不是 `__init__` 方法。 

除了在初始化状态机传递回调或动态添加回调，为增加代码的清晰度，还可以在模型类本身中定义回调。例如

```
class Matter(object):
    def say_hello(self): print("hello, new state!")
    def say_goodbye(self): print("goodbye, old state!")
    def on_enter_A(self): print("We've just entered state A!")

lump = Matter()
machine = Machine(lump, states=['A', 'B', 'C'])
```

现在，任何时候 `lump` 转移到状态A，类`Matter` 中的 `on_enter_A()` 方法总是会触发。

##### 检查状态

我们可以随时通过以下方式检查模型的当前状态

- 检查 `.state` 属性；
- 调用 `is_«state name»()`  方法。

如果要检索当前状态的实际状态对象，可以通过 `Machine` 实例的 `get_state()` 方法来实现

```
lump.state
>>> 'solid'
lump.is_gas()
>>> False
lump.is_solid()
>>> True
machine.get_state(lump.state).name
>>> 'solid'
```

##### 状态转换

上面的一些例子已经说明了状态转移的使用，但是在这里我们将更详细地探讨它们。

与状态一样，每个状态转换在内部用 `Transitions` 对象来表示，我们可以传递一个字典或字典列表来快速初始化状态转换集合。

如上面我们已经写过的

```
transitions = [
    { 'trigger': 'melt', 'source': 'solid', 'dest': 'liquid' },
    { 'trigger': 'evaporate', 'source': 'liquid', 'dest': 'gas' },
    { 'trigger': 'sublimate', 'source': 'solid', 'dest': 'gas' },
    { 'trigger': 'ionize', 'source': 'gas', 'dest': 'plasma' }
]
machine = Machine(model=Matter(), states=states, transitions=transitions)
```

使用字典定义转换有利于代码的清晰，但显得比较笨重。比较简单的方法是使用列表定义转换，只需确保每个列表中的元素与 `Transition` 初始化中的位置参数的顺序相同(比如 `trigger`, `source`, `destination ` 等)

我们可以这样定义转换

```
transitions = [
    ['melt', 'solid', 'liquid'],
    ['evaporate', 'liquid', 'gas'],
    ['sublimate', 'solid', 'gas'],
    ['ionize', 'gas', 'plasma']
]
```

另外，我们也可以在状态机初始化之后添加转换

```
machine = Machine(model=lump, states=states, initial='solid')
machine.add_transition('melt', source='solid', dest='liquid')
```

 `trigger` 参数定义了绑定到模型上的触发方法的名称，当回调这个方法的时候，状态机会执行相应的状态转换。

```
>>> lump.melt()
>>> lump.state
'liquid'
```

当我们调用非法的转换时，`Machine` 会抛出异常

```
>>> lump.to_gas()
>>> # This won't work because only objects in a solid state can melt
>>> lump.melt()
transitions.core.MachineError: "Can't trigger event melt from state gas!"
```

这种做法通常是可取的，因为它有助于提醒代码中的问题。但是在某些情况下，我们希望忽略这些非法转换，这时可以设置 `ignore_invalid_triggers=True` (可以全部忽略，也可忽略特定一条)

```
>>> # Globally suppress invalid trigger exceptions
>>> m = Machine(lump, states, initial='solid', ignore_invalid_triggers=True)
>>> # ...or suppress for only one group of states
>>> states = ['new_state1', 'new_state2']
>>> m.add_states(states, ignore_invalid_triggers=True)
>>> # ...or even just for a single state. Here, exceptions will only be suppressed when the current state is A.
>>> states = [State('A', ignore_invalid_triggers=True), 'B', 'C']
>>> m = Machine(lump, states)
```

如果在做状态转换之前，希望知道在当前状态下，哪些是合法的转换，可以使用 `get_triggers()` 方法

```
m.get_triggers('solid')
>>> ['melt', 'sublimate']
m.get_triggers('liquid')
>>> ['evaporate']
m.get_triggers('plasma')
>>> []
# you can also query several states at once
m.get_triggers('solid', 'liquid', 'gas', 'plasma')
>>> ['melt', 'evaporate', 'sublimate', 'ionize']
```

##### 自动转换所有状态

除了显式添加的任何转换之外，每当将状态添加到 `Machine` 实例时，都会自动创建一个 `to_«state»()` 方法，该方法无论状态机当前处于哪种状态，都会转换到目标状态。

```
lump.to_liquid()
lump.state
>>> 'liquid'
lump.to_solid()
lump.state
>>> 'solid'
```

我们可以在状态机初始化的时候设置 `auto_transitions=False` 禁用这个功能。

##### 多状态转换

给定的触发器可以附加到多个转换，其中一些可以在相同的状态下潜在地开始或结束。比如

```
machine.add_transition('transmogrify', ['solid', 'liquid', 'gas'], 'plasma')
machine.add_transition('transmogrify', 'plasma', 'solid')
# This next transition will never execute
machine.add_transition('transmogrify', 'plasma', 'gas')
```

在这种情况下，如果当前状态是 `plasma` 调用 `transmogrify()` ，会转换到 `solid` 状态，然后继续转换到 `plasma` 状态。**注意**：只有第一个匹配的转换将执行，因此，上述最后一行中定义的转换将不会做任何事情。

我们还可以通过使用 `*` 通配符来引发触发器从所有状态转换到特定状态

```
machine.add_transition('to_liquid', '*', 'liquid')
```

一个反身触发（触发器具有与源和目标相同的状态）可以使用“=”来表示目的状态，比如通过 `touch` 触发，`solid` 的目标状态为自身。如果我们希望同一个反身触发器应用到多个状态中，使用“=”是比较方便的。

```
machine.add_transition('touch', ['liquid', 'gas', 'plasma'], '=', after='change_shape')
```

通过上面的语句， `'liquid', 'gas', 'plasma'` 三个状态通过 `touch` 触发器，可以到达的目的状态分别是 `'liquid', 'gas', 'plasma'` 。

##### 有序转换

比如我们现在有这样一个需求，状态的转换是遵循自定义的序列的，如给定状态 `['A', 'B', 'C']`，我们可能需要这样的状态转换 `A → B, B → C, C → A` （没有其他的非法转换）。为了实现这个功能，`Transitions` 在 `Machine` 类中提供了 `add_ordered_transitions()` 方法

```
states = ['A', 'B', 'C']
 # See the "alternative initialization" section for an explanation of the 1st argument to init
machine = Machine(states=states, initial='A')
machine.add_ordered_transitions()
machine.next_state()
print(machine.state)
>>> 'B'
# We can also define a different order of transitions
machine = Machine(states=states, initial='A')
machine.add_ordered_transitions(['A', 'C', 'B'])
machine.next_state()
print(machine.state)
>>> 'C'
```

##### 排队转换

Transitions中的默认行为是立即处理事件。也就是说，调用绑定在 `after` 上的回调函数之前， `on_enter` 方法中的事件会预先处理。

```
def go_to_C():
    global machine
    machine.to_C()

def after_advance():
    print("I am in state B now!")

def entering_C():
    print("I am in state C now!")

states = ['A', 'B', 'C']
machine = Machine(states=states)

# we want a message when state transition to B has been completed
machine.add_transition('advance', 'A', 'B', after=after_advance)

# call transition from state B to state C
machine.on_enter_B(go_to_C)

# we also want a message when entering state C
machine.on_enter_C(entering_C)
machine.advance()
>>> 'I am in state C now!'
>>> 'I am in state B now!' # what?
```

上述状态机的执行顺序为：

```
prepare -> before -> on_enter_B -> on_enter_C -> after.
```

如果启用排队处理，则在触发下一个转换之前，转换将完成：

```
machine = Machine(states=states, queued=True)
...
machine.advance()
>>> 'I am in state B now!'
>>> 'I am in state C now!' # That's better!
```

执行顺序为：

```
prepare -> before -> on_enter_B -> queue(to_C) -> after  -> on_enter_C.
```

**注意**：当使用排队处理事件时，触发器的调用始终返回True，所以在排队执行过程中，无法确定一个包含排队回调函数的转换最终是否能成功，即使是在处理一个事件的时候。

```
machine.add_transition('jump', 'A', 'C', conditions='will_fail')
...
# queued=False
machine.jump()
>>> False
# queued=True
machine.jump()
>>> True
```

##### 条件转换

在某些情况下，我们可能需要在某些条件成立的时候去执行特定的转换，这个时候，`conditions` 就派上用场了。

```
# Our Matter class, now with a bunch of methods that return booleans.
class Matter(object):
    def is_flammable(self): return False
    def is_really_hot(self): return True

machine.add_transition('heat', 'solid', 'gas', conditions='is_flammable')
machine.add_transition('heat', 'solid', 'liquid', conditions=['is_really_hot'])
```

在上面的示例中，如果模型初始化的状态为 `solid` ，当条件 `is_flammable` 成立时，调用 `heat()` 会转换到 `gas ` 状态。同样，如果条件 `is_really_hot` 成立，调用 `heat()` 会转换到 `liquid ` 状态。

我们还可能在某些条件不成立的条件下，转换到另一状态，`Transitions` 为我们提供了 `unless` 方法。

```
machine.add_transition('heat', 'solid', 'gas', unless=['is_flammable', 'is_really_hot'])
```

这时，如果 `is_flammable(), is_really_hot()` 返回的都是 `False` ，模型调用 `heat()` 会从 `solid` 转换到 `gas ` 状态。

**注意**：条件检查方法会被动接收传递给触发函数的参数或数据对象。比如：

```
lump.heat(temp=74)
# equivalent to lump.trigger('heat', temp=74)
```

这样会将 `temp=74` 的可选参数传递给 `is_flammable()` 方法（以 `EventData` 实例的方式），参数的传递我们在后面会介绍。

##### 回调方法

我们可以将回调方法绑定到状态集和转换集上，每一个转换都包含 `before` 和 `after` 属性，这也是我们比较常用的，这两个属性包含了一系列在状态转换前后可以调用的方法。

```
class Matter(object):
    def make_hissing_noises(self): print("HISSSSSSSSSSSSSSSS")
    def disappear(self): print("where'd all the liquid go?")

transitions = [
    { 'trigger': 'melt', 'source': 'solid', 'dest': 'liquid', 'before': 'make_hissing_noises'},
    { 'trigger': 'evaporate', 'source': 'liquid', 'dest': 'gas', 'after': 'disappear' }
]

lump = Matter()
machine = Machine(lump, states, transitions=transitions, initial='solid')
lump.melt()
>>> "HISSSSSSSSSSSSSSSS"
lump.evaporate()
>>> "where'd all the liquid go?"
```

在状态转换一开始，在其他转换开始执行之前，我们也可以使用 `prepare` 回调函数：

```
class Matter(object):
    heat = False
    attempts = 0
    def count_attempts(self): self.attempts += 1
    def is_really_hot(self): return self.heat
    def heat_up(self): self.heat = random.random() < 0.25
    def stats(self): print('It took you %i attempts to melt the lump!' %self.attempts)

states=['solid', 'liquid', 'gas', 'plasma']

transitions = [
    { 'trigger': 'melt', 'source': 'solid', 'dest': 'liquid', 'prepare': ['heat_up', 'count_attempts'], 'conditions': 'is_really_hot', 'after': 'stats'},
]

lump = Matter()
machine = Machine(lump, states, transitions=transitions, initial='solid')
lump.melt()
lump.melt()
lump.melt()
lump.melt()
>>> "It took you 4 attempts to melt the lump!"
```

**注意**：除非当前状态是定义在转换集上有效的状态，否则，`prepare` 回调是不会起作用的。

如果在每一个转换执行之前或之后都需要进行一些默认操作，我们可以在状态机初始化的时候显示使用 `before_state_change`  和 `after_state_change`  属性。

```
class Matter(object):
    def make_hissing_noises(self): print("HISSSSSSSSSSSSSSSS")
    def disappear(self): print("where'd all the liquid go?")

states=['solid', 'liquid', 'gas', 'plasma']

lump = Matter()
m = Machine(lump, states, before_state_change='make_hissing_noises', after_state_change='disappear')
lump.to_gas()
>>> "HISSSSSSSSSSSSSSSS"
>>> "where'd all the liquid go?"
```

还有一些可以独立使用的关键字：

-  `prepare_event` ：通过 `prepare_event` 关键字传递给状态机的回调方法，在处理可能发生的状态转换（包括私有的 `prepare` 回调方法）之前，仅执行一次；
-  `finalize_event` ：通过 `finalize_event` 关键字传递给状态机的回调方法，不管转换是否成功，都会被执行；
-  `send_event` ：如果在状态转换过程中出现了错误，这个错误会绑定到 ` event_data` 上，我们可以使用 `send_event=True` 来检索错误 。

```
from transitions import Machine

class Matter(object):
    def raise_error(self, event): raise ValueError("Oh no")
    def prepare(self, event): print("I am ready!")
    def finalize(self, event): print("Result: ", type(event.error), event.error)

states=['solid', 'liquid', 'gas', 'plasma']

lump = Matter()
m = Machine(lump, states, prepare_event='prepare', before_state_change='raise_error',
            finalize_event='finalize', send_event=True)
try:
    lump.to_gas()
except ValueError:
    pass
print(lump.state)

>>> I am ready!
>>> Result:  <class 'ValueError'> Oh no
>>> initial
```

##### 执行命令

以下是转换集上回调方法可以执行的命令：

| 回调                            | 状态       | 备注                 |
| ----------------------------- | -------- | ------------------ |
| `machine.prepare_event`       | 源状态      | 在私有转换执行之前仅执行一次     |
| `transition.prepare`          | 源状态      | 转换一开始就执行           |
| `transition.conditions`       | 源状态      | 可使转换失败或停止          |
| `transition.unless`           | 源状态      | 可使转换失败或停止          |
| `machine.before_state_change` | 源状态      | 声明在模型上的默认回调方法      |
| `transition.before`           | 源状态      |                    |
| `state.on_exit`               | 源状态      | 声明在源状态上的回调方法       |
| `<STATE CHANGE>`              |          |                    |
| `state.on_enter`              | 目的状态     | 声明在目的状态上的回调方法      |
| `transition.after`            | 目的状态     |                    |
| `machine.after_state_change`  | 目的状态     | 声明在模型上的默认回调方法      |
| `machine.finalize_event`      | 源状态/目的状态 | 即使出现错误或异常，回调方法也会执行 |

##### 传递数据

在大多数情况下，我们需要传递给注册在状态机中的回调方法一些参数，来反应当前模型的状态和进行一些必要计算。`Transitions` 提供了两种方法。

第一种方法（默认方法）：

我们可以直接传递参数给触发器方法。

```
class Matter(object):
    def __init__(self): self.set_environment()
    def set_environment(self, temp=0, pressure=101.325):
        self.temp = temp
        self.pressure = pressure
    def print_temperature(self): print("Current temperature is %d degrees celsius." % self.temp)
    def print_pressure(self): print("Current pressure is %.2f kPa." % self.pressure)

lump = Matter()
machine = Machine(lump, ['solid', 'liquid'], initial='solid')
machine.add_transition('melt', 'solid', 'liquid', before='set_environment')

lump.melt(45)  # positional arg;
# equivalent to lump.trigger('melt', 45)
lump.print_temperature()
>>> 'Current temperature is 45 degrees celsius.'

machine.set_state('solid')  # reset state so we can melt again
lump.melt(pressure=300.23)  # keyword args also work
lump.print_pressure()
>>> 'Current pressure is 300.23 kPa.'
```

我们通过这种方法传递任何参数给触发器方法。

但是这种方式有一种局限性：状态转换触发的每一个回调方法都必须处理所有的参数。如果我们的不同回调方法需要不同的参数，这样就会导致一些不必要的麻烦。

为了解决这个问题，`Transitions` 提供了另一种方法，上文中我们提到过，参数可以以`EventData` 实例的方式传递。在`Machine` 初始化的时候设置 `send_event=True` ，这样，所有传递给非触发器的参数都以`EventData` 实例的方式传递给回调方法，为了方便我们随时访问与事件相关的源状态、模型、转换、触发等参数，`EventData` 实例会维护这些参数的内部引用。

```
class Matter(object):

    def __init__(self):
        self.temp = 0
        self.pressure = 101.325

    # Note that the sole argument is now the EventData instance.
    # This object stores positional arguments passed to the trigger method in the
    # .args property, and stores keywords arguments in the .kwargs dictionary.
    def set_environment(self, event):
        self.temp = event.kwargs.get('temp', 0)
        self.pressure = event.kwargs.get('pressure', 101.325)

    def print_pressure(self): print("Current pressure is %.2f kPa." % self.pressure)

lump = Matter()
machine = Machine(lump, ['solid', 'liquid'], send_event=True, initial='solid')
machine.add_transition('melt', 'solid', 'liquid', before='set_environment')

lump.melt(temp=45, pressure=1853.68)  # keyword args
lump.print_pressure()
>>> 'Current pressure is 1853.68 kPa.'
```

##### 其他初始化方式

在以上所有的示例中，我们将新的 `Machine` 实例绑定到一个单独的模型上（`Matter` 类的实例 `lump`），虽然这种方式可以使我们的类和回调方法比较整洁，但是我们必须能跟踪到状态机上调用的方法，并且知道模型调用的这些方法绑定了哪个状态机（比如： `lump.on_enter_StateA()` vs `machine.add_transition()`）

庆幸的是，`Transitions` 为我们提供了两种不同的初始化方式。

第一种方式，我们可以创建一个独立的状态机，完全不需要其他模型，在初始化的时候忽略模型参数。

```
machine = Machine(states=states, transitions=transitions, initial='solid')
machine.melt()
machine.state
>>> 'liquid'
```

如果我们使用这种方式初始化状态机，我们就可以直接绑定所有的触发事件和回调方法到 `Machine` 实例上。

这种方法有利于在一个地方整合所有的状态机功能，但如果考虑到状态逻辑应该包含在模型本身中，而不是单独出来，这种方式恐怕也不是最好的。

一种替代的办法是让 `Machine` 要绑定的模型继承 `Machine` 类，只要重写 `Machine` 的 `__init__()` 方法即可：

```
class Matter(Machine):
    def say_hello(self): print("hello, new state!")
    def say_goodbye(self): print("goodbye, old state!")

    def __init__(self):
        states = ['solid', 'liquid', 'gas']
        Machine.__init__(self, states=states, initial='solid')
        self.add_transition('melt', 'solid', 'liquid')

lump = Matter()
lump.state
>>> 'solid'
lump.melt()
lump.state
>>> 'liquid'
```

现在我们已经整合所有的状态机功能到 `lump` 模型中，这比将功能独立出来让人感觉更舒服。

如果我们绑定多个模型到 `Machine` 中，状态机也是可以处理的。如果想要添加模型和 `Machine` 实例本身，我们可以在初始化的时候使用 `self` 关键字（ `Machine(model=['self', model1, ...])`），也可以创建一个单独的状态机，然后通过 `machine.add_model` 动态注册模型到状态机中，如果模型不再使用的情况下，应该调用  `machine.remove_model`  来回收模型。

```
class Matter():
    pass

lump1 = Matter()
lump2 = Matter()

machine = Machine(states=states, transitions=transitions, initial='solid', add_self=False)

machine.add_model(lump1)
machine.add_model(lump2, initial='liquid')

lump1.state
>>> 'solid'
lump2.state
>>> 'liquid'

machine.remove_model([lump1, lump2])
del lump1  # lump1 is garbage collected
del lump2  # lump2 is garbage collected
```

如果在状态机初始化的时候没有提供初始状态，在添加模型的时候就必须提供这个初始状态。

```
machine = Machine(states=states, transitions=transitions, add_self=False)

machine.add_model(Matter())
>>> "MachineError: No initial state configured for machine, must specify when adding model."
machine.add_model(Matter(), initial='liquid')
```

##### 日志

`Transitions` 提供一些基本的日志功能，使用 `Python` 标准的 `logging` 模块将一些如状态转换、转换触发、条件检查等信息作为 `INFO` 级别的信息记录下来，我们可以在脚本中轻松地将日志记录配置为标准输出：

```
# Set up logging
import logging
from transitions import logger
logger.setLevel(logging.INFO)

# Business as usual
machine = Machine(states=states, transitions=transitions, initial='solid')
...
```

##### 存储/恢复状态机实例

如果想要存储或加载状态机，必须使用`Python3.3` 或较早版本的 `dill` 。

```
import dill as pickle # only required for Python 3.3 and earlier

m = Machine(states=['A', 'B', 'C'], initial='A')
m.to_B()
m.state  
>>> B

# store the machine
dump = pickle.dumps(m)

# load the Machine instance again
m2 = pickle.loads(dump)

m2.state
>>> B

m2.states.keys()
>>> ['A', 'B', 'C']
```



以上介绍了 `Transitions` 的基本使用方法，手工翻译难免有些晦涩难懂之处，敬请谅解。除了这些基本用法之外，`Transitions` 还提供了一些扩展方法，比如图表可视化机器的当前状态、用于嵌套和重用的分层状态机、并行执行的线程安全锁等，这里暂时就不介绍了，先欠着吧，等以后深入使用 `Transitions` 库的时候再去学习。

`Transitions` 英文完整版介绍请移步 [Transitions](https://github.com/tyarkoni/transitions) 