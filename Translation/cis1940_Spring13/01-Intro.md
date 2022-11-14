# Haskell基础
#Haskell #Translation

> 原作者建议阅读：
> -   [Learn You a Haskell for Great Good, chapter 2](http://learnyouahaskell.com/starting-out)
> -   [Real World Haskell](http://book.realworldhaskell.org/), chapters 1 and 2

## 什么是Haskell
Haskell是一门惰性的函数式编程语言，它在1980年由一个众多学者组成的委员会创建。在当时，社区中存在着太多太多的惰性函数式语言，并且由于每个人都有自己的偏好，这使得人们很难交流彼此的想法。因此，一堆人聚集在了一起，并且取当时存在着的众多函数式编程语言之长——辅之以他们的一些绝妙想法——从而设计出了Haskell这门语言。
![](https://www.seas.upenn.edu/~cis1940/spring13/images/haskell-logo-small.png)
那么，什么是Haskell呢？首先，Haskell是：

**函数式的**

![](https://www.seas.upenn.edu/~cis1940/spring13/images/function-machine.png)

目前并没有一个确切的、可以为绝大多数人所接受的对于“函数式”这个词的定义。但当我们提及Haskell是一门函数式语言时，我们通常指代的是这两个意思：

- 函数是一等公民（first-class）。它的意思是，函数可以像任何“数值”那样被使用。

- Haskell程序以评估表达式(evaluating expression)为主要表意，而不是执行命令（excuting instructions）。

这两点使得我们将要以一种完全不同的思维方式看待编程。本学期我们的大部分时间都会花在探索这种思考方式上。

**纯净的**

![](https://www.seas.upenn.edu/~cis1940/spring13/images/pure.jpg)


Haskell表达式是引用透明（referentially transparent）的。它的意思是：

- 没有改变（mutation）。Haskell的所有东西（变量、数据结构等等）都是不可变的，

- 表达式不会有任何“副作用”（side effects）。例如更新全局变量或者向屏幕打印东西。

- 对于同样的一个函数，同样的输入永远能得到同样的输出。

看到这里，您可能会觉得这一切听起来非常疯狂：怎么可能在不做任何改变与没有任何副作用的情况下把事情做完呢？其实，这正是需要您转换思维方式的地方（尤其是您在已经习惯了祈使式或面向对象式的编程范式的情况下）。当您完成思维方式的转变的时候，您便会发现引用透明会有许多好处：

- 等式推导与重构(Equational reasoning and refactoring):你总是能像你在代数课上学的那样，使用等式替换（"replace equals by equals"）
>  -  [What is meant by 'replace equals by equals'](https://stackoverflow.com/questions/30145271/what-is-meant-by-replace-equals-by-equals) 

- 利于并行：当表达式之间不会相互影响时，并行式地执行表达式是十分容易的。

- 让你在编写程序时轻松一些：

**惰性的**

![](https://www.seas.upenn.edu/~cis1940/spring13/images/relax.jpg)

在Haskell中，表达式只会在它们的结果被需要时才会执行。这是一个带来深远影响的决策，我们将会在这个学期中探讨这一点。

它带来的后果包括：

- 你可以很容易通过定义一个函数去定义一个控制结构。
- 这使得我们可以定义并去使用无限数据结构（_infinite data structures_）
- 这使我们能够使用更加组合式（compositional）的编程风格（关于这一点，可见下文的全麦编程*wholemeal programming*）
- 但糟糕的影响在于，在Haskell中评估时间与空间的使用变得更加复杂了。

**静态类型**

![](https://www.seas.upenn.edu/~cis1940/spring13/images/static.jpg)

每个Haskell表达式都具有类型，而所有的类型都会在编译时检查。存在类型错误的程序不会通过编译。

## 主题

在这门课程中，我们将主要集中学习探讨这三个主题。

**类型**

静态类型系统（static type system）可能看起来令人不快。事实上，在像C++、Java这样的语言中，静态类型系统确实令人不快。但这并不是因为静态类型系统本身是一种令人不快的存在，而是因为C++和Java的类型系统的表义性实在不强。这学期我们将深入学习Haskell的类型系统，它们有许多益处，这些益处包括：

- 帮助你阐明你的想法，理清你的程序结构
	编写Haskell程序的第一步通常是写下你所要用到的所有类型。由于Haskell的类型系统是如此富有表现力，因此这是并非无意义的一步而且将对你理清你关于程序的想法具有巨大帮助。
- 代码即文档
	借助一个富有表现力的类型系统，你不需要阅读一点相关的文档便能从函数的类型中得知这个函数的作用是什么、它的用处可能在哪。
- 可行则无误，防患于未然
	理想情况下，程序最好能在进行测试前便能将所有的错误修复。“如果一个程序跑起来了，那么它一定是没有错误的”这句话在大多数情况下都是一个笑话（在类型正确的程序里，依然可能存在逻辑上的错误），但在Haskell中，它通常是正确的。

**抽象**

在编程的世界中，你可能经常听到这样的一句话：“不要重复你自己”。这句话也通常以另一个名字“抽象原则”为我们所熟知，它的观点是任何东西（你的想法、算法与数据等等）在你的代码中应该只出现一次，如果你要多次使用，那么你应该要对它进行抽象。找到相似的代码之中的共通之处进行构造——这便是抽象的过程。

Haskell在抽象上具有得天独厚的优势。它的许多特性，例如参数化多态（parametric polymorphism）、高阶函数（high order function）与类型类（type class），便是为了解决重复进行抽象而存在的。我们课程将会投入相当大的一部分去探讨抽象。
[[The Abstraction Principle]]

**全麦编程（Wholemeal programming）**

我们将要探讨的另一个主题是：全麦编程。在这里引用一下Ralf Hinze的话进行说明：
> 函数式编程天然地擅长于全麦编程——这是由Geraint Jones提出来的词。全麦编程的意思是，你应该更笼统地去考虑程序的编写（think big）：与整个列表打交道，而不是为其中的元素绞尽脑汁;尝试得出一个解空间（solution space），而非一个独立解;
> 想象出一张图，而非单纯想象出图的一条路径。全麦编程的思考方式通常能为你遇到的问题提供新的视角。（？）投影式的编程方式很好地阐释了这一点：first solve a more general problem, then extract the interesting bits and pieces by transforming the general program into more specialised ones.”

例如，考虑像下面C++/Java式的伪码：

```
int acc = 0;
for ( int i = 0; i < lst.length; i++ ) {
  acc = acc + 3 * lst[i];
}
```

这样的代码为Richard Bird以“indexitis”为诟病：它通过一个数组的当前索引去考虑数组的迭代，这样子需要考虑太多低级的细节了。它完全可以以两个分开的操作去考虑，如此更加有效：把list中的每一个元素乘以3,而后计算它们的和。

在Haskell，我们只需要这么写：

```
sum (map (3*) lst)
```

这个学期我们将以这样的思考方式去转变我们的思维，同时我们也会探讨Haskell是如何实现这一点的。

## 供读写的Haskell文档文件（Literate Haskell）

这个文件是一个Haskell文档文件：只有用`>`与一个空格表示的行是代码，其他的都只是注释。供读写的Haskell文档文件以拓展名`.lhs`结尾，而普通的Haskell程序源代码文件以扩展名`hs`结尾。



