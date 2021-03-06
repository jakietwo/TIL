# Chapter 2: Callbacks

在第1章中，我们探讨了JavaScript中异步编程的术语和概念。我们的关注点是理解驱动所有“事件”(异步函数调用)的单线程(一次一个)事件循环队列。我们还探讨了并发模式解释 *同时* 运行的事件链或“进程”(任务、函数调用等)之间关系的各种方法(如果有的话)。

我们在第1章中的所有例子都使用函数作为单独的、不可分割的操作单元，在函数内部，语句以可预测的顺序运行(高于编译器级别!)，但是在函数顺序级别，事件(也称为异步函数调用)可以以各种顺序发生。

在所有这些情况下，函数都充当一个“回调”，因为它充当事件循环的目标，以便在处理队列中的项时“回调”程序。

毫无疑问，你可能已经注意到，回调是到目前为止JS程序中表示和管理异步最常见的方式。实际上，回调是语言中最基本的异步模式。

无数的JS程序，甚至是非常精妙且复杂的程序，都是在回调的基础上编写的(当然还有我们在第1章中讨论的并发交互模式)。回调函数是JavaScript的异步工作马车，它可以很好的完成任务。

除了……回调并非没有缺点。许多开发人员对 *promise* 带来的更好的异步模式感到兴奋。但是，如果你不理解抽象是什么以及为什么抽象，就不可能有效地使用任何抽象。

在这一章中，我们将深入探讨其中的一些，作为更复杂的异步模式(在本书的后续章节中进行了探讨)是必要和必要的动机。

## 延续

让我们回到第1章开始的异步回调示例，但让我稍微修改一下，以说明一点:

```js
// A
ajax( "..", function(..){
	// C
} );
// B
```

`// A`和`// B`表示程序的前半部分(即*现在*)，`// C`表示程序的后半段(即*以后*)。前半部分立刻执行，然后有一个不确定要多久的“暂停”。在后续的某个时刻，如果Ajax调用完成，那么程序将从中断处继续，并继续执行下半部分。

换句话说，回调函数封装或封装程序的*延续*。

让我们使代码更简单：

```js
// A
setTimeout( function(){
	// C
}, 1000 );
// B
```

停下来，问问你自己，你会如何描述(对其他不太了解JS工作原理的人)程序的行为方式。来吧，大声说出来。这是一个很好的练习，将帮助我的下一个观点更有意义。

大多数读者现在可能会这样想或这样说:“执行A，然后设置超时等待1000毫秒，然后一旦触发，执行c。”你的说法有多接近?

你可能已经发现了不太对，并自编辑为:“执行A，将超时设置为1000毫秒，然后执行B，超时触发后执行c。”这比第一个版本更准确。你能看出区别吗?

尽管第二种版本更加准确，但两种版本在解释这段代码时都存在不足，无法将我们的大脑与代码匹配起来，也无法将代码与JS引擎匹配起来。这种脱节既微妙又重大，是理解回调作为异步表达式和管理的缺点的核心。

只要我们以回调函数的形式引入一个延续(或者像许多程序那样引入几十个延续)，我们就允许在我们的大脑工作方式和代码操作方式之间形成分歧。任何时候这两者出现分歧，我们就会不可避免地遇到这样一个事实:我们的代码变得更难理解、推理、调试和维护。

## 顺序的头脑

我很确定你们大多数读者都听过有人说(甚至你自己也这么说过):“我是一个一心多用的人。” 试图表现得一心多用的效果包含幽默(孩子们拍头揉肚子游戏)到平淡无奇(走路时嚼口香糖)，再到彻头彻尾的危险(开车时发短信)，不一而足。

但是我们是一心多用的吗?我们真的能同时做两件有意识的、有意的事情，并且同时思考/推理这两件事吗?我们大脑功能的最高水平是否有并行多线程?

答案可能会让你大吃一惊：**可能不是** 。

这并不是我们大脑的构造。我们比很多人(尤其是a型性格的人)更喜欢独自完成任务。我们在任何时候都只能想一件事。

我说的不是我们所有无意识的、潜意识的、自动的大脑功能，比如心跳、呼吸和眼皮眨动。这些都是我们维持生命的重要任务，但我们并没有刻意地分配任何脑力。值得庆幸的是，当我们在三分钟内第15次沉迷于查看社交网络提要时，我们的大脑却在后台(线程!)处理所有这些重要的任务。

