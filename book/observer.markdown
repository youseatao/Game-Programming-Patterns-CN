^title 观察者
^section 重访设计模式

扔块石头到电脑世界中，
不可能砸不中一个不使用<span name="devised">[MVC架构][MVC]</span>的应用，
根本在于观察者模式。
观察者模式非常普遍，Java将其放到了核心库之中（[`java.util.Observer`][java]），而C#直接将其嵌入了*语言*（[`event`][event]关键字）。

[MVC]: http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller
[java]: http://docs.oracle.com/javase/7/docs/api/java/util/Observer.html
[event]: http://msdn.microsoft.com/en-us/library/8627sbea.aspx

<aside name="devised">

就像软件中的很多东西，MVC是Smalltalkers在七十年代创造的。Lisp程序员也许声明他们在六十年代就发明了但懒得记下来。

</aside>

观察者模式是最广泛使用和最广为人知的GoF模式，
但是游戏开发世界与世隔绝，
所以这对你也许是全新的领域。
防止你还没有从修道院中走出来，
让我带你看看一个更加形象的例子。

## 成就解锁

假设我们向游戏中添加了<span name="weasel">成就系统</span>。
它存储了玩家可以完成的各种各样的成就，比如“杀死1000只猴子恶魔”，“从桥上掉下去”，或者“一命通关”。

<aside name="weasel">

<img src="images/observer-weasel-wielder.png" width="240" alt="Achievement: Weasel Wielder" />

我发誓画的这个没有第二个意思。

</aside>

有这么多行为可以解锁的成就系统，清晰地实现它是很有技巧的。
如果我们不够小心，成就系统会缠绕在代码库的每个黑暗角落。
当然，“从桥上掉落”被绑定到<span name="physics">物理引擎</span>上，
但我们并不想看到的在处理撞击代码的线性代数时，
有个对`unlockFallOffBridge()`的调用是不？

<aside name="physics">

这是个尊严问题。
任何有自尊的物理程序员都不会允许我们用像*游戏玩法*这样平常的东西玷污优美的算式。

</aside>

我们喜欢的是，照旧，让所有的关注游戏一部分的代码集成一块。
挑战在于成就在游戏的不同层面被触发。我们怎么解耦成就系统和其他部分呢？

这就是观察者模式出现的原因。
这让一块代码宣称有些有趣的事情发生了*而不必关心谁接受通知。*

举个例子，我们有物理代码处理重力，追踪哪些物体待在地表，哪些坠入深渊。
为了实现“桥上掉落”的徽章，我们可以直接把成就代码放在那里，但那就会一团糟。
相反，我们可以这样做：

^code physics-update

<span name="subtle">他做的就是</span>，“额，我不知道有谁感兴趣，但是这个东西刚刚掉下去了。做你想做的事吧。”

<aside name="subtle">

物理引擎确实决定了发送什么通知，所以它没有完全解耦。但在架构中，我们通常只是让系统*更好*，而不是*完美*。

</aside>

成就系统注册它自己，这样无论何时物理代码发送通知，成就系统都能收到。
它可以检查掉落的物体是不是我们不那么伟大的英雄，
他之前有没有做过这种新的，不愉快的与桥的经典力学遭遇。
如果满足条件，就解锁合适的成就，伴着礼花和炫光，做完所有这些都不必牵扯到物理代码。

<span name="tear">事实上</span>，我们可以改变成就的集合或者删除整个成就系统，而不必修改一行物理引擎。它仍然会发送它的通知，哪怕事实上没有东西在接收。

<aside name="tear">

当然，如果我们*暂时*移除成就，没有任何东西监听物理引擎的通知，我们也同样可以移除通知代码。但是在游戏的演进中，最好保持这样的灵活性。

</aside>

## 它如何运作

如果你还不知道如何实现这个模式，你可能可以从之前的描述中猜到，但是为了减轻你的负担，我会过一遍代码。

### 观察者

我们从那个需要知道别的对象做了什么有趣事情的吵闹类开始。
这些好打听的对象用如下接口定义：

<span name="signature"></span>

^code observer

<aside name="signature">

`onNotify()`的参数取决于你。这就是为什么是观察者*模式*，
而不是观察者“可以粘贴到游戏中的真实代码”。
典型的参数是发送通知的对象和一个将其他细节装入的“数据”参数。

