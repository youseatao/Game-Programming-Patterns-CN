^title 子类沙箱
^section 行为模式

## 意图

*用一系列由基类提供的操作定义子类中的行为。*

## 动机

每个孩子都梦想过变成超级英雄，但是不幸的是，高能射线在地球上很短缺。
游戏是让你扮演超级英雄最简单的方法。
因为我们的游戏设计者从来没有学会说“不”，*我们的*超级英雄游戏有成打，如果没有成百种，不同的超级能力可以选择。

我们的集合是有一个`Superpower`基类。然后由它<span name="lots">推导</span>出各种超级能力的实现。
我们在程序员队伍中分发设计文档，然后开始编程。
当我们完成时，我们就会有上百的超级能力类。

<aside name="lots">

当你发现自己有*很多*子类时，就像这个例子，那通常意味着数据驱动的方式更好。
不再用*代码*定义不同的能力，用*数据*吧。

像<a class="pattern" href="type-object.html">类型对象</a>，<a class="pattern" href="bytecode.html">字节码</a>，和<a class="gof-pattern" href="http://en.wikipedia.org/wiki/Interpreter_pattern">解释器</a>模式都能帮忙。

</aside>

我们想让玩家处于充满变化的世界中。无论他们在孩童时想象过什么能力，我们都要在游戏中表现。
这就意味着这些超能力子类需要做任何事情：播放声音，产生视觉刺激，与AI交互，创建和销毁其他游戏实体，与物理打交道。没有哪处代码是它们不会接触的。

假设我们让团队信马由缰地写超能力类。会发生什么？

 *  *会有很多冗余代码。*当超能力种类繁多，我们可以预期有很多重叠。它们很多都会用相同的方式发出视觉效果并播放声音。当你坐下来看看，冷冻光线，热能光线，芥末酱光线都很相似。如果人们实现这些的时候没有协同，那就会有很多冗余的代码和付出。

 *  *游戏引擎中的每一部分都会与这些类耦合。*没有其他信息的话，任何人都会写出直接调用子系统的代码，但子系统从来没打算直接与超能力类绑定。如果我们的渲染系统被好好组织成多个层次，只有一个能被外部的图形引擎使用，我们可以打赌超级能力代码最终会与它们中的每一个接触。

 *  *当外部代码需要改变时，有很大几率，一些随机超能力代码会损坏。*一旦我们有了不同的超能力类绑定到多种杂乱的游戏引擎部分，改变那些系统必然影响能力类。这毫无理由，因为图形，音频，UI程序员很可能不想*也*成为玩法程序员。

 *  *很难定义所有超能力遵守的不变量。*假设我们想保证超能力播放的所有音频都有正确的顺序和优先级。如果我们几百个类都直接自己调用音频引擎，就没什么好办法来完成那一点。

我们要的是给每个实现超能力的玩法程序员一系列可使用的基本单元。
你想要播放声音？这是`playSound()`函数。
你想要粒子？这是`spawnParticles()`。
我们保证了这些操作覆盖了你要做的事情，所以你不需要`#include`随机的头文件，干扰到代码库的其他部分。

我们通过定义这些操作为`Superpower`*基类*的*protected方法*。
将它们放在基类给了每个子类直接，方便的途径获取方法。
让它们为protected（有些像非虚）暗示了它们存在就是为了被子类*调用*。

一旦有了这些东西来使用，我们需要一个地方使用他们。
为了做到那点，我们定义*沙箱方法*，子类必须实现的抽象的protected方法。
有了那些，要实现一种新的能力，你需要：

1. 创建从`Superpower`继承的新类。

2. 重载`activate()`，沙箱方法。

3. 通过调用`Superpower`提供的protected方法实现主体。

我们现在可以使用这些尽可能高层次的操作来解决冗余代码问题了。
当我们看到代码在多个子类间重复，我们总可以将其打包到`Superpower`中，作为它们都可以使用的新操作。

我们通过将耦合约束到一个地方解决了耦合问题。
`Superpower`最终与不同的系统耦合，但是继承它的几百个类不会。
相反，它们*只*耦合基类。
当一个游戏系统改变时，修改`Superpower`也许是必须的，但是成打的子类不需要修改。