我们现在正在谈论目前处于思想最前沿的任务。对我来说，它现在正在写这本书中的文字。我是否在同一时刻做了其他更高级别的大脑功能？不，不是真的。我很快就很容易分心——在最后几段话里，我已经分心好几次了!

当我们假装同时处理多项任务时，比如试图在与朋友或家人通电话的同时打字，我们实际上最有可能做的是快速切换上下文。换句话说，我们在两个或更多任务之间快速连续地来回切换，同时以微小，快速的小块进行每项任务。我们做得如此之快，以至于外界看起来好像我们正在并行地做这些事情。我们做得如此之快，以至于在外界看来，我们似乎在并行地做这些事情。

这听起来是不是有点像异步事件并发(就像JS中发生的那种)?如果没有，回到第一章再读一遍!

事实上，简化的一种方法(即我希望在这里讨论的是，我们的大脑工作起来有点像事件循环队列。

如果你把我输入的每一个字母(或单词)看作一个单独的异步事件，仅在这句话中，我的大脑就有几十个机会被其他事件打断，比如我的感官，甚至只是我的随机想法。

我不会一有机会就被打断，被拉到另一个“过程”(谢天谢地——否则这本书永远也写不出来!)但这种情况经常发生，我觉得自己的大脑几乎不断地切换到各种不同的环境中(也就是“过程”)。这很像JS引擎的感觉。

### 执行和计划

好吧，所以我们的大脑可以被认为是在单线程事件循环队列中运行，就像JS引擎一样。这听起来很不错。

但我们需要在分析中更加细致入微。我们如何规划各种任务，以及我们的大脑如何实际执行这些任务，这两者之间存在着巨大而明显的差异。

再次，回到写这个文本作为我的比喻。我这里粗略的心理大纲计划是继续写啊写的，依次通过我在脑海中排列的一系列要点。我不打算在写作中有任何中断或非线性活动。但是，我的大脑一直在转换。

尽管在操作层面我们的大脑是异步的，但我们似乎以顺序，同步的方式规划任务。 “我需要去商店，然后买些牛奶，然后放弃干洗。”

你会注意到，这种更高层次的思考(计划)在它的形成过程中似乎不是很异步的。事实上，我们很少会刻意只考虑事件本身。相反，我们仔细地计划事情，按顺序(A然后B然后C)，我们假设在某种程度上存在一种时间阻塞，迫使B等待A，而C等待B。

当开发人员编写代码时，他们正计划要执行一系列操作。如果他们擅长做开发人员，他们会 **仔细规划** 。我需要把`z`设为`x`的值，然后把`x`设为`y`的值，以此类推。

当我们逐条语句地编写同步代码时，它的工作原理很像我们的待办事项列表:

```js
// swap `x` and `y` (via temp variable `z`)
z = x;
x = y;
y = z;
```

这三个赋值语句是同步的，因此`x = y`等待`z = x`完成，而`y = z`依次等待`x = y`完成。另一种说法是，这三个陈述在时间上必然以某种顺序执行，一个接一个地执行。值得庆幸的是，我们不需要为任何异步事件的细节而烦恼。如果我们这样做了，代码很快就会变得非常复杂!

因此，如果同步大脑规划很好地映射到同步代码语句，那么我们的大脑在规划异步代码方面做得如何呢？

事实证明，我们如何在代码中表达异步(使用回调)根本不能很好地映射到同步的大脑规划行为。

你真的能想象有这样一条思路来规划你要做的事情吗?

> “我需要去趟商店，但是我确信在路上会接到一个电话，‘嗨，妈妈’，当她开始讲话的时候，我会在GPS上搜索商店的位置，但那会花几分钟加载，所以我把收音机音量调小以便听到妈妈讲话，然后我发现我忘了穿夹克而且外面很冷，但没关系，继续开车并和妈妈说话，然后安全带警报提醒我要系好，于是‘是的，妈，我系着安全带呢，我总是系着安全带！’。啊，GPS终于得到方向了，现在……”

尽管这听起来很荒谬，但它确实是我们大脑在功能层面运作的方式。请记住，这不是多任务处理，只是快速上下文切换。

我们作为开发人员难以编写异步事件代码的原因，特别是当我们所有人都有回调时，我们大多数人认为思维/计划的流程是不自然的。

我们一步一步地思考，但是一旦我们从同步转向异步，代码中可用的工具（回调）就不会逐步表达出来。

就是为什么用回调准确地编写和推理异步JS代码是如此困难：因为它不是我们的大脑规划的工作方式。

**注意：** 唯一比不知道为什么代码不好用更糟糕的是，从一开始就不知道为什么代码好用！这是一种经典的“纸牌屋”心理：“它好用，但不知为什，所以大家都别碰！”你可能听说过，“他人即地狱”（萨特），而程序员们模仿这种说法，“他人的代码即地狱”。我相信：“不明白我自己的代码才是地狱。”而回调正是肇事者之一。

### 嵌套/链式的回调

考虑下面的代码：

```js
listen( "click", function handler(evt){
	setTimeout( function request(){
		ajax( "http://some.url.1", function response(text){
			if (text == "hello") {
				handler();
			}
			else if (text == "world") {
				request();
			}
		} );
	}, 500) ;
} );
```

这样的代码很有可能是你能识别出来的。我们有一个嵌套在一起的三个函数链，每个函数表示异步系列中的一个步骤(任务，“流程”)。

这种代码通常被称为“回调地狱”，有时也被称为“厄运金字塔”（由于嵌套缩进而面向侧面的三角形）。

这种代码通常被称为“回调地狱”(callback hell)，有时也被称为“末日金字塔”(pyramid of doom)(由于嵌套缩进，其侧面是三角形)。

但是“回调地狱”实际上与嵌套/缩进几乎没有任何关系。这是一个更深层次的问题。我们将在本章的其余部分继续学习，了解如何以及为什么这样做。

首先，我们等待“click”事件，然后等待计时器触发，然后等待Ajax响应返回，此时它可能再次执行所有操作。

乍一看，这段代码似乎可以很自然地将异步联想到顺序大脑规划。

首先（*现在*），我们：

```js
listen( "..", function handler(..){
	// ..
} );
```

*然后* ，我们：

```js
setTimeout( function request(..){
	// ..
}, 500) ;
```

*再然后*，我们：

```js
ajax( "..", function response(..){
	// ..
} );
```

最后，我们：

```js
if ( .. ) {
	// ..
}
else ..
```

但是用这种方式线性地解释这段代码有几个问题。

首先，我们的步骤在随后的几行(1、2、3和4…)上执行，这是个巧合。在真正的异步 JS程序中，经常会有很多杂音把事情弄乱，当我们从一个函数跳到另一个函数时，我们必须巧妙地在大脑中处理这些杂音。理解这种回调代码中的异步流并非不可能，但即使经过大量练习，它仍然不那么自然或不容易。

但是，还有更深层次的错误，这在代码示例中并不明显。让我再编一个场景(伪代码)来说明:

```js
doA( function(){
	doB();

	doC( function(){
		doD();
	} )

	doE();
} );

doF();
```

虽然你们中经验丰富的人会在这里正确地确定操作的真实顺序，但我认为乍一看它不仅有点令人困惑，而是需要一些协调一致的心理周期来达到。操作将按以下顺序进行：

- `doA()`
- `doF()`
- `doB()`
- `doC()`
- `doE()`
- `doD()`

你第一次看代码的时候就看对了吗?

好吧，你们有些人认为我在函数命名上不公平，故意误导你们。我发誓我只是按照自顶向下的外观顺序命名。但让我再试一把:

```js
doA( function(){
	doC();

	doD( function(){
		doF();
	} )

	doE();
} );

doB();

```

现在，我按照实际执行的顺序按字母顺序命名。但我仍然敢打赌，即使有这种经验情况下，对很多读者来说，跟踪`A -> B -> C -> D -> E -> F`顺序也不是很自然的。当然，你的眼睛会在代码片段中上下跳动，对吧？

但即使这一切对你来说很自然，还有一个可能造成严重破坏的危险。你能发现它是什么吗?

如果`doA(..)`或`doD(..)`实际上不是异步的呢?哦，现在顺序不一样了。如果它们都是同步的(可能只是有时，这取决于当时程序的条件)，现在的顺序是`A -> C -> D -> F -> E -> B`。

你刚才在背景中隐约听到的声音是数千名JS开发人员的叹息声，他们只是面对面地交流了一下。

嵌套问题吗?这就是跟踪异步流如此困难的原因吗?当然，这也是原因之一。

但是，让我在不使用嵌套的情况下重写先前嵌套的事件/超时/ Ajax示例：

```js
listen( "click", handler );

function handler() {
	setTimeout( request, 500 );
}

function request(){
	ajax( "http://some.url.1", response );
}

function response(text){
	if (text == "hello") {
		handler();
	}
	else if (text == "world") {
		request();
	}
}
```

代码的这种表达方式不像以前的形式那样容易识别嵌套/缩进问题，但是它同样容易受到“回调地狱”的影响。为什么?

当我们对这段代码进行线性(顺序)推理时，我们必须从一个函数跳到下一个函数，再跳到下一个函数，然后在代码库中来回跳转，以“查看”序列流。请记住，这是最优情况下的简化代码。我们都知道，真正的异步 JS程序代码库通常非常混乱，这使得这种数量级的推理更加困难。

另一件需要注意的事情是:要将步骤2、3和4链接在一起，以便它们依次发生，仅使用可视性回调就可以将步骤2硬编码为步骤1，步骤3编码为步骤2，步骤4编码为步骤3，以此类推。硬编码不一定是件坏事，如果它确实是一个固定的条件，那么步骤2应该总是导致步骤3。

但硬编码肯定会使代码更加脆弱，因为它没有考虑到可能导致步骤进展偏差的任何错误。例如，如果步骤2失败，则步骤3永远不会到达，步骤2也不会重试，或者转移到备用错误处理流程，依此类推。

所有这些问题都可以手动硬编码到每个步骤中，但是这些代码总是重复的，在程序的其他步骤或其他异步流中不可重用。

尽管我们的大脑可能会以一种顺序的方式计划一系列的任务(这个，然后这个，然后这个)，我们大脑运作的偶然性使得恢复/重试/分流的流量控制几乎毫不费力。如果你外出办事，发现自己把购物清单忘在家里了，这并不是因为你没有提前计划好。你的大脑很容易绕过这个小问题:你回家，拿着购物单，然后直接回到商店。

但是手工硬编码回调的脆弱本质(即使使用硬编码错误处理)常常远没有那么优雅。一旦你最终指定(也称为预先计划)所有各种可能性/路径，代码就会变得非常复杂，以至于很难维护或更新它。

**这** 就是“回调地狱”的全部内容!嵌套/缩进基本上是一种旁门左道，转移了人们的注意力。

如果这些还不够，我们甚至还没有讨论当两个或多个回调延续链同时发生时，或者当第三步分支为带有门或锁的“并行”回调时，或者……天哪，我的大脑受伤了，你的呢?

你是否注意到，我们的顺序的、阻塞的大脑规划行为不能很好地映射到面向回调的异步代码?这是关于回调的第一个主要缺陷:它们在代码中表示异步性，我们的大脑必须通过这种方式来保持同步(双关语!)

## 信任问题

顺序大脑的规划和回调驱动的异步JS代码之间的不匹配只是回调问题的一部分。还有更深层的问题需要关注。

让我们再次回顾回调函数作为程序的延续(也就是下半部分)的概念:

```js
// A
ajax( "..", function(..){
	// C
} );
// B
```

`// A`和`// B` *现在*发生，是在主JS程序的直接控制下发生。但是`// C`被推迟到稍后发生，并且由另一方控制——在本例中是`ajax(..)`函数。从一个基本的意义上说，这种控制权的交接不会经常给程序带来很多问题。

但是不要被它的不常见所愚弄，这种控制开关并不是什么大问题。事实上，这是回调驱动设计中最糟糕(但也是最微妙)的问题之一。它的核心思想是，有时候`ajax(..)`，将回调延续传递给)的“party”不是你编写的函数，也不是你直接控制的函数。很多时候，它是由第三方提供的实用程序。