如果你用通用语言或者模板编码，你可能会在这里使用它们，但是将它们根据你的特殊用况裁剪也很好。
这里我将其硬编码为接受一个游戏实体和一个描述发生了什么的枚举。

</aside>

任何实现了这个的具体类就成为了观察者。
在我们的例子中，是成就系统，所以我们会做些像这样的事：

^code achievement-observer

### 客体

接受观察的对象拥有通知方法，以GoF的说法那些对象被称为“客体”。
它有两个任务。首先，它保存默默等待它的观察者列表：

<span name="stl"></span>

^code subject-list

<aside name="stl">

在真实代码中，你会使用动态大小的集合而不是一个固定数组。
我在这里这种基本的形式是为了那些从其他语言过来而不懂C++的标准库的人们。

</aside>

重点是客体暴露了一个*公开*API来修改这个列表：

^code subject-register

这就允许了外界代码控制谁接收通知。
客体与观察者交流，但是不与它们*耦合*。
在我们的例子中，没有一行物理代码会提及成就。
现在，它仍然可以与成就系统交流。这就是这个模式的聪慧之处。

客体有一*列表*观察者而不是一个也是很重要的。
这保证了观察者不会相互竞争。
举个例子，假设我们的音频引擎也需要观察坠落事件来播放合适的音乐。
如果客体只支持一个观察者，当音频引擎注册了它自己，这就会*取消*成就系统的注册。

这意味着这两个系统需要相互交互——用一种极其糟糕的方式，
由于第二个会使第一个失效。
支持一列表的观察者保证了每个观察者都是与其他独立处理的。
它们知道的所有事情是，它是世界上唯一看着客体的东西。

客体的剩余任务就是发送通知：

<span name="concurrent"></span>

^code subject-notify

<aside name="concurrent">

注意代码假设了观察者不在它们的`onNotify()`方法中修改列表。
一个更加强健的实现会阻止或优雅地处理像这样的并发修改。

</aside>

### 可观察的物理

现在，我们只需要给物理引擎绑上钩子，这样它可以发送消息，
成就系统可以将它自己接上线来接受消息。
我们靠近之前的*设计模式*秘方，
然后<span name="event">继承</span>`Subject`：

^code physics-inherit

这让我们在`Subject`中保护的完成`notify()`。
这样推导的物理引擎类可以调用发送通知，但是外部的代码不行。
同时，`addObserver()`和`removeObserver()`是公开的，
所以任何可以接触物理引擎的东西都可以观察它。

<aside name="event">

在真实代码中，我会避免使用这里的继承。相反，我会让`Physics`*有*一个`Subject`的实例。不是观察物理引擎自己，客体会是一个单独的“下落时间”对象。
观察者可以用像下面这样的东西注册它们自己：

^code physics-event

对于我，这是“观察者”系统与“事件”系统的不同之处。
使用前者，你观察*做了有趣事情的事物*。
使用后者，你观察代表*发生的有趣的事物*。

</aside>

现在，当物理引擎做了些值得关注的东西，它调用`notify()`，就像之前的例子。
它遍历了观察者列表然后通知所有观察者。

<img src="images/observer-list.png" alt="A Subject containing a list of Observer pointers. The first two point to Achievements and Audio." />

很简单，对吧？只要一个类管理一列表指向一些接口的实例的指针。
很难相信如此直观的东西是无数程序和应用框架交流的主心骨。

观察者模式不是没有诋毁者。当我问其他程序员怎么看，他们提出了一些抱怨。
让我们看看我们可以做些什么来掌控，如果能做的话。

## 太慢了

我经常听到这一点，通常是从那些不知道模式细节的程序员那里。
他们有一种假设，只要有东西听起来像是“设计模式”一定包含了一堆类，跳转和其他创造性的方式浪费CPU循环。

观察者模式有特别坏的名声，因为他通常与一些坏名声的事物结伴出行，
比如“事件”，<span name="names">“消息”</span>，甚至“数据绑定”。
其中的一些系统*会*慢。（通常是故意的，基于很好的原因）。
他们包含了队列或者为每个通知做些动态分配。

<aside name="names">

这就是为什么我认为文档化模式很重要。
当我们为术语发狂，我们失去了简洁明确表达的能力。
你说“观察者”，别人听到了“事件”或“消息”，
因为没人花时间写下差异，也没人阅读它。

这就是我在这本书中要做的。为了涵盖我的支点，我也有一章关于事件和消息：<a href="event-queue.html" class="pattern">事件队列</a>.

</aside>