这个模式带来浅层但是广泛的类层次。
你的<span name="wide">继承链</span>不*深*，但是有*很多*类挂在`Superpower`上。
通过有很多直接子类的基类，我们在代码库中创造了一个支撑点。
我们投入到`Superpower`的时间和爱可以让游戏中众多类获益。

<aside name="wide">

最近，你发现很多人批评面向对象语言中的继承。
继承*是*有问题——在代码库中没有比父类子类之间的耦合更深的了——但我发现*宽阔的*继承树比起*深的*更好处理。

</aside>

## 模式

**基类**定义抽象的**沙箱方法**和几个**提供的操作**。
让它们标为protected，表明它们只为子类所使用。
每个推导出的**沙箱子类**用提供的操作实现了沙箱方法。

## 何时使用

子类沙箱模式是潜伏在代码库中简单常用的模式，哪怕是在游戏之外的地方。
如果你有一个非虚的protected方法，你可能已在用类似的东西了。
沙箱方法在以下情况适用：

*  你有一个能推导很多子类的基类。

*  基类可以提供子类需要的所有操作。

*  在子类中有行为重复，你想要更容易的在它们间分享代码。

*  你想要最小化子类和程序的其他部分的耦合。

## 记住

现在“继承”在很多编程圈子中是个坏词，一个原因是基类趋向于增加越来越多的代码
这个模式特别容易染上那点。

由于子类通过基类接触游戏的剩余部分，基类最后和子类需要的*每个*系统耦合。
当然，子类也紧密的与基类相绑定。这种蛛网耦合让你很难在不破坏什么的情况下改变基类——你得到了[brittle base class problem][]。

硬币的另一面是由于你耦合的大部分都被推到了基类，子类现在与世界的其他部分分离。
理想的，你大多数的行为都子类中。这意味着你的代码库大部分是孤立的，很容易管理。

如果你发现这个模式正把你的基类变成一锅代码糊糊，
考虑将它提供的一些操作放入分离的类中，
这样基类可以分散它的责任。<a class="pattern" href="component.html">组件</a>模式可以在这里帮上忙。

[brittle base class problem]: http://en.wikipedia.org/wiki/Fragile_base_class

## 示例代码

因为这个模式太简单了，示例代码中没有太多东西。
这不是说它没用——这个模式关键在于“意图”，而不是它实现复杂度。

我们从`Superpower`基类开始：

^code 1

`activate()`方法是沙箱方法。由于它是虚的和抽象的，子类*必须*重载它。
这让那些需要创建子类的人知道要在处理哪些工作。

其他的保护方法，`move()`，`playSound()`，和`spawnParticles()`都是提供的操作。
它们是子类在实现`activate()`要调用的。

在这个例子中，我们没有实现提供的操作，但真正的游戏在那里有真正的代码。
那些代码是`Superpower`与游戏中其他部分的耦合——`move()`也许调用物理代码，`playSound()`会与音频引擎交互，等等。
由于这都在基类的*实现*中，保证了耦合封闭在`Superpower`中。

好了，拿出我们的放射蜘蛛侠，创建一个能力。这里是一个：

<span name="jump"></span>

^code 2

<aside name="jump">

好吧，也许能*跳*不是*超级能力*，但我在这里说的是基础。

</aside>

这种能力将超级英雄弹射到天空，播放合适的声音，扬起尘土。
如果所有的超能力都这样简单——声音，粒子效果，动作的组合——那么就根本不需要这个模式了。
相反，`Superpower`有内置的`activate()`获得了声音ID，粒子类型和运动的字段。
但是这只在所有能力都以相同方式运行，只在一些数据上不同时可行。让我们精细一些：

^code 3

这里我们增加了些方法获取英雄的位置。我们的`SkyLaunch`现在可以使用它们了：

^code 4

由于我们接触了状态，现在我们的沙箱方法可以做有用有趣的控制流了。
这，还需要几个简单的`if`声明，
但你可以做<span name="data">任何</span>想做东西。
使用包含任意代码的成熟沙箱方法，天高任鸟飞了。

