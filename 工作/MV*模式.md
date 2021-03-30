## GUI程序所面临的问题

在开发图形界面应用程序的时候，会把管理用户界面的层次称为View，应用程序的数据为Model（这里的Model指的是Domain Model，这个应用程序对需要解决的问题的数据抽象，不包含应用的状态，可以简单理解为对象），Model提供数据的接口，执行相应的业务逻辑。

![image-20210202205908495](https://raw.githubusercontent.com/silence/blog/assets/assets/20210202205908.png)

有了View和Model的分层，那么问题就来了：**View如何同步Model的变更，View和Model之间如何粘合在一起。**

带着这个问题开始探索MV\*模式，*会发现这些模式之间的差异可以归纳为对这个问题处理的方式的不同。而几乎所有的MV模式都是经典的**Smalltalk-80 MVC**的修改版*

## Smalltalk-80 MVC

### 历史背景

>早在上个世纪70年代，美国的施乐公司（Xerox）的工程师研发了Smalltalk编程语言，并且开始用它编写图形界面的应用程序。而在Smalltalk-80这个版本的时候，一位叫Trygve Reenskaug的工程师设计了MVC图形应用程序的架构模式，极大地降低了图形应用程序的管理难度。而在四人帮（GoF）的设计模式当中并没有把MVC当做是设计模式，而仅仅是把它看成解决问题的一些类的集合。Smalltalk-80 MVC和GoF描述的MVC是最经典的MVC模式。

### MVC的调用关系

用户的对View操作以后，View捕获到这个操作，会把处理的权利交移给Controller；Controller会对来自View数据进行预处理、决定调用哪个Model的接口；然后由Model执行相关的业务逻辑；当Model变更了以后，会通过**观察者模式（Observer Pattern）通知View; View通过观察者模式**收到Model变更的消息以后，会向Model请求最新的数据，然后重新更新界面。

![image-20210203111824556](https://raw.githubusercontent.com/silence/blog/assets/assets/20210203111824.png)

需要特别注意

1. View是把控制权交移给Controller, Controller执行应用程序相关的应用逻辑（对来自View数据进行预处理、决定调用哪个Model的接口等等）。
2. Controller操作Model，Model执行业务逻辑对数据进行处理。但不会直接操作View，可以说它是对View无知的。
3. View和Model的同步消息是通过观察者模式进行，而同步操作是由View自己请求Model的数据然后对视图进行更新。

### MVC的优缺点

优点：

1. 把业务逻辑和展示逻辑分离，模块化程度高。且当应用逻辑需要变更的时候，不需要变更业务逻辑和展示逻辑，只需要Controller换成另外一个Controller就行了（Swappable Controller）
2. 观察者模式可以做到多视图同时更新。

缺点：

1. Controller测试困难。因为视图同步操作是由View自己执行，而View只能在有UI的环境下运行。在没有UI环境下对Controller进行单元测试的时候，应用逻辑正确性是无法验证的：Model更新的时候，无法对View的更新操作进行断言。
2. View无法组件化。View是强依赖特定的Model的，如果需要把这个View抽出来作为一个另外一个应用程序可复用的组件就困难了。因为不同程序的Domain Model是不同的。

## MVC Model 2

在Web服务端开发的时候也会接触到MVC模式，而这种MVC模式不能严格称为MVC模式。经典的MVC模式只是解决客户端图形界面应用程序的问题，而对服务端无效。服务端的MVC模式有自己特定的名字：MVC Model 2，Model 2 客户端服务端的交互模式如下：![image-20210203113559791](https://raw.githubusercontent.com/silence/blog/assets/assets/20210203113559.png)

服务端接收到来自客户端的请求，服务端通过路由规则把这个请求交由给特定的Controller进行处理，Controller执行相应的应用逻辑，对Model进行操作，Model执行业务逻辑以后；然后用数据去渲染特定的模板，返回给客户端。

因为HTTP协议是单工协议并且是无状态的，服务器无法直接给客户端推送数据。除非客户端再次发起请求，否则服务器端的Model的变更就无法告知客户端。所以可以看到经典的Smalltalk-80 MVC中Model通过观察者模式告知View更新这一环被无情地打破，不能称为严格的MVC。

后来这种模式几乎被应用在所有语言的Web开发框架当中。PHP的ThinkPHP，Python的Dijango、Flask、NodeJs的Express，Ruby的RoR，基本都采纳了这种模式。平常所讲的MVC基本是这种服务端的MVC。

## MVP

MVP模式有两种：

1. Passive View
2. Supervising Controller

大多数情况下讨论的都是Passive View模式。本文会对PV模式进行较为详细的介绍，而SC模式则简单提及。

### MVP(Passive View)的依赖关系

MVP模式把MVC中的Controller换成了Presenter。MVP层次之间的依赖关系如下：

![image-20210203150516401](https://raw.githubusercontent.com/silence/blog/assets/assets/20210203150516.png)

MVP打破了View原来对于Model的依赖，其余的依赖关系和MVC模式一致。

### MVP(Passive View)的调用关系

![image-20210205171656575](https://raw.githubusercontent.com/silence/blog/assets/assets/20210205171656.png)

和MVC模式一样，用户对View的操作都会从View交移给Presenter。Presenter会执行相应的应用程序逻辑，并且对Model进行相应的操作；而这时候Model执行完业务逻辑以后，也是通过观察者模式把自己变更的消息传递出去，但是是传给Presenter而不是View。Presenter获取到Model变更的消息以后，**通过View提供的接口更新界面**。

关键点：

1. View 不再负责同步的逻辑，而是由Presenter负责。Presenter中既有应用程序逻辑也有同步逻辑。
2. **View需要提供操作界面的接口给Presenter调用。**

对比在MVC中，Controller是不能操作View的，View也没有提供相应的接口；而在MVP当中，Presenter可以操作View，View需要提供一组对界面操作的接口给Presenter进行调用；Model仍然通过事件广播自己的变更，但由Presenter监听而不是View。

### MVP(Passive View)的优缺点

优点：

1. 便于测试。Presenter对View是通过接口进行，在对Presenter进行不依赖UI环境的单元测试的时候。可以通过Mock一个View对象，这个对象只需要实现了View的接口即可。然后依赖注入到Presenter中，单元测试的时候就可以完整的测试Presenter应用逻辑的正确性。
2. View可以进行组件化。在MVP当中，View不依赖Model。这样就可以让View从特定的业务场景中脱离出来，可以说View可以做到对业务完全无知。它只需要提供一系列接口提供给上层操作。这样就可以做到高度可复用的View组件。

缺点：

1. Presenter中除了应用逻辑以外，还有大量的View -> Model，Model -> View的手动同步逻辑，造成Presenter比较笨重，维护起来会比较困难。

### MVP（Supervising Controller）

上面讲的是MVP的Passive View模式，该模式下View非常Passive，它几乎什么都不知道，Presenter让它干什么它就干什么。而Supervising Controller模式中，Presenter会把一部分简单的同步逻辑交给View自己去做，Presenter只负责比较复杂的、高层次的UI操作，所以可以把它看成一个Supervising Controller。

![image-20210205171736829](https://raw.githubusercontent.com/silence/blog/assets/assets/20210205171736.png)

因为Supervising Controller用得比较少，对它的讨论就到此为止。

## MVVM

MVVM可以看作是一种特殊的MVP(Passive View)模式，或者说是对MVP模式的一种改良。

### ViewModel

MVVM代表的是Model-View-ViewModel，这里需要解释一下什么是ViewModel。ViewModel的含义就是“Model of View”，视图的模型。它的含义包含了领域模型（Domain Model）和视图的状态（State）。在图形界面应用程序当中，界面所提供的信息可能不仅仅包含应用程序的领域模型。还可能包含一些领域模型不包含的视图状态，例如电子表格程序上需要显示当前排序的状态是顺序的还是逆序的，而这是Domain Model所不包含的，但也是需要显示的信息。

可以简单把ViewModel理解为页面上所显示内容的数据抽象，和Domain Model不一样，ViewModel更适合用来描述View。

### MVVM的依赖

MVVM的依赖关系和MVP依赖，只不过是把P换成了VM

### MVVM的调用关系

MVVM的调用关系和MVP一样。但是，在ViewModel当中会有一个叫Binder，或者是Data-binding engine的东西。以前全部由Presenter负责的View和Model之间数据同步操作交由给Binder处理。你只需要在View的模板语法当中，指令式地声明View上的显示的内容是和Model的哪一块数据绑定的。当ViewModel对进行Model更新的时候，Binder会自动把数据更新到View上去，当用户对View进行操作（例如表单输入），Binder也会自动把数据更新到Model上去。这种方式称为：Two-way data-binding，双向数据绑定。可以简单而不恰当地理解为一个模板引擎，但是会根据数据变更实时渲染。

![image-20210205171814291](https://raw.githubusercontent.com/silence/blog/assets/assets/20210205171814.png)

也就是说，MVVM把View和Model的同步逻辑自动化了。以前Presenter负责的View和Model同步不再手动地进行操作，而是交由框架所提供的Binder进行负责。只需要告诉Binder，View显示的数据对应的是Model哪一部分即可。

### MVVM的优缺点

优点：

1. 提高可维护性。解决了MVP大量的手动View和Model同步的问题，提供双向绑定机制。提高了代码的可维护性。
2. 简化测试。因为同步逻辑是交由Binder做的，View跟着Model同时变更，所以只需要保证Model的正确性，View就正确。大大减少了对View同步更新的测试。

缺点：

1. 过于简单的图形界面不适用，或说牛刀杀鸡。
2. 对于大型的图形应用程序，视图状态较多，ViewModel的构建和维护的成本都会比较高。
3. 数据绑定的声明是指令式地写在View的模板当中的，这些内容是没办法去打断点debug的。

### Reference

https://github.com/livoras/blog/issues/11