现在你看到了模式是如何真正被实现的，
你知道了并不是这样。
发送一个通知只是简单的遍历列表，然后调用一些虚方法。
是的，这比静态调用慢*一点*，但是这点消耗在大多数性能攸关的代码都是微不足道的。

我发现这个模式在热点代码路径之外有很好的应用，
那些你可以付得起动态分配消耗的地方。
除了那点，这里几乎没有天花板。
我们不必为消息分配对象。这里没有队列。这里只有一个跳转用在同步方法调用上。

### 太*快*？

事实上，你得小心观察者模式*是*同步的。
客体直接引入了他的观察者，这就意味着直到所有观察者从通知方法返回后，
它才会继续自己的工作。一个缓慢的观察者会阻塞客体。

这听起来很疯狂，但在实践中，这还不是世界末日。
这只是你得注意的事情。
UI程序员——那些基于事件的编程已经这么干了好几年了——有句经典名言：“滚出UI线程”。

如果要对事件同步响应，你需要完成然后尽可能快的返回控制权，这样UI就不会锁死。
当你有缓慢工作要做的时候，将其推到另一个线程或工作队列中去。

你需要小心观察者混合线程和锁。
如果一个观察者试图获得客体拥有的锁，你就死锁了游戏。
在更多线程的机器，你最好使用<a href="event-queue.html"
class="pattern">事件队列</a>来做异步通信。

## “它做了太多动态分配”

整个程序员部落——包括很多游戏开发者——迁移到了垃圾回收语言，
动态分配不再是以前的样子了。
但对于性能攸关的软件比如游戏，内存分配仍然重要，哪怕是在有管理的语言。
<span name="fragment">动态</span>分配需要时间，回收内存也需要时间，哪怕是自动运行的。

<aside name="fragment">

很多游戏开发者很少担忧分配而更多担忧*分页*。
当你的游戏需要不崩溃的连续运行多日来获得认证，不断增加的分页堆会阻止你发布。

<a href="object-pool.html" class="pattern">对象池</a>模式一章介绍了更多细节，以及避免它的常用技术。

</aside>

在上面的示例代码中，我使用了一个固长度的数组，因为我想尽可能保证简单。
在真实的实现中，观察者列表总是在随着观察者的添加和删除而动态地增长和消减。
这种内存搅拌吓坏了一些人。

当然，第一件需要注意的事情是只在观察者连线的时候分配内存。
*发送*一个通知不需要内存分配——只是一个方法调用。
如果你在游戏一开始就挂上了你的观察者而不乱动它们，分配的总量是很小的。

但是，如果这还是问题，我会介绍一种方式增加和删除观察者，而无需任何动态分配。

### 连接观察者

我们现在看到的所有代码中，`Subject`拥有一列指针指向观察它的`Observer`。
`Observer`类本身没有对这个列表的引用。
它只是一个纯虚接口。接口比具体有状态的类更受欢迎，所以这大体上是一件好事。

但是如果我们*确实*愿意在`Observer`中放一些状态，
我们可以通过将客体的列表包含*观察者自己*来解决分配问题。
不是客体有一集合分散的指针，观察者对象成为了链表中的一部分：

<img src="images/observer-linked.png" alt="A linked list of Observers. Each has a next_ field pointing to the next one. A Subject has a head_ pointing to the first Observer." />

为了实现这一点，我们首先要摆脱`Subject`中的数组，然后用链表头部的指针取而代之：

^code linked-subject

然后，我们在`Observer`中添加指向列表中下一个观察者的指针。

^code linked-observer

我们也可以让`Subject`成为一个友类。
客体拥有增加和删除观察者的API，但是现在列表在`Observer`内部管理。
接触这个列表最简单的办法就是让它成为友类。

注册一个新观察者就是将其连到链表中。我们使用更简单的选项，将其插到前头：

^code linked-add

另一个选项是将其添加到链表的末尾。这么做增加了一定的复杂性。
`Subject`要么遍历整个链表来找到尾部，要么保留一个分离的`tail_`指针指向最后一个节点。

将它增加在列表的头很简单，但也有另一副作用。
当我们遍历列表给每个观察者发送一个通知，
最*近*注册的观察者最*先*接到通知。
所以如果以A，B，C的顺序来注册观察者，他们会以C，B，A的顺序接到通知。

理论上，这种方式和那种没什么区别。
一种好的观察者设计原则是观察同一客体的两个观察者互相之间不应该有任何顺序相关。
如果顺序*确实*有影响，这意味着这两个观察者有一些微妙的耦合，最终会伤到你。