<aside name="data">

早先，我建议以数据驱动的方式建立超能力。
这里是你可能*不*想那么做的原因之一。
如果你的行为复杂而有命令，它更难在数据中定义。

</aside>

## 设计决策

如你所见，子类沙箱是一个“软”模式。它表述了一个基本思路，但是没有很多细节机制。
这意味着每次使用可以做出有趣的选择。这里是一些需要思考的问题。

### 应该提供什么的操作？

这是最大的问题。这深深影响了模式感觉上和实际上有多好。
在一种极端中，基类几乎不提供*任何*操作。只有一个沙箱方法。
为了实现功能，需要调用基类外部的系统。如果你这样做，很难说你在使用这个模式。

另一个极端，基类提供了<span name="include">*所有*</span>子类也许需要的操作。
子类*只*与基类耦合，不调用任何外部系统的东西。

<aside name="include">

具体的，这意味着每个子类的源文件都需要一个单一——`#include`，对它基类的。

</aside>

在这两极端之间，操作由基类提供还是向外部直接调用有很大的操作余地。
你提供的操作越多，外部系统与子类耦合越少，但是与基类耦合*越多*。
从子类中移除了耦合是通过将耦合推给基类完成的。

如果你有一堆子类与外部系统耦合的话，这是胜利。通过将耦合移到提供的操作，你将其移动到了一个地方：基类。但是你越这么做，基类就越大越难管理。

所以分界线在哪里？这里是一些首要原则：

 *  如果提供的操作只被一个或几个子类使用，你就获益不了太多。你向基类添加了复杂性，会影响所有事物，但是只有几个类收益。

    让操作与其他提供的操作保持一致也许更有价值，或者让这些特殊情况子类直接调用外部系统更加简单明了。

 *  当你调用游戏中其他地方的方法，如果方法没有修改状态就有更少的干扰。它仍然制造耦合，但是这是<span name="safe">“安全”</span>耦合，因为它没有破坏游戏中的任何东西。

    <aside name="safe">

    “安全”在这里打了引号是因为严格来说，接触数据也能造成问题。如果你的游戏是多线程的，你可能读取一个正被修改的数据。如果你不小心，就会读入错误的数据。

    另一个不愉快的情况是如果你的游戏状态是严格确定性的（为了保持玩家同步，很多在线游戏都是这样的）。如果你接触了游戏同步状态之外的东西，你会造成极糟的不确定性漏洞。

    </aside>

    另一方面，修改状态的调用会和代码库的其他方面紧密绑定，你需要三思。打包他们成基类提供的操作是个好的候选项。

 *  如果提供操作的实现是增加了向外部系统转发调用，那它就没增加太多价值。那种情况下，也许直接调用外部方法更简单。

    但是，但简单的转发也是有用的——那些方法接触了基类不想直接暴露给子类的状态。举个例子，假设`Superpower`提供这个：

    ^code 5

    它只是转发调用给`Superpower`中`soundEngine_`字段。但是，好处是保持字段封装在`Superpower`中，子类不能接触它。

### 方法应该直接提供，还是包在对象中提供？

这个模式的挑战是基类中最终加入了很多方法。
你可以将一些方法移到其他类中来缓和。基类通过返回对象提供方法。

举个例子，为了让超能力播放声音，我们可以直接将它们加到`Superpower`中：

^code 6

但是如果`Superpower`已经很大很宽泛，我们也许想要避免那样。
取而代之的是创建`SoundPlayer`类暴露那个函数：

^code 7

`Superpower`提供了对其的接触：

^code 8

将提供的操作分到辅助类可以为你做一些事情：

*  *减少了基类的方法。*在这里的例子，将三个方法变成了一个简单的获取。

*  *在辅助类中的代码通常更好管理。*核心基类像`Superpower`，不管我们的好意图，它被如此多的类依赖因而很难改变。通过将函数移到更少耦合的次要的类，我们让代码更容易使用而不破坏任何东西。

*  *减少了基类和其他系统的耦合度。*当`playSound()`直接是`Superpower`中的方法，基类与`SoundId`以及其他涉及音频代码直接绑定。将它移动到`SoundPlayer`中，减少了`Superpower`与`SoundPlayer`类的耦合，这就封装了它其他的依赖。