我们将此称为“控制反转”，即你参与程序并将其执行控制权交给另一个第三方。在你的代码和第三方实用程序之间存在一个不言而喻的“契约”——一组你希望维护的东西。

### 五个回调的故事

为什么这是一件大事，可能并不十分明显。让我来构建一个夸张的场景来说明信任的危害。

假设你是一个开发人员，负责为一个售价昂贵的电视做站点构建电子商务结帐系统。你已经完成了结帐系统的所有各种页面。在最后一页，当用户点击“确认”购买电视时，你需要调用第三方功能(比如一些分析跟踪公司提供的功能)，以便跟踪销售情况。

你注意到，他们提供了看起来像异步跟踪实用程序的东西，这可能是出于性能最佳实践的考虑，这意味着你需要传入一个回调函数。在传递的这个延续中，你将获得最终代码，用于向客户的信用卡收费并显示感谢页面。

此代码可能如下所示：

```js
analytics.trackPurchase( purchaseData, function(){
	chargeCreditCard();
	displayThankyouPage();
} );
```

足够简单,对吗?编写代码，测试它，一切正常，然后部署到生产环境中。每个人都很快乐!

六个月过去没有问题。你几乎忘了你甚至写了那段代码。一天早上，你在上班前的咖啡馆里，漫不经心地喝着拿铁咖啡，突然接到老板打来的电话，他很惊慌，坚持让你把咖啡放下，然后立刻赶去上班。