让我们把删除完成：

<span name="remove"></span>

^code linked-remove

<aside name="remove">

从链表移除一个节点通常需要一点丑陋的特殊情况来应对头节点，就像你在这里看到的。
这里还有一个更优雅的，用指向指针的指针的解决方案。

我在这里没有那么做，是因为我展示的半数人都迷糊了。
但这对你是一个很值得做的练习：它帮助你深入思考指针。

</aside>

因为我们有一个链表，得遍历它找到我们要删除的观察者。
如果我们使用一个普通的数组也要做相同的事。
如果我们使用*双向*链表，每个观察者都有指向前面和后面的各一个指针，
我们可以用常量时间移除一个观察者。如果这是真实代码，我会那么做。

<span name="chain">唯一</span>需要做的事情是发送通知，这和遍历列表同样简单；

^code linked-notify

<aside name="chain">

这里，我们遍历了整个链表，通知了其中每一个观察者。
这保证了所有的观察者有同样的优先级并相互独立。

我们可以这样实现，当观察者被通知时，它返回了一个标识表明客体是否应该继续遍历列表还是停止。如果你那么做，你就接近了<a href="http://en.wikipedia.org/wiki/Chain-of-responsibility_pattern" class="gof-pattern">职责链</a>模式。

</aside>

不差嘛，对吧？一个客体现在想有多少观察者就有多少观察者，不必使用动态内存。注册和取消注册就像使用简单数组一样快。但是，我们牺牲了一些小小的特性。

由于我们使用观察者对象作为链表节点，
这暗示了它只能在一个对象的观察者链表中。
换言之，一个观察者一次只能观察一个客体。
在传统的实现中，每一个客体有他独立的列表，
一个观察者同时可以在多个列表中。

你也许可以接受这一限制。
我发现*客体*有多个*观察者*要比相反更常见。
如果这*真是*一个问题，这里还有一种不必使用动态分配的解决方案。
要将这个塞到这章就太长了，但我会大致描述，让你填补空白……

### 一池链表节点

就像以前，每个客体有一个观察者的链表。
但是，这些链表节点不是观察者本身。
相反，它们是分散的小“链表<span name="intrusive">节点</span>”对象，
包含了一个指向观察者的指针和另一个指向链表下一节点的指针。

<img src="images/observer-nodes.png" alt="A linked list of nodes. Each node has an observer_ field pointing to an Observer, and a next_ field pointing to the next node in the list. A Subject's head_ field points to the first node." />

由于多个节点可以指向同一观察者，这就意味着观察者可以同时在超过一个客体列中。
我们可以同时观察多个对象了。

<aside name="intrusive">

链表有两种风味。在你学校学习的那一种，你的节点对象包含数据。
在我们之前的链接观察者例子中，就反过来了：
*数据*（在这种例子中的观察者）包含了*节点*（`next_`指针）。

后者的风格被称为“侵入式”链表，因为在对象内部使用链表侵入了对象本身的定义。
这让侵入式链表有更少的灵活性，但如我们所见，也更有效率。
他们在Linux核心这样交易有意义的地方很流行，

</aside>

避免动态分配的方法很简单：由于这些节点都是同样大小和类型，
可以预先在<a href="object-pool.html" class="pattern">对象池</a>中分配他们。
这给了你固定大小的列表节点处理，你可以随你所需使用并重用他们，
无需使用一个真正的内存分配器。

## 剩余的问题

我认为我们已经搞定了三个这个模式将人们吓阻的主要问题。
我们看到，它简单，快速，对内存管理友好。
但是这意味着你总是应该使用观察者吗？

现在，这是另一个的问题。
就像所有的设计模式，观察者模式不是万能药。
哪怕可以实现的正确高效，它也不一定是好的解决方案。
设计模式获得坏名声的原因之一就是人们将好模式运用在错问题上，最终结果更糟。

还剩两个挑战，一个是技术性的，另一个更像是可维护性层次。
我们先处理技术性的，因为它总是更好处理。

### 销毁客体和观察者

我们看到的样例代码很牢靠，但有一个严重的<span name="destruct">副作用</span>：
当删除一个客体或观察者时会发生什么？
如果你不小心的在某些观察者上面调用了`delete`，客体也许仍然持有指向它的指针。
这就是指向一片释放了的区域的悬挂指针。
当客体试图发送一个通知，好吧……就说你不会有好时光吧。