### 基类如何获得它需要的状态？

你的基类经常需要封装向子类隐藏的数据。
在第一个例子中，`Superpower`类提过了`spawnParticles()`方法。
如果那个实现需要一些粒子系统对象，怎么获得呢？

 *  **将它传给基类构造器：**

    最简单的解决方案是让基类将其作为构造器变量：

    ^code pass-to-ctor-base

    这安全地保证了每个超能力在构造时有粒子系统。但让我们看看子类：

    ^code pass-to-ctor-sub

    我们在这看到了问题。每个子类都需要构造器调用基类构造器并传递变量。这让子类接触了我们不想要它知道的状态。

    这也让维护头疼。如果我们后续向基类添加了状态，每个子类都需要修改并传递它。

 *  **使用两阶初始化：**

    为了避免通过构造器传递所有东西，我们可以将初始化划分为两个部分。构造器不接受任何参数，只是创建对象。然后，我们调用一个定义在基类的分离方法传入所有它需要的数据：

    ^code 9

    注意我们没有为`SkyLaunch`的构造器传入任何东西，它与`Superpower`中想要保持 私有的任何东西都不耦合。这种实现的问题，是你需要保证永远记得调用`init()`，如果忘了，你会获得处于半完成的，无法运行的超能力。

    你可以将整个过程封装到一个函数中来修复这一点，就像这样：

    <span name="friend"></span>

    ^code 10

    <aside name="friend">

    使用一点像私有构造器和友类的技巧，你可以保证`createSkylaunch()`函数是唯一能够创建能力的函数。这样，你不会忘记任何初始化。

    </aside>

 *  **让状态变为静态：**

    在先前的例子中，我们用粒子系统初始化每一个`Superpower`*实例*。这在每个能力都需要它独特状态时有意义。但是假设粒子系统是一个<a class="pattern" href="singleton.html">单例</a>，每个能力都会分享相同的状态。

    在那种情况下，我们可以让状态是基类私有而<span name="singleton">*静态*</span>的。游戏仍然要保证初始化状态，但是它只需要为整个游戏初始化`Superpower`*类*一遍，而不是为每个实例初始化一遍。

    <aside name="singleton">

    记住单例仍然有很多问题。你在很多对象中分享了状态（所有的`Superpower`实例）。粒子系统被封装了，因此它不是全局*可见的*，这很好，但它们都接触了同一对象让理解能力更难了。

    </aside>

    ^code 11

    注意这里的`init()`和`particles_`都是静态的。只要游戏早先调用过一次`Superpower::init()`，每种能力都能接触粒子系统。同时，可以调用正确的推导类构造器自由创建`Superpower`实例。

    甚至更好的，现在`particles_`是一个*静态*变量，我们不需要在每个`Superpower`中存储它，这样我们的类需要更少的内存。

 *  **使用服务定位器：**

    前一选项中，外部代码要在基类请求前压入基类需要的状态。初始化的责任落在了周围的代码。另一选项是让基类调出它需要的状态。一种实现的方法是使用<a class="pattern" href="service-locator.html">服务定位器</a>模式：

    ^code 12

    这儿，`spawnParticles()`需要粒子系统，不是外部系统*给*它，它自己从服务定位器中拿了一个。

## 参见

*  当你使用<a class="pattern" href="update-method.html">更新模式</a>时，你的更新模式通常也是沙箱方法。

*  这个模式是<a class="gof-pattern" href="http://en.wikipedia.org/wiki/Template_method_pattern">模板方法</a>的反面。两种模式中，都使用一系列受限操作实现方法。在子类沙箱中，方法在推导类中，限制操作在基类中。在模板方法中，*基*类有方法，而受限操作被*推导*类实现。

*  你也可以认为这是<a class="gof-pattern" href="http://en.wikipedia.org/wiki/Facade_Pattern">外观</a>模式的变形。那个模式将一系列不同系统藏在简化的API后。使用子类沙箱，基类起到了在子类前隐藏整个游戏引擎的外观作用。