当你到达的时候，你发现一位知名客户的同一台电视机的信用卡被扣了五次钱，他很不高兴，这是可以理解的。客服已经发出道歉并处理退款。但是你的老板要求知道这是怎么发生的。“难道我们没有像这样的测试吗!?”

你甚至不记得你写的代码。但你会重新开始，试图找出可能出了什么问题。

在深入研究了一些日志之后，你得出的结论是，惟一的解释是由于某种原因，分析工具以某种方式调用了五次回调，而不是一次。他们的文档中没有任何内容提到这一点。

沮丧之下，你联系了客户支持，他们当然和你一样惊讶。他们同意将其升级到他们的开发人员去处理，并承诺会回复你。第二天，你会收到一封很长的电子邮件，解释他们发现了什么，然后你立刻把它转发给你的老板。

显然，分析公司的开发人员一直在研究一些实验性代码，这些代码在特定条件下每秒重试一次提供的回调，每次五秒钟，然后超时失败。他们从来没有想过要把它投入生产，但不知何故他们做到了，他们感到非常尴尬和抱歉。他们会详细说明他们是如何识别故障的，以及他们将如何确保此类故障不会再次发生。丫丫个呸的。

那么接下来呢？

你和你的老板谈过了，但是他对目前的情况感到不太满意。他坚持认为(你也不情愿地同意了)你不能再信任他们，并且你需要找出如何再次保护签出代码免受此类漏洞的攻击。