<aside name="destruct">

Not to point fingers, but I'll note that *Design Patterns* doesn't mention this
issue at all.

</aside>

删除客体更加容易，因为在大多数实现中，观察者没有对它的引用。
但是即使这样，将客体所占的位发送到内存管理系统的回收站也许会造成一些问题。
这些观察者也许仍然期待在以后收到通知，而他们不知道这再也不会发生了。
他们不再是观察者了，真的，他们只是认为他们是。

你可以用好几种方式处理这点。
最简单的就是像我做的，然后一脚踩空。
这是观察者的职责在其被删除的时候取消注册。
多数情况下，观察者*确实*知道它在观察哪个实体，
所以这通常是给它的析构器<span name="destructor">添加</span>一个`removeObserver()`调用的工作量。

<aside name="destructor">

通常在这种情况下，难点不在于如何做，而是*记得*做。

</aside>

如果在客体放弃存在，而你不想让观察者挂着，这也很好解决。
只需要让客体在它被摧毁前发送一个最终的“死亡叹息”通知。
通过这种方式，任何观察者都可以接收到，然后做那些它认为合适的<span name="mourn">行为</span>。

<aside name="mourn">

默哀，献花，挽歌……

</aside>

人们——哪怕是那些花费在大量时间在机器前，拥有让我们黯然失色才能的人——也是可靠地不可靠。
这就是为什么我们发明了电脑：它们不像我们那样经常犯错误。

更安全的回答是在每个客体销毁时，让观察者自动取消注册。
如果你在观察者基类中实现了这个逻辑，每个人不必记住就可以使用它。
这确实增加了一定的复杂度。
这意味着每个*观察者*都需要一个它在观察的*客体*列表。
你最终拥有了双向指针。

### 别担心，我有垃圾回收器

你们那些装备有垃圾回收系统的孩子现在一定很洋洋自得。
觉得你不必担心这个，因为你从来不必显式删除任何东西？再想一想！

想象一下这个：你有UI显示玩家角色情况的状态，比如健康和道具。
当玩家在屏幕上时，你为其初始化了一个对象。
当他们退出时，你直接忘掉那个对象让GC清理。

每一个角色脸上（或者其他什么地方）挨了一拳，它发送一个通知。
UI屏幕观察到了，然后更新健康槽。很好。
当玩家离开了屏幕，但你没有取消观察者的注册，会发生什么？

UI不再可见，但也不会进入垃圾回收系统，因为角色的观察者列表还保存着对它的引用。
每一次屏幕加载后，我们给那个不断增长的长列表添加一个新实例。

玩家玩游戏的整个时间，来回跑，打架，角色都会发送通知给*所有*这些屏幕。
它们不在屏幕上，但它们接受通知，浪费CPU循环在不可见的UI元素上。
如果它们做了一些像播放声音的事情，就得到了可见的错误行为。

这在通知系统中非常常见，甚至有个名字：<span name="lapsed">*失效监听者问题*</span>。
由于客体保留了对监听者的引用，你最终有僵尸UI对象徘徊在内存中。
这里上的一课是关于取消注册的纪律。

<aside name="lapsed">