经过一些修补，你实现了一些简单的特殊代码，如下所示，团队似乎很满意：

```js
var tracked = false;

analytics.trackPurchase( purchaseData, function(){
	if (!tracked) {
		tracked = true;
		chargeCreditCard();
		displayThankyouPage();
	}
} );
```

**注意：** 这在第1章中看起来应该很熟悉，因为如果回调碰巧有多个并发调用，我们实际上是在创建一个锁来处理。

但是随后你的一个QA工程师问道:“如果他们从来不调用回调会发生什么?”哦。你们都没想过。

你开始跟踪漏洞，并考虑所有可能出现的问题，他们调用你的回调。下面是你大致列出的分析工具可能出现的问题:

- 过早调用回调（在跟踪之前）
- 调用回调太晚（或从不）
- 调用回调太少或太多次（就像你遇到的问题！）
- 无法将任何必要的环境/参数传递给回调
- 吞下可能发生的任何错误/异常
- 。。。

这应该是一份令人不安的清单，因为确实如此。你可能慢慢地开始意识到，你将不得不在传递给你不能信任的实用程序的 **每个回调中** 创建大量的特别逻辑。

现在，你更全面地了解了“回调地狱”是多么的地狱。

### 不仅是别的代码

你们中的一些人可能会怀疑这是否像我说的那么重要。也许你根本没有与真正的第三方实用程序进行过多的交互。也许你使用版本化的api或自托管这样的库，这样就不能从你的底层更改它的行为。

因此，考虑一下这个问题: 你甚至能够真正信任理论上控制的实用程序(在自己的代码库中)吗?

这样想:我们大多数人都同意，至少在某种程度上，我们应该构建自己的内部函数，对输入参数进行一些防御性检查，以减少/防止意外问题。

过分信任输入:

```js
function addNumbers(x,y) {
	// + is overloaded with coercion to also be
	// string concatenation, so this operation
	// isn't strictly safe depending on what's
	// passed in.
	return x + y;
}

addNumbers( 21, 21 );	// 42
addNumbers( 21, "21" );	// "2121"
```

对于不信任的：

```js
function addNumbers(x,y) {
	// ensure numerical input
	if (typeof x != "number" || typeof y != "number") {
		throw Error( "Bad parameters" );
	}

	// if we get here, + will safely do numeric addition
	return x + y;
}

addNumbers( 21, 21 );	// 42
addNumbers( 21, "21" );	// Error: "Bad parameters"
```

 或者也许仍然安全但更友好:

```js
function addNumbers(x,y) {
	// ensure numerical input
	x = Number( x );
	y = Number( y );

	// + will safely do numeric addition
	return x + y;
}

addNumbers( 21, 21 );	// 42
addNumbers( 21, "21" );	// 42
```

无论如何，这些类型的检查/规范化在函数输入上是相当常见的，即使对于理论上完全信任的代码也是如此。以一种粗略的方式，这就像编程中“信任但验证”的地缘政治原则。

那么，难道我们不应该对异步函数回调的组合做同样的事情吗?不仅是使用真正的外部代码，而且即使使用我们知道通常是“在我们自己控制下”的代码也是如此。我们当然应该。

但是回调并不能提供任何帮助。我们必须自己构建所有这些机制，并且它通常会成为大量的样板/开销，我们对每个异步回调都要重复这些样板/开销。

回调最麻烦的问题是控制反转，导致所有这些信任线完全崩溃。

如果你的代码使用回调，尤其是但不完全使用第三方实用程序，而且你还没有为所有这些控制反转信任问题应用某种缓解逻辑，那么你的代码现在就有bug，即使它们可能还没有影响你。潜在的bug仍然是bug。

地狱。

## 试图保存回调

有几种回调设计的变体试图解决我们刚刚看到的一些(不是全部!)信任问题。这是一项勇敢但注定要失败的努力，目的是避免回调模式自身崩溃。

例如，对于更优雅的错误处理，一些API设计提供了分割回调(一个用于成功通知，一个用于错误通知):

```js
function success(data) {
	console.log( data );
}

function failure(err) {
	console.error( err );
}

ajax( "http://some.url.1", success, failure );
```

在这种设计的api中，failure()错误处理程序通常是可选的，如果没有提供该处理程序，则假定你是希望将错误咽下去。啊哈。

**注意：** 这种拆分回调设计是ES6 Promise API使用的。我们将在下一章更详细地介绍ES6 Promises。

另一种常见的回调模式称为“error-first style”(有时称为“Node style”，因为它也是几乎所有Node.js api中使用的约定)，其中单个回调的第一个参数保留给一个error对象(如果有的话)。如果成功，这个参数将是空的/假的(任何后续的参数将是成功数据)，但是如果一个错误结果被发出信号，第一个参数是set/truthy(通常没有传递任何其他参数):

```js
function response(err,data) {
	// error?
	if (err) {
		console.error( err );
	}
	// otherwise, assume success
	else {
		console.log( data );
	}
}

ajax( "http://some.url.1", response );

```

在这两种情况下，都应该注意以下几点。

首先，它并没有像看上去那样真正解决大多数信任问题。回调既不能防止也不能过滤不必要的重复调用。而且，现在情况更糟了，因为你可能同时获得成功和错误信号，或者两者都没有，而且你仍然必须围绕这两种情况编写代码。

另外，不要忽略这样一个事实，虽然这是一个可以使用的标准模式，但它肯定更冗长，而且没有太多重用，所以你将会厌倦为应用程序中的每个回调输出所有这些内容。



那么从来没有人给你打电话的信任问题呢?如果这是一个问题(可能应该是!)，你可能需要设置一个超时来取消事件。你可以做一个实用工具(只显示概念验证)来帮助你:

```js
function timeoutify(fn,delay) {
	var intv = setTimeout( function(){
			intv = null;
			fn( new Error( "Timeout!" ) );
		}, delay )
	;

	return function() {
		// timeout hasn't happened yet?
		if (intv) {
			clearTimeout( intv );
			fn.apply( this, [ null ].concat( [].slice.call( arguments ) ) );
		}
	};
}
```

以下是你使用它的方式：

```js
// using "error-first style" callback design
function foo(err,data) {
	if (err) {
		console.error( err );
	}
	else {
		console.log( data );
	}
}

ajax( "http://some.url.1", timeoutify( foo, 500 ) );
```

另一个信任问题是被调用的“过早”。在应用程序规范上讲，这可能涉及在某些重要的任务完成之前被调用。但更一般地，在那些即可以 *现在*（同步地），也可以在 *稍后*（异步地）调用你提供的回调的工具中这个问题更明显。