哈有更加突出的标志：它有[维基条目](http://en.wikipedia.org/wiki/Lapsed_listener_problem)。

</aside>

### 发生了什么？

另一个观察者的深层次问题是它意图的直接结果。
我们使用它是因为它帮助我们放松了两块代码之间的耦合。
它让客体间接与没有静态绑定的观察者交流。

当你要让客体行为有意义时，这是很有价值的，任何悬挂件都讨厌地分散注意力。
如果你在接触物理引擎，你根本不想要你的编辑器——或者你的脑子——为一堆成就系统的东西而繁忙。

另一方面，如果你的程序没有工作，漏洞跨越了多个观察者，理清交流流程更加困难。
通过显式耦合，更容易查看哪一个方法被调用了。
这是由于耦合是静态的，分析它对于IDE是小孩子的把戏。

但是如果耦合发生在观察者列表中，唯一能告诉你谁被通知的方法是看看哪个观察者恰巧在列表中而且*处于运行中*。
不再是理清程序的*静态*交流结构，你得理清它的*命令式，动态*行为。

我对如何处理这个的指导原则很简单。
如果为了理解程序的一部分，沟通的两边*都*需要考虑，不要使用观察者模式。
使用其他更加显式的东西。

当你在某些大型程序上用黑魔法时，你会感觉这些一起处理很笨拙。
我们有很多术语给他，比如“角落分离”，“一致性和内聚性”和“模块化”，
但是总归就是“这些东西待在一起而不与其他东西待在一起。”

观察者模式是一个让这些大量不相关的代码块互相交流，而不必打包成更大块的好方法。
这在专注于一个特性或层面的单一代码块*内*不会太有用。

这就是为什么它能很好地适应我们的例子：
成就和物理是几乎完全不相干的领域，通常被不同人实现。
我们想要它们之间的交流最小化，
这样无论在哪一个上工作都不需要另一个的太多信息。

## 今日观察者

*设计模式*源于<span name="90s">1994</span>。
那时候，面向对象语言是*那个*热门范式。
每一个程序员都想要“30天学会OOP”，
中层管理员根据他们创建类的数量为他们支付工资。
工程师通过继承层次的深度评价质量。

<aside name="90s">

同一年，Ace of Base发行了不是一首而是*三首*畅销单曲，那也许能告诉你一些我们那时的品味和洞察力。

</aside>

观察者模式在那个时代中很流行，所以它有很多类就不奇怪了。
但是现代的主流程序员更加适应函数式语言。
实现一整套接口只是为了接受一个通知不再适合今日的美学了。

它感觉上去是又<span name="different">沉重</span>又死板。它*确实*又沉重又死板。
举个例子，你不能有为不同的对象使用不同通知方法的类。

<aside name="different">

这就是为什么客体经常讲它自己传给观察者。
哟与一个观察者只有单一的`onNotify()`方法，如果它观察多个客体，它需要知道哪个在调用它。

</aside>

现代的解决办法是让“观察者”只是对方法或者函数的引用。
在函数作为第一公民的语言中，特别是那些有<span name="closures">闭包</span>的，
这是一个实现观察者更加普遍的方式。

<aside name="closures">

今日，几乎*每种*语言都有闭包。C++克服了在没有垃圾回收的语言中构建闭包的挑战，
甚至是Java都一起行动起来在JDK8中引入了它们。

</aside>

举个例子，C#有“事件”嵌在语言中。
通过这些，你注册的观察者是一个“委托”，
这是那种语言的描述方法的引用的术语。
在JavaScript事件系统中，观察者*可以*是支持了特定`EventListener`协议的类，
但是他们也可以是函数。
后者是人们常用的方式。

如果现在设计观察者模式，我会让它<span name="function">基于函数</span>而不是基于类。
哪怕是在C++，我倾向于让你注册一个成员函数指针作为观察者，而不是`Observer`接口的实例。

<aside name="function">

[这里][delegate]的一篇有趣博文在C++上以某种方式实现了这一点。

[delegate]: http://molecularmusings.wordpress.com/2011/09/19/generic-type-safe-delegates-and-events-in-c/

</aside>

## 明日观察者

事件系统和其他类观察者模式在今日令人惊奇的多。
它们都是经典方法。
但是如果你用它们写一个稍微大一些的应用，你会发现一件事情。
在观察者中很多代码最后都长得一样。通常是像这样：

  1. 获知有状态改变了。

  2. 命令式的改变一些UI来反映新的状态。

这就是全部了，“哦，英雄的健康现在是7了？让我们把血条的宽度设为70像素。
过上一段时间，这会变得很沉闷。
计算机科学学术界和软件工程师已经尝试结束这种沉闷*很长*时间了。
他们用不同的方式做了很多次：“数据流编程”，“函数反射编程”等等。

如果有突破，一般局限在特定的领域中，比如音频处理或芯片设计，圣杯还没有被找到。
与此同时，一个更小野心的方式开始获得成效。很多现在的应用框架使用“数据绑定”。

不像激进的模型，数据绑定不再指望完全终结命令式代码，
也不尝试在巨大的宣言式数据图表中架构你的整个应用。
它做的只是自动改变UI元素或计算数值来反映一些值的改变。

就像其他宣言式系统，数据绑定也许适应游戏引擎的核心太慢太复杂。
但是如果说没看到它开始侵入游戏不那么性能攸关的部分，比如UI，那我会很惊讶。

与此同时，经典观察者模式仍然在那里等着我们。
是的，它不像其他的新热门技术一样填满了“函数”“反射”在名字中，
但是它超简单而且能工作。对于我来说，这通常是一个解决方案最重要的条件。