这种围绕着同步或异步行为的不确定性，几乎总是导致非常难追踪的Bug。在某些圈子中，一个名叫Zalgo的可以导致人精神错乱的虚构怪物被用来描述这种同步/异步的噩梦。经常能听到人们喊“别放出Zalgo！”，而且它引出了一个非常响亮的建议：总是异步地调用回调，即便它是“立即”在事件轮询的下一个迭代中，这样所有的回调都是可预见的异步。

**注意：** 更多关于Zalgo的信息，参见Oren Golan的“Don't Release Zalgo!（不要释放Zalgo！）”（ <https://github.com/oren/oren.github.io/blob/master/posts/zalgo.md> ）和Isaac Z. Schlueter的“Designing APIs for Asynchrony（异步API设计）”（ <http://blog.izs.me/post/59142742143/designing-apis-for-asynchrony> ）。

考虑下面的代码：

```js
function result(data) {
	console.log( a );
}

var a = 0;

ajax( "..pre-cached-url..", result );
a++;
```

这段代码是打印`0`（同步回调调用）还是打印`1`（异步回调调用）？这……要看情况。

你可以看到Zalgo的不可预见性能有多快地威胁你的JS程序。所以听起来傻呼呼的“别放出Zalgo”实际上是一个不可思议地常见且实在的建议——总是保持异步。

如果你不知道当前的API是否会总是异步地执行呢？你可以制造一个像`asyncify(..)`这样的工具：

```js
function asyncify(fn) {
	var orig_fn = fn,
		intv = setTimeout( function(){
			intv = null;
			if (fn) fn();
		}, 0 )
	;

	fn = null;

	return function() {
		// 触发太快，在`intv`计时器触发来
		// 表示异步回合已经过去之前？
		if (intv) {
			fn = orig_fn.bind.apply(
				orig_fn,
				// 将包装函数的`this`加入`bind(..)`调用的
				// 参数，同时currying其他所有的传入参数
				[this].concat( [].slice.call( arguments ) )
			);
		}
		// 已经是异步
		else {
			// 调用原版的函数
			orig_fn.apply( this, arguments );
		}
	};
}
```

你像这样使用`asyncify(..)`:

```js
function result(data) {
	console.log( a );
}

var a = 0;

ajax( "..pre-cached-url..", asyncify( result ) );
a++;
```

不管Ajax请求是由于存在于缓存中而解析为立即调用回调，还是它必须走过网线去取得数据而异步地稍后完成，这段代码总是输出`1`而不是`0`——`result(..)`总是被异步地调用，这意味着`a++`有机会在`result(..)`之前运行。

噢耶，又一个信任问题被“解决了”！但它很低效，而且又有更多臃肿的模板代码让你的项目变得沉重。

这只是关于回调一遍又一遍地发生的故事。它们几乎可以做任何你想做的事，但你不得不努力工作来达到目的，而且大多数时候这种努力比你应当在推理这样的代码上所付出的多得多。

你可能发现自己希望有一些内建的API或语言机制来解决这些问题。终于ES6带着一个伟大的答案到来了，所以继续读下去！

## Review

回调是JS中异步的基础单位。但是随着JS的成熟，它们对于异步编程的演化趋势来讲显得不够。

首先，我们的大脑用顺序的，阻塞的，单线程的语义方式规划事情，但是回调使用非线性，非顺序的方式表达异步流程，这使我们正确推理这样的代码变得非常困难。不好推理的代码是导致不好的Bug的不好的代码。

我们需要一个种方法，以更同步化，顺序化，阻塞的方式来表达异步，正如我们的大脑那样。

第二，而且是更重要的，回调遭受着 *控制反转* 的蹂躏，它们隐含地将控制权交给第三方（通常第三方工具不受你控制！）来调用你程序的 *延续*。这种控制权的转移使我们得到一张信任问题的令人不安的列表，比如回调是否会比我们期望的被调用更多次。

制造特殊的逻辑来解决这些信任问题是可能的，但是它比它应有的难度高多了，还会产生更笨重和更难维护的代码，而且在bug实际咬到你的时候代码会显得在这些危险上被保护的不够。

我们需要一个 **所有这些信任问题** 的一般化解决方案。一个可以被所有我们制造的回调复用，而且没有多余的模板代码负担的方案。

我们需要比回调更好的东西。目前为止它们做的不错，但JavaScript的 *未来* 要求更精巧和强大的异步模式。本书的后续章节将会深入这些新兴的发展变化。