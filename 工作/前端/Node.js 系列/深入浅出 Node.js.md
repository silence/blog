本文为 《深入浅出 Node.js》的重点摘抄。

[TOC]

### 第 1 章 Node 简介

#### 1.2 Node 的命名与起源

##### 1.2.1 为什么是 JavaScript

考虑到高性能、符合事件驱动、没有历史包袱这 3 个主要原因，JavaScript 成为了 Node 的实现语言。

##### 1.2.2 为什么叫 Node

Node 发展为一个强制不共享任何资源的单线程、单进程系统，包含十分适宜网络的库，为构建大型分布式应用程序提供基础设施，其目标也是**成为一个构建快速、可伸缩的网络应用平台**。它自身非常简单，通过通信协议来组织许多 Node，非常容易通过扩展来达成构建大型网络应用的目的。每一个 Node 进程都构成这个网络应用中的一个节点，这是它名字所含意义的真谛。

#### 1.4 Node 的特点

##### 1.4.1 异步 I/O

异步调用中对于结果值的捕获是符合 “**Don't call me, I will call you**” 的原则的，这也是注重结果，不关心过程的一种表现。

##### 1.4.2 事件与回调函数

##### 1.4.3 单线程

单线程的最大好处是不用像多线程编程那样处处在意状态的同步问题，这里没有死锁的存在，也没有线程上下文交换所带来的性能上的开销。

同样，单线程的弱点具体有以下 3 个方面。

- 无法利用多核 CPU。
- 错误会引起整个应用退出，应用的健壮性值得考验。
- 大量计算占用 CPU 导致无法继续调用异步 I/O。

像浏览器中 JavaScript 与 UI 共用一个线程一样，JavaScript 长时间执行会导致 UI 的渲染和响应被中断。在 Node 中，长时间的 CPU 占用也会导致后续的异步 I/O 发不出调用，已完成的异步 I/O 的回调函数也会得不到及时执行。

在 HTML5 中，Web Workers 能够创建工作线程来进行计算，以解决 JavaScript 大计算阻塞 UI 渲染的问题。工作线程为了不阻塞主线程，通过消息传递的方式来传递运行结果，这也使得工作线程不能访问到主线程中的 UI。

Node 采用了与 Web Workers 相同的思路来解决单线程中大计算量的问题：child_process。

子进程的出现，意味着 Node 可以从容地应对单线程在健壮性和无法利用多核 CPU 方面的问题。通过将计算分发到各个子进程，可将大量计算分解掉，然后再通过进程之间的事件消息来传递结果，这可以很好地保持应用模型的简单和低依赖。通过 **Master-Worker** 的管理方式，也可以很好地管理各个工作进程，以达到更高的健壮性。

##### 1.4.4 跨平台

它在操作系统与 Node 上层模块系统之间构建了一层**平台层架构**，即 **libuv**。目前，libuv 已经成为许多系统实现跨平台的基础组件。关于 libuv 的设计，我们将在第 3 章中介绍。

#### 1.5 Node 的应用场景

关于 Node，探讨得较多的主要有 I/O 密集型和 CPU 密集型。

##### 1.5.1 I/O 密集型

Node 面向网络且擅长并行 I/O，能够有效地组织起更多的硬件资源，从而提供更多好的服务。

I/O 密集的优势主要在于 **Node 利用事件循环的处理能力，而不是启动每一个线程为每一个请求服务，资源占用极少。**

##### 1.5.2 是否不擅长 CPU 密集型业务

CPU 密集型应用给 Node 带来的挑战主要是：由于 JavaScript 单线程的原因，如果有长时间运行的计算（比如大循环），将会导致 CPU 时间片不能释放，使得后续 I/O 无法发起。但是适当调整和分解大型运算任务为多个小任务，使得运算能够适时释放，不阻塞 I/O 调用的发起，这样既可同时享受到并行异步 I/O 的好处，又能充分利用 CPU。

对于长时间运行的计算，Node 虽然没有提供多线程用于计算支持，但是还是有以下两个方式来充分利用 CPU。

- Node 可以通过编写 C/C++ 扩展的方式更高效地利用 CPU，将一些 V8 不能做到性能极致的地方通过 C/C++ 来实现。由上面的测试结果可以看到，通过 C/C++ 扩展的方式实现斐波那契数列计算，速度比 Java 还快。
- 如果单线程的 Node 不能满足需求，甚至用了 C/C++ 扩展后还觉得不够，那么通过子进程的方式，将一部分 Node 进程当做常驻服务进程用于计算，然后利用进程间的消息来传递结果，将计算与 I/O 分离，这样还能充分利用多 CPU。

CPU 密集不可怕，如何合理调度是诀窍。

##### 1.5.4 分布式应用

阿里巴巴的数据平台对 Node 的分布式应用算是一个典型的例子。分布式应用意味着对可伸缩性的要求非常高。数据平台通常要在一个数据库集群中去寻找需要的数据。阿里巴巴开发了中间层应用 NodeFox、ITier，将数据库集群做了划分和映射，查询调用依旧是针对单张表进行 SQL 查询，中间层分解查询 SQL，并行地去多台数据库中获取数据并合并。NodeFox 能实现对多台 MySQL 数据库的查询，如同查询一台 MySQL 一样，而 ITier 更强大，查询多个数据库如同查询单个数据库一样，这里的多个数据库是指不同的数据库，如 MySQL 或其他数据库。

### 第 2 章 模块机制

#### 2.2 Node 的模块实现

在 Node 中引入模块，需要经历如下 3 个步骤。

1. 路径分析
2. 文件定位
3. 编译执行

在 Node 中，模块分为两类：一类是 Node 提供的模块，称为核心模块；另一类是用户编写的模块，称为文件模块。

- 核心模块部分在 Node 源代码的编译过程中，编译进了二进制执行文件。在 Node 进程启动时，部分核心模块就被直接加载进内存中，所以这部分核心模块引入时，文件定位和编译执行这两个步骤可以省略掉，并且在路径分析中优先判断，所以它的加载速度是最快的。
- 文件模块则是在运行时动态加载，需要完整的**路径分析、文件定位、编译执行**过程，速度比核心模块慢。

接下来，我们展开详细的模块加载过程。

##### 2.2.1 优先从缓存加载

展开介绍路径分析和文件定位之前，我们需要知晓的一点是，与前端浏览器会缓存静态脚本文件以提高性能一样，Node 对引入过的模块都会进行缓存，以减少二次引入时的开销。不同的地方在于，**浏览器仅仅缓存文件，而 Node 缓存的是编译和执行之后的对象**。

不论是核心模块还是文件模块，**require() 方法对相同模块的二次加载都一律采用缓存优先的方式，这是第一优先级的。不同之处在于核心模块的缓存检查先于文件模块的缓存检查。**

##### 2.2.2 路径分析和文件定位

因为标识符有几种方式，对于不同的标识符，模块的查找和定位有不同程度上的差异。

1. 模块标识符分析

前面提到过，require() 方法接受一个标识符作为参数。在 Node 实现中，正是基于这样一个标识符进行模块查找的。模块标识符在 Node 中主要分为以下几类。

- 核心模块，如 http、fs、path 等。

- . 或 .. 开始的相对路径文件模块。
- 以 / 开始的绝对路径文件模块。
- 非路径形式的文件模块，如自定义的 connect 模块。

[1] 核心模块

核心模块的优先级仅次于缓存加载，它在 Node 的源代码编译过程中已经编译为二进制代码，其加载过程最快。

如果试图加载一个与核心模块标识符相同的自定义模块，那是不会成功的。如果自己编写了一个 http 用户模块，想要加载成功，必须选择一个不同的标识符或者换用路径的方式。

[2] 路径形式的文件模块

以 .、.. 和 / 开头标识符，这里都被当做文件模块来处理。在分析路径模块时，require() 方式会将路径转为真实路径，并以真实路径作为索引，将编译执行后的结果存放到缓存中，以使二次加载时更快。

由于文件模块给 Node 指明了确切的文件位置，所以在查找过程中可以节约大量时间，其加载速度慢于核心模块。

[3] 自定义模块

自定义模块指的是非核心模块，也不是路径形式的标识符。它是一种特殊的文件模块，可能是一个文件或者包的形式。这类模块的查找是最费时的，也是所有方式中最慢的一种。

在介绍自定义模块的查找方式之前，需要先介绍一下 模块路径 这个概念。

模块路径是 Node 在定位文件模块的具体文件时制定的查找策略，具体表现为一个路径组成的数组。关于这个路径的生成规则，我们可以手动尝试一番。

(1). 创建 *module_path.js* 文件，其内容为 console.log(module.paths);。

(2). 将其放到任意一个目录中然后执行 node *module_path.js*。

在 Linux 下，你可能得到的是这样一个数组输出：

['/home/jackson/research/node_modules', '/home/jackson/node_modules', '/home/node_modules', '/node_modules']

而在 windows 下，也许是这样：

['c:\\\\nodejs\\\\node_modules', 'c:\\\\node_modules']

可以看出，模块路径的生成规则如下所示。

- 当前文件目录下的 node_modules 目录。
- 父目录下的 node_modules 目录。
- 父目录的父目录下的 node_modules 目录。
- 沿路径向上逐级递归，直到根目录下的 node_modules 目录。

它的生成方式与 JavaScript 的原型链或作用域链的查找方式十分类似。在加载的过程中，Node 会逐个尝试模块路径中的路径，直到找到目标文件为止。可以看出，当前文件的路径越深，模块查找耗时会越多，这是自定义模块的加载速度是最慢的原因。

2. 文件定位

从缓存加载的优化策略使得二次引入时不需要路径分析、文件定位和编译执行的过程，大大提高了再次加载模块时的效率。

但在文件的定位过程中，还有一些细节需要注意，这主要包括文件扩展名的分析、目录和包的处理。

- 文件扩展名的分析

  require() 在分析标识符的过程中，会出现标识符中不包含文件拓展名的情况。CommonJS 模块规范也允许在标识符中不包含文件拓展名，这种情况下，**Node 会按 .js、.json、.node 的次序补足拓展名，依次尝试**。

  在尝试的过程中，需要调用 fs 模块同步阻塞式地判断文件是否存在。因为 Node 是单线程的，所以这里是一个会引起性能问题的地方。小诀窍是：如果是 .node 和 .json 文件，在传递给 require() 的标识符中带上拓展名，会加快一点速度。另一个诀窍是：同步配合缓存，可以大幅度缓解 Node 单线程中阻塞式调用的缺陷。
  
- 目录分析和包
  
  在分析标识符的过程中，require() 通过分析文件扩展名之后，可能没有查找到对应文件，但却得到一个目录，这在引入自定义模块和逐个模块路径进行查找时经常会出现，**此时 Node 会将目录当做一个包来处理。**
  
  在这个过程中，Node 对 CommonJS 包规范进行了一定程度的支持。首先，Node 在当前目录下查找 package.json（CommonJS 包规范定义的包描述文件），通过 JSON.parse() 解析出包描述对象，**从中取出 main 属性指定的文件名进行定位，或者压根没有 package.json 文件，Node 会将 index 当做默认文件名，然后依次查找 index.js、index.json、index.node**。
  
  如果在目录分析的过程中没有定位成功任何文件，则自定义模块进入下一个模块路径进行查找。如果模块路径数组都被遍历完毕，依然没有查找到目标文件，则会抛出查找失败的异常。

##### 2.2.3 模块编译

在 Node 中，每个文件模块都是一个对象，它的定义如下：

```javascript
function Module(id, parent) {
  this.id = id;
  this.exports = {};
  this.parent = parent;
  if (parent && parent.children) {
    parent.children.push(this);
  }
  
  this.filename = null;
  this.loaded = false;
  this.children = [];
}
```

编译和执行是引入文件模块的最后一个阶段。定位到具体的文件后，Node 会新建一个模块对象，然后根据路径载入并编译。对于不同的文件扩展名，其载入方法也有所不同，具体如下所示。

- **.js 文件**。通过 fs 模块**同步**读取文件后编译执行。
- **.node 文件**。这是用 C/C++ 编写的扩展文件，通过 dlopen() 方法加载最后编译生成的文件。
- **.json 文件**。通过 fs 模块同步读取文件后，用 JSON.parse() 解析返回结果。
- **其余扩展名文件**。它们都被当做 .js 文件载入。

每一个编译成功的模块都会将其文件路径作为索引缓存在 Module._cache 对象上，以提高二次引入的性能。

**根据不同的文件扩展名，Node 会调用不同的读取方式**。可以通过在代码中访问 require.extensions 可以自动系统中已有的扩展加载方式。

```javascript
console.log(require.extensions);
```

得到的执行结果如下：

```javascript
{ '.js': [Function], '.json': [Function], '.node': [Function] }
```

在确定文件的扩展名之后，Node 将调用具体的编译方式来将文件执行后返回给调用者。

1. JavaScript 模块的编译

在编译的过程中，Node 对获取的 JavaScript 文件内容进行头尾包装。在头部添加了 (function (exports, require, module, __filename, __dirname) {\n，在尾部添加了\n});

一个正常的 JavaScript 文件会被包装成如下的样子：

```javascript
(function (exports, require, module, __filename, __dirname) { 
  var math = require('math');
	exports.area = function (radius) {
	return Math.PI * radius * radius; 
  };
});
```

这样每个模块文件之间都进行作用域隔离。包装之后的代码会通过 vm 原生模块的 runInThisContext() 方法执行（类似 eval，只是具有明确上下文，不污染全局），返回一个具体的 function 对象。最后，将当前模块对象的 exports 属性、require() 方法、module（模块对象自身），以及在文件定位中得到的**完整文件路径**(__filename)和**文件目录**(\_\_dirname)作为参数传递给这个 function() 执行。

这就是这些变量并没有定义在每个模块文件中却存在的原因。在执行之后，模块的 exports 属性被返回给了调用方。exports 属性上的任何方法和属性都可以被外部调用到，但是模块中的其余变量或属性则不可直接被调用。

至此，require、exports、module 的流程已经完整，这就是 Node 对 CommonJS 模块规范的实现。

#### 2.3 核心模块

核心模块分为 C/C++ 编写的和 JavaScript 编写的两部分，其中 C/C++ 文件存放在 Node 项目的 src 目录下。JavaScript 文件存放在 lib 目录下。

##### 2.3.1 JavaScript 核心模块的编译过程

1. 转存为 C/C++ 代码

   Node 采用了 V8 附带的 js2c.py 工具，将所有内置的 JavaScript 代码 (src/node.js 和 lib/*.js) 转换成 C++ 里的数组，生成 node_native.h 头文件。

   在这个过程中，JavaScript 代码以字符串的形式存储在 node 命名空间中，是不可直接执行的。在启动 Node 进程时，JavaScript 代码直接加载进内存中。**在加载的过程中，JavaScript 核心模块经历标识符分析后直接定位到内存中，比普通的文件模块从磁盘中一处一处查找要快很多。**

##### 2.3.3 核心模块的引入流程

从图 2-5 所示的 os 原生模块的引入流程可以看到，为了符合 CommonJS 模块规范，从 JavaScript 到 C/C++ 的过程是相当复杂的，它要经历 C/C++ 层面的内建模块定义、（JavaScript）核心模块的定义和引入以及（JavaScript）文件模块层面的引入。但是对于用户而言，require() 十分简洁、友好。

![image-20210602110908651](https://raw.githubusercontent.com/silence/blog/assets/assets/20210602110908.png)

#### 2.4 C/C++ 扩展模块

##### 2.4.1 前提条件

- GYP 项目生成工具。

  GYP 即 “Generate Your Projects” 短句的缩写。它的好处在于，可以帮助你生成各个平台下的项目文件，比如 Windows 下的 Visual Studio 解决方案(.sln)、Mac 下的 XCode 项目配置文件以及 Scons 工具。在这个基础上，再动用各自平台下的编译器编译项目。这大大减少了跨平台模块在项目组织上的精力投入。

### 第 3 章 异步 I/O

异步 I/O 的调用示意图

![image-20210603164039610](https://raw.githubusercontent.com/silence/blog/assets/assets/20210603164039.png)

#### 3.2 异步 I/O 实现现状

##### 3.2.1 异步 I/O 与非阻塞 I/O

操作系统内核对于 I/O 只有两种方式：阻塞与非阻塞。

![image-20210605174937066](https://raw.githubusercontent.com/silence/blog/assets/assets/20210605174937.png)

操作系统对计算机进行了抽象，将所有输入输出设备抽象为文件。内核在进行文件 I/O 操作时，通过**文件描述符**进行管理，而**文件描述符**类似于应用程序与系统内核之间的凭证。应用程序如果需要进行 I/O 调用，需要先打开文件描述符，然后再根据文件描述符去实现文件的数据读写。此处非阻塞 I/O 与 阻塞 I/O 的区别在于阻塞 I/O 完成整个获取数据的过程，而**非阻塞 I/O 则不带数据直接返回，要获取数据，还需要通过文件描述符再次读取**。

![image-20210605175252798](https://raw.githubusercontent.com/silence/blog/assets/assets/20210605175252.png)

非阻塞 I/O 返回之后，CPU 的时间片可以用来处理其他事务，此时的性能提升是明显的。

但非阻塞 I/O 也存在一些问题。由于完整的 I/O 并没有完成，立即返回的**并不是业务层期望的数据，而仅仅是当前调用的状态**。为了获取完整的数据，应用程序需要重复调用 I/O 操作来确认是否完成。这种重复调用判断操作是否完成的技术叫做**轮询**。

任意技术都并非完美的。阻塞 I/O 造成 CPU 等待浪费，非阻塞带来的麻烦却是需要轮询去确认是否完全完成数据的获取，它会让 CPU 处理状态判断，是对 CPU 资源的浪费。

现现存的轮询技术主要有 read、select、poll、epoll。

epoll 方案是 Linux 下效率最高的 I/O 事件通知机制，在进入轮询的时候如果没有检查到 I/O 事件，将会进行休眠，直到事件发生将它唤醒。它是真实利用了**事件通知、执行回调**的方式，而不是遍历查询，所以不会浪费 CPU，执行效率最高。图 3-7 为通过 epoll 方式实现轮询的示意图。

![image-20210605180204571](https://raw.githubusercontent.com/silence/blog/assets/assets/20210605180204.png)

##### 3.2.2 理想的非阻塞异步 I/O

尽管 epoll 已经利用了事件来降低 CPU 的耗用，但是休眠期间 CPU 几乎是闲置的，对于当前程序而言利用率不够。那么是否用一种理想的异步 I/O 呢？

##### 3.2.3 现实的异步 I/O 

前面我们将场景限定在了单线程的状况下，如果是多线程会是另一番风景。通过让部分线程进行阻塞 I/O 或者非阻塞 I/O 加轮询技术来完成数据获取，让一个线程进行计算处理，通过线程之间的通信将 I/O 得到的数据进行传递，这就轻松实现了异步 I/O（尽管它是模拟的）。

![image-20210605180849383](https://raw.githubusercontent.com/silence/blog/assets/assets/20210605180849.png)

在 Node v0.9.3 中，自行实现了线程池来完成异步 I/O。

Windows 下的 IOCP，它在某种程度上提供了理想的异步 I/O：调用异步方法，等待 I/O 完成之后的通知，执行回调，用户无须考虑轮询。但是它的内部其实仍然是线程池原理，不同之处在于这些线程池由系统内核接手管理。

由于 Window 平台和 *nix 平台的差异，Node 提供了 libuv 作为抽象封装层。Node 在编译期间会判断平台条件，选择性编译 unix 目录或是 win 目录下的源文件到目标程序。

![image-20210605181540133](https://raw.githubusercontent.com/silence/blog/assets/assets/20210605181540.png)

需要强调的是，这里的 I/O 不仅仅只限于磁盘文件的读写。***nix 将计算机抽象了一番，磁盘文件、硬件、套接字等几乎所有计算机资源都被抽象为了文件**，因此这里描述的阻塞和非阻塞的情况同样能适合于套接字等。

**另一个需要强调的地方在于我们时常提到 Node 是单线程的，这里的单线程仅仅只是 JavaScript 执行在单线程中罢了。在 Node 中，无论是 *nix 还是 Windows 平台，内部完成 I/O 任务的另有线程池。**

#### 3.3 Node 的异步 I/O

##### 3.3.1 事件循环

首先我们着重强调一下 Node 自身的执行模型 --- 事件循环，正是它使得回调函数十分普遍。

在进程启动时，Node 便会创建一个类似于 while(true) 的循环，每执行一次循环体的过程我们称为 tick。**每个 tick 的过程就是查看是否有事件待处理，如果有，就取出事件及其相关的回调函数。如果存在关联的回调函数，就执行它们。然后进入下个循环，如果不再有事件处理，就退出进程。**

![image-20210605182356688](https://raw.githubusercontent.com/silence/blog/assets/assets/20210605182356.png)

##### 3.3.2 观察者

在每个 tick 的过程中，如何判断是否有事件需要处理呢？这里必须要引入的概念是**观察者**。

每个事件循环中有一个或者多个观察者，而判断是否有事件要处理的过程就是向这些观察者询问是否有要处理的事件。

浏览器采用了类似的机制。事件可能来自用户的点击或者加载某些文件时产生，而这些产生的事件都有对应的观察者。在 Node 中，时间主要来源于网络请求、文件 I/O 等，这些事件对应的观察者有文件 I/O 观察者、网络 I/O 观察者等。观察者将事件进行了分类。

事件循环是一个典型的 **生产者/消费者模型** 。异步 I/O、网络请求等则是事件的**生产者**，源源不断为 Node 提供不同类型的事件，这些事件被传递到对应的观察者那里，**事件循环则从观察者那里取出事件并处理**。

在 Windows 下，这个循环基于 IOCP 创建，而在 *nix 下则基于多线程创建。

##### 3.3.3 请求对象

在这一节中，我们将通过解释 Windows 下异步 I/O（利用 IOCP 实现）的简单例子来探寻从 JavaScript 代码到系统内核之间都发生了什么。

对于一般的（非异步）回调函数，函数由我们自行调用，如下所示：

```javascript
var forEach = function (list, callback) {
  for (var i = 0; i < list.length; i++) {
    callback(list[i], i , list);
  }
}
```

对于 Node 中的异步 I/O 调用而言，回调函数却不由开发者来调用。那么从我们发出调用后，到回调函数执行，中间发送了什么？事实上，从 JavaScript 发起调用到内核执行完 I/O 操作的过渡过程中，存在一种中间产物，它叫做 **请求对象**。

下面我们以最简单的 fs.open() 方法来作为例子，探索 Node 与底层之间是如何异步 I/O 调用以及回调函数究竟是如何被调用执行的：

```javascript
fs.open = function (path, flags, mode, callback) {
  // ...
  binding.open(pathModule._makeLong(path), stringToFlags(flags), mode, callback)
}
```

fs.open() 的作用是根据指定路径和参数去打开一个文件，从而得到一个文件描述符，这是后续所有 I/O 操作的初始操作。从前面的代码中可以看到，JavaScript 层面的代码通过调用 C++ 核心模块进行下层的操作。图 3-12 为调用示意图。

![image-20210605205130510](https://raw.githubusercontent.com/silence/blog/assets/assets/20210605205130.png)

在 nv_fs_open() 的调用过程中，我们创建了一个 FSReqWrap 请求对象。从 JavaScript 层传入的参数和当前方法都被封装在这个请求对象中，其中我们最为关注的回调函数则被设置在这个对象的 oncomplete_sym 属性上：

```javascript
req_wrap -> object_ -> Set(oncomplete_sym, callback);
```

对象包装完毕后，在 Windows 下，则调用 QueueUserWorkItem() 方法将这个 FSReqWrap 对象推入线程池中等待执行。

当线程池中有可用线程时，我们会调用 uv_fs_thread_proc() 方法。uv_fs_thread_proc() 方法会根据传入参数的类型调用相应的底层函数。

至此，JavaScript 调用立即返回，由 JavaScript 层面发起的异步调用的第一阶段就此结束。JavaScript 线程可以继续执行当前任务的后续操作。**当前的 I/O 操作在线程池中等待执行，不管它是否阻塞 I/O，都不会影响到 JavaScript 线程的后续执行，如此就达到了异步的目的。**

请求对象是异步 I/O 过程中的重要中间产物，所有的状态都保存在这个对象中，包括**送入线程池等待执行以及 I/O 操作完毕后的回调处理**。

##### 3.3.4 执行回调

组装好请求对象、送入 I/O 线程池等待执行，实际上完成了异步 I/O 的第一部分，回调通知是第二部分。

线程池中的 I/O 操作调用完毕之后，会将获取的结果储存在 req -> result 属性上，然后调用 PostQueuedCompletionStatus() 通知 IOCP，告知当前对象操作已经完成：

```javascript
PostQueuedCompletionStatus((loop) -> iocp, 0, 0, &((req) -> overlapped))
```

PostQueuedCompletionStatus() 方法的作用是向 IOCP 提交执行状态，并将线程归还线程池。通过 PostQueuedCompletionStatus() 方法提交的状态可以通过 GetQueuedCompletionStatus() 提取。

在这个过程中，我们其实还动用了事件循环的 I/O 观察者。在每次 Tick 的执行中，它会调用 IOCP 相关的 GetQueuedCompletionStatus() 方法检查线程池中是否有执行完的请求，如果存在，会将请求对象加入到 I/O 观察者的队列中，然后将其当做事件处理。

**I/O 观察者回调函数的行为就是取出请求对象的 result 属性作为参数，取出 oncomplete_sym 属性作为方法，然后调用执行，以此达到调用 JavaScript 中传入的回调函数的目的。**

至此，整个异步 I/O 的流程完全结束，如图 3-13 所示。

![image-20210605211057144](https://raw.githubusercontent.com/silence/blog/assets/assets/20210605211057.png)

事件循环、观察者、请求对象、I/O 线程池这四者共同构成了 Node 异步 I/O 模型的基本要素。

不同的是线程池在 Windows 下由内核（IOCP）直接提供，*nix 系列下由 libuv 自行实现。

##### 3.3.5 小结

事实上，在 Node 中，除了 JavaScript 是单线程外，Node 自身其实是多线程的，只是 I/O 线程使用的 CPU 较少。另一个需要重视的观点则是，除了用户代码无法并行执行外，所有的 I/O（磁盘 I/O 和网络 I/O 等）则是可以并行起来的。

#### 3.4 非 I/O 的异步 API

与 I/O 无关的异步 API，这一部分也值得略微关注一下，它们分别是 **setTimeout()**、**setInterval()**、**setImmediate()** 和 **process.nextTick()**。

##### 3.4.1 定时器

setTimeout() 和 setInterval() 与浏览器中的 API 是一致的，分别用于单次和多次定时执行任务。它们的实现原理与异步 I/O 比较类似，只是不需要 I/O 线程池的参与。

定时器的问题在于，它并非精确的（在容忍范围内）。尽管事件循环十分快，但是如果某一次循环占用的时间较多，那么下次循环时，它也许已经超时很久了。譬如通过 setTimeout() 设定一个任务在 10ms 后执行，但是在 9ms 后，有一个任务占用了 5ms 的 CPU 时间片，再次轮到定时器执行时，时间就已经过期 4ms。

![image-20210605212441951](https://raw.githubusercontent.com/silence/blog/assets/assets/20210605212442.png)

##### 3.4.2 process.nextTick()

在未了解 process.nextTick() 之前，很多人也许为了立即异步执行一个任务，会这样调用 setTimeout() 来达到所需的效果：（说的就是我）

```javascript
setTimeout(() => {
  // ... TODO
}, 0)
```

由于事件循环的特点，定时器的精确度不够。而事实上，采用定时器需要动用红黑树，创建定时器对象和迭代等操作，而 setTimeout(fn, 0) 的方式较为浪费性能。实际上，process.nextTick() 方法的操作相对较为轻量。

```javascript
process.nextTick = function(callback) {
  // on the way out, don't bother
  // it won't get fired anyway
  if (process._exiting) return;
  
  if (tickDepth >= process.maxTickDepth)
    maxTickWarn();
  
  var tock = { callback: callback };
  if (process.domain) tock.domain = process.domain;
  nextTickQueue.push(tock);
  if (nextTickQueue.length) {
    process._needTickCallback();
  }
}
```

**每次调用 process.nextTick() 方法，只会将回调函数放入队列中，在下一轮 Tick 时取出执行**。~~定时器中采用红黑树的操作复杂度为 O(lg(n))，nextTick() 的时间复杂度为 O(1)。相较之下，process.nextTick() 更高效~~（我个人认为这种语法上的性能差异不用理会）。

##### 3.4.3 setImmediate()

setImmediate() 和 process.nextTick() 方法十分类似，都是将回调函数延迟执行。

将它们放在一起时，示例代码如下：

```javascript
process.nextTick(function() {
  console.log('nextTick 延迟执行')
})

setImmediate(function() {
  console.log('setImmediate 延迟执行')
})

console.log('正常执行')

// 执行结果如下：
// 正常执行
// nextTick 延迟执行
// setImmediate 延迟执行
```

从结果里可以看到，process.nextTick() 中的回调函数执行的优先级要高于 setImmediate()。这里的原因在于**事件循环对观察者的检查是有先后顺序的**，process.nextTick() 属于 idle 观察者，setImmediate() 属于 check 观察者。在每一轮循环检查中，idle 观察者先于 I/O 观察者，I/O 观察者先于 check 观察者。

在具体实现上，process.nextTick() 的回调函数保存在与各数组中，setImmediate() 的结果则是保存在链表中。在行为上，process.nextTick() **在每轮循环中会将数组中的回调函数全部执行完**，而 setImmediate() **在每轮循环中执行链表中的一个回调函数**。

#### 3.5 事件驱动与高性能服务器

对于网络套接字的处理，Node 也应用到了异步 I/O，利用 Node 构建 Web 服务器的流程图如下：

![image-20210605214622637](https://raw.githubusercontent.com/silence/blog/assets/assets/20210605214622.png)

下面为几种经典的服务器模型，这里对比下它们的优缺点。

- **同步式**。一次只能处理一个请求，并且其余请求都处于等待状态。
- **每进程/每请求**。为每个请求启动一个进程，这样可以处理多个请求，但是它不具备扩展性，因为系统资源只有那么多。
- **每线程/每请求**。为每个请求启动一个线程来处理。扩展性比每进程/每请求要好，但对于大型站点而言依然不够。

每线程/每请求的方式目前还被 Apache 所采用。事件驱动带来的高效已经渐渐开始为业界所重视。知名服务器 Ngnix，也摒弃了多线程的方式，采用了和 Node 相同的事件驱动，达到高性能的效果。

### 第 4 章 异步编程

最新的 ECMAScript 6+ 里的 Promise 已经是很好的异步编程实践了。

### 第 5 章 内存控制

#### 5.1 V8 的垃圾回收机制与内存限制

##### 5.1.2 V8 的内存限制

在 Node 中通过 JavaScript 使用内存时就会发现只能使用部分内存（64位系统下约为 1.4 GB，32位系统下约为 0.7GB）。

要知晓 V8 为何限制了内存的用量，则需要回归到 V8 在内存使用上的策略。知晓其原理后，才能避免问题并更好地进行内存管理。

##### 5.1.3 V8 的对象分配

在 V8 中，所有的 JavaScript 对象都是通过堆来进行分配的。Node 提供了 V8 中内存使用量的查看方式，执行下面的代码，将得到输出的内存信息：

```javascript
process.memoryUsage()
{
	rss: 14858567,
  heapTotal: 71934567,
  heapUsed: 28245732
}
```

在上述代码中，在 memoryUsage() 方法返回的 3 个属性中，heapTotal 和 heapUsed 是 V8 的堆内存使用情况，**前者是已申请到的堆内存**，**后者是当前使用的量**。至于 rss 为何，我们在后续的内容中会介绍到。图 5-1 为 V8 的堆示意图：

![image-20210605225522097](https://raw.githubusercontent.com/silence/blog/assets/assets/20210605225522.png)

当我们在代码中声明变量并赋值时，所使用的对象的内存就分配在堆中。如果已申请的堆空闲内存不够分配新的对象，将继续申请堆内存，直到堆的大小超过 V8 的限制为止。

按官方的说法，以 1.5GB 的垃圾回收堆内存为例，V8 做一次小的垃圾回收需要 50ms 以上，做一次非增量式的垃圾回收甚至要 1s 以上。这是垃圾回收引起 JavaScript 线程暂停执行的时间，在这样的时间花销下，应用的性能和响应能力都会直线下降。

Node 在启动时可以传递 --max-old-space-size 或 --max-new-space-size 来调整内存限制的大小。

##### 5.1.4 V8 的垃圾回收机制

在展开介绍 V8 的垃圾回收机制前，有必要简略介绍下 V8 用到的各种垃圾回收算法。

1. V8 主要的垃圾回收算法

现代的垃圾回收算法中按对象的存活时间将内存的垃圾回收进行不同的分代，然后分别对不同分代的内存施以更高效的算法。

- V8 的内存分代

  在 V8 中，主要将内存分为新生代和老生代两代。**新生代中的对象为存活时间较短的对象**，**老生代中的对象为存活时间较长或常驻内存的对象**。图 5-2 为 V8 的分代示意图。

  ![image-20210605230559173](https://raw.githubusercontent.com/silence/blog/assets/assets/20210605230559.png)

- Scavenge 算法

  在 Scavenge 的具体实现中，它主要采用了 Cheney 算法。它将堆内存一分为二，每一部分空间称为 semispace。这两个 semisapce 的空间中，只有一个处于使用中，另一个处于闲置状态。处于使用中的称为 From 空间，处于闲置状态的空间称为 To 空间。**当开始进行垃圾回收时，会检查 From 空间中的存活对象，这些存活对象将被复制到 To 空间中，而非存活对象占用的空间将会被释放。完成复制后，From 空间和 To 空间的角色发生互换。**

  V8 的堆内存示意图应当如图 5-3 所示。

  ![image-20210605233038256](https://raw.githubusercontent.com/silence/blog/assets/assets/20210605233038.png)

  在默认情况下，V8 的对象分配主要集中在 From 空间中。对象从 From 空间中复制到 To 空间时，会检查它的内存地址来判断这个对象是否已经经历过一次 Scavenge 回收。如果已经经历过了，会将该对象从 From 空间复制到老生代空间中，如果没有，则复制到 To 空间中。

  另一个判断条件是 To 空间的内存占用比。当要从 From 空间复制一个对象到 To 空间时，如果 To 空间已经使用了超过 25%，则这个对象直接晋升到老生代空间中。

- Mark-Sweep 和 Mark-Compact

  V8 在老生代中主要采用了 Mark-Sweep 和 Mark-Compact 相结合的方式进行垃圾回收。

  Mark-Sweep 是标记清除的意思。**Mark-Sweep 在标记阶段遍历堆中的所有对象，并标记活着的对象，在随后的清除阶段中，只清除没有被标记的对象。**图 5-6 为 Mark-Sweep 在老生代空间中标记后的示意图，黑色部分标记为死亡的对象。

  ![image-20210605233713191](https://raw.githubusercontent.com/silence/blog/assets/assets/20210605233713.png)

  Mark-Sweep 最大的问题是在进行一次标记清除回收后，内存空间会出现不连续的状态。这种内存碎片会对后续的内存分配造成问题。

  为了解决 Mark-Sweep 的内存碎片的问题，Mark-Compact 被提出来。Mark-Compact 是标记整理的意思。**它与 Mark-Sweep 的区别在于对象在标记死亡后，在整理的过程中，将活着的对象往一端移动，移动完成后，直接清理掉边界外的内存。**

  ![image-20210605234151358](https://raw.githubusercontent.com/silence/blog/assets/assets/20210605234151.png)

  完成移动后，就可以直接清除最右边的存活对象后面的内存区域完成回收。

  表 5-1 是目前介绍到的 3 种主要垃圾回收算法的简单对比。

  ![image-20210605234326096](https://raw.githubusercontent.com/silence/blog/assets/assets/20210605234326.png)

  从表 5-1 中可以看到，在 Mark-Sweep 和 Mark-Compact 之间，由于 Mark-Compact 需要移动对象，所以它的执行速度不可能很快，所以在取舍上，V8 主要使用 Mark-Sweep，在空间不足以对从新生代中晋升过来的对象进行分配时才使用 Mark-Compact。

- Incremental Marking

  为了降低全堆垃圾回收带来的停顿时间，V8 先从标记阶段入手，将原本要一口气停顿完成的动作改为增量标记（incremental marking），**也就是拆分为许多小 “步进”，每做完一“步进”就让 JavaScript 应用逻辑执行一小会儿，垃圾回收与应用逻辑交替执行直到标记阶段完成。**图 5-8 为增量标记示意图。

  ![image-20210605234811939](https://raw.githubusercontent.com/silence/blog/assets/assets/20210605234812.png)

  V8 在经过增量标记的改进后，垃圾回收的最大停顿时间可以减少到原本的 1/6 左右。

  V8 后续还引入了延迟清理（lazy sweeping）与增量式整理（incremental compaction），让清理与整理动作也变成增量式的。同时还计划引入并行标记与并行整理，进一步利用多核性能降低每次停顿的时间。

#### 5.2 高效使用内存

##### 5.2.3 小结

在正常的 JavaScript 执行中，无法立即回收的内存有**闭包**和**全局变量引用**这两种情况。由于 V8 的内存限制，要十分小心此类变量是否无限制地增加，因为它会导致老生代中的对象增多。

#### 5.3 内存指标

##### 5.3.1 查看内存使用情况

1. 查看进程的内存占用

```bash
$ node
> process.memoryUsage()
{
	rss: 13823233,
	heapTotal: 62345000,
	heapUsed: 27567439
}
```

rss 是 resident set size 的缩写，即进程的常驻内存部分。进程的内存总共有几部分，一部分是 rss，其余部分在交换区（swap）或者文件系统（filesystem）中。

##### 5.3.2 堆外内存

通过 process.momoryUsage() 的结果可以看到，堆中的内存用量总是小于进程的常驻内存用量，这意味着 Node 中的内存使用并非都是通过 V8 进行分配的。我们将那些不是通过 V8 分配的内存称为 堆外内存。

Buffer 对象不同于其他对象，它不经过 V8 的内存分配机制，所以也不会有堆内存的大小限制。

这意味着利用堆外内存可以突破内存限制的问题。

##### 5.3.3 小结

Node 的内存构成主要由**通过 V8 进行分配的部分**和 **Node 自行分配的部分**。受 V8 的垃圾回收限制的主要是 **V8 的堆内存**。

#### 5.4 内存泄漏

##### 5.4.1 慎将内存当做缓存

1. 缓存限制策略

如控制对象键值对的数量，超过数量就删除旧的键值对。或者使用 LRU 算法的缓存，如 node-lru-cache。

2. 缓存的解决方案

可以采用进程外缓存，进程自身不存储状态。如 Redis 和 Memcached。

##### 5.4.2 关注队列状态

队列中生产-消费模型中经常充当中间产物。这是一个容易忽略的情况，因为在大多数应用场景下，消费的速度远远大于生产的速度，内存泄漏不易产生。但是一旦消费速度低于生产速度，将会形成堆积。

举个实际例子，有的应用会收集日志。如果欠缺考虑，也会会采用数据库来记录日志。数据库构建在文件系统之上，写入效率远远低于文件直接写入，于是会形成数据库写入操作的堆积，而 JavaScript 中相关的作用域也不会得到释放，内存占用不会回落，从而出现内存泄漏。

遇到这种场景，表层的解决方案是换用消费速度更高的技术。在日志收集的案例中，换用文件写入日志的方式会更高效。

深度的解决方案应该是**监控队列的长度**，一旦堆积，应当通过监控系统产生报警并通知相关人员。另一个解决方案是**任意异步调用都应该包含超时机制**，一旦在限定的时间内未完成相应，通过回调函数传递超时异常，使得任意异步调用的回调都具备可控的响应时间，给消费速度一个下限值。

#### 5.5 内存泄漏排查

如工具 node-heapdump 和 node-memwatch 等。

#### 5.6 大内存应用

在 Node 中，不可避免地还是会存在操作大文件的场景。由于 Node 的内存限制，操作大文件也需要小心，好在 Node 提供了 stream 模块用于处理大文件。

由于 V8 的内存限制，我们无法通过 fs.readFile() 和 fs.writeFile() 直接进行大文件的操作，而改用 fs.createReadStream() 和 fs.createWriteStream() 方法通过流的方式实现大文件的操作。下面的代码展示了如何读取一个文件，然后将数据写入到另一个文件的过程：

```javascript
var reader = fs.createReadStream('in.txt');
var writer = fs.createWriteStream('out.txt');
reader.on('data', function(chunk) {
  writer.write(chunk);
})
reader.on('end', function() {
  writer.end();
})
```

由于读写模型固定，上述方案有更简洁的方式，具体如下所示：

```javascript
var reader = fs.createReadStream('in.txt');
var writer = fs.createWriteStream('out.txt');
reader.pipe(writer)
```

可读流提供了管道方法 pipe()，封装了 data 事件和写入操作。通过流的方式，上述代码不会受到 V8 内存限制的影响，有效地提高了程序的健壮性。

如果不需要进行字符串层面的操作，则不需要借助 V8 来处理，可以尝试进行纯粹的 Buffer 操作，这不会受到 V8 堆内存的限制。但是这种大片使用内存的情况依然要小心，**即使 V8 不限制堆内存的大小，物理内存依然有限制。**

### 第 6 章 理解 Buffer

由于应用场景不同，在 Node 中，应用需要处理网络协议、操作数据库、处理图片、接收上传文件等，在网络流和文件的操作中，还要处理大量二进制数据，JavaScript 自有的字符串远远不能满足这些需求，于是 Buffer 对象应运而生。

#### 6.1 Buffer 结构

Buffer 是一个像 Array 的对象，但它主要用于操作字节。下面我们从模块结构和对象结构的层面上来认识它。

##### 6.1.1 模块结构

Buffer 是一个典型的 JavaScript 与 C++ 结合的模块，它将性能相关部分用 C++ 实现，将非性能相关部分用 JavaScript 实现，如图 6-1 所示。

![image-20210606121025335](https://raw.githubusercontent.com/silence/blog/assets/assets/20210606121025.png)

第 5 章揭示了 Buffer 所占用的内存不是通过 V8 分配的，属于堆外内存。

##### 6.1.2 Buffer 对象

Buffer 对象类似于数组，它的元素为 16 进制的两位数，即 0 到 255 的数值。示例代码如下所示：

```javascript
var str = "深入浅出node.js"；
var buf = new Buffer(str, "utf-8");
console.log(buf);
// => <Buffer e6 b7 b1 e5 85 a5 e6 b5 85 e5 87 ba 6e 6f 64 65 2e 6a 73>
```

由上面的示例可见，不同编码的字符串占用的元素个数各不相同，上面代码中的中文字在 UTF-8 编码下占用 3 个元素，字母和半角标点符号占用 1 个元素。

Buffer 受 Array 类型影响很大，可以访问 length 属性得到长度，也可以通过下标访问元素。

给 Buffer 元素的赋值如果小于 0，就将该值逐次加 256，直到得到一个 0 到 255 之间的整数。如果得到的数值大于 255，就逐次减 256，直到得到 0 ~ 255 区间内的数值。**如果是小数，舍弃小数部分，只保留整数部分**。

##### 6.1.3 Buffer 内存分配

简单而言，真正的内存是在 Node 的 C++ 层面提供的，JavaScript 层面只是使用它，当进行小而频繁的 Buffer 操作时，采用 slab 的机制进行预先申请和事后分配，使得 JavaScript 到操作系统之间不必有过多的内存申请方面的系统调用。对于大块的 Buffer 而言，则直接使用 C++ 层面提供的内存，而不需细腻的分配操作。

#### 6.2 Buffer 的转换

Buffer 对象可以与字符串之间相互转换。目前支持的字符串编码类似有如下这几种。

- ASCII
- UTF-8
- UTF-16LE/UCS-2
- Base64
- Binary
- Hex

##### 6.2.1 字符串转 Buffer

由于可以不断写入内存到 Buffer 对象中，并且每次写入可以指定编码，所以 Buffer 对象中可以存在多种编码转化后的内存。需要小心的是，每种编码所用的字节长度不同，将 Buffer 反转回字符串时需要谨慎处理。

##### 6.2.2 Buffer 转字符串

实现 Buffer 向字符串的转换也十分简单

```javascript
buf.toString([encoding], [start], [end])
```

比较精巧的是，可以设置 encoding（默认为 UTF-8）、start、end 这 3 个参数实现整体或局部的转换。

### 第 7 章 网络编程

Node 提供了 net、dgram、http、https 这 4 个模块，分别用于处理 TCP、UDP、HTTP、HTTPS，适合用于服务器和客户端。

#### 7.1 构建 TCP 服务

##### 7.1.1 TCP

TCP 全名为传输控制协议，在 OSI 模型中属于传输层协议。

七层协议示意图如图 7-1 所示：

![image-20210606132608500](https://raw.githubusercontent.com/silence/blog/assets/assets/20210606132608.png)

TCP 是面向连接的协议，其显著的特征是在传输之前需要 3 次握手形成会话，如图 7-2 所示。

![image-20210606133456500](https://raw.githubusercontent.com/silence/blog/assets/assets/20210606133456.png)

只有会话形成之后，服务器端和客户端之间才能互相发送数据。在创建会话的过程中，服务器端和客户端分别提供一个套接字，这两个套接字共同形成一个连接。服务器端和客户端则通过套接字实现两者之间连接的操作。

##### 7.1.2 创建 TCP 服务器端

```javascript
var net = require('net');

var server = net.createServer(function(socket) {
  // 新的连接
  socket.on('data', function(data) {
    socket.write('你好');
  })
  
  socket.on('end', function() {
    console.log('连接断开')
  })
  socket.write('欢迎光临 《深入浅出 Node.js》示例')
})

server.listen(8124, function() {
  console.log('server bound');
})
```

我们可以利用 Telnet 工具作为客户端对刚才创建的简单服务器进行会话交流，相关代码如下：

```bash
$ telnet 127.0.0.1 8124
Trying 127.0.0.1...
Connected to localhost.
欢迎光临《深入浅出Node.js》示例：
你好
```

除了端口外，同样我们也可以对 Domain Socket 进行监听，代码如下：

```
server.listen('/tmp/echo.sock');
```

通过 nc 工具进行会话，测试上面构建的 TCP 服务的代码如下所示：

```bash
$ nc -U /tmp/echo.sock
欢迎光临《深入浅出Node.js》示例：
你好
```

通过 net 模块自行构造客户端进行会话，测试上面构建的 TCP 服务的代码如下所示：

```javascript
var net = require('net');
var client = net.connect({ prot: 8124 }, function() {
  // 'connect' listener
  console.log('client connected');
  client.write('world!\r\n');
})

client.on('data', function(data) {
  console.log(data.toString());
  client.end();
})

client.on('end', function() {
  console.log('client disconnected');
})
```

将以上客户端代码存为 client.js 并执行，如下所示：

```bash
$ node client.js
client connected
欢迎光临《深入浅出Node.js》示例：
你好
client disconnected
```

其结果与使用 Telnet 和 nc 的会话结果并无差异。如果是 Domain Socket，在填写选项时，填写 path 即可，代码如下：

```javascript
var client = net.connect({ path: '/tmp/echo.sock' });
```

##### 7.1.3 TCP 服务的事件

TCP 针对网络中的小数据包有一定的优化策略：Nagle 算法。如果每次只送一个字节的内容而不优化，网络中将充满只有极少数有效数据的数据包，将十分浪费网络资源**。Nagle 算法针对这种情况，要求缓冲去的数据达到一定数量或者一定时间后才将其发出，所以小数据包将会被 Nagle 算法合并，以此来优化网络。**这种优化虽然使网络带宽被有效地使用，但是数据有可能被延迟发送。

在 Node 中，由于 TCP 默认启用了 Nagle 算法，可以调用 socket.setNoDelay(true) 去掉 Nagle 算法，使得 write() 可以立即发送数据到网络中。

另一个需要注意的是，尽管在网络的一端调用 write() 会触发另一端的 data 事件，但是并不意味着每次 write() 都会触发一次 data 事件，在关闭掉 Nagle 算法后，另一端可能会将接收到的多个小数据包合并，然后只触发一次 data 事件。

#### 7.2 构建 UDP 服务

UDP 又称用户数据包协议，与 TCP 一样同属于网络传输层。UDP 与 TCP 最大的不同是 **UDP 不是面向连接的**。**TCP 中连接一旦建立，所有的会话都基于连接完成**，客户端如果要与另一个 TCP 服务通信，需要另创建一个套接字来完成连接。**但在 UDP 中，一个套接字可以与多个 UDP 服务通信**，它虽然提供面向事务的简单不可靠信息传输服务，在网络差的情况下存在丢包严重的问题，但是由于它无须连接，资源消耗低，处理快速且灵活，所以常常应用在那种偶尔丢一两个数据包也不会产生重大影响的场景，比如音频、视频等。UDP 目前应用很广泛，DNS 服务即是基于它实现的。

##### 7.2.1 创建 UDP 套接字

创建 UDP 套接字十分简单，UDP 套接字一旦创建，既可以作为客户端发送数据，也可以作为服务器端接收数据。

```javascript
var dgram = require('dgram');
var socket = dgram.createSocket('udp4');
```

##### 7.2.2 创建 UDP 服务器端

若想让 UDP 套接字接收网络消息，只要调用 dgram.bind(port, [address]) 方法对网卡和端口进行绑定即可。以下为一个完整的服务器端示例：

```javascript
var dgram = require('dgram');
var server = dgram.createSocket('udp4');

server.on('message', function(msg, rinfo) {
  console.log('server got: ' + msg + " from " + rinfo.address + ":" + rinfo.port);
})

server.on('listening', function() {
  var address = server.address();
  console.log("server listening " + address.address + ":" + address.port)
})

server.bind(41234)
```

该套接字将接收所有网卡上 41234 端口上的消息。在绑定完成后，将触发 listening 事件。

##### 7.2.3 创建 UDP 客户端

接下来我们创建一个客户端与服务器端进行会话。

```javascript
var dgram = require('dgram');

var message = new Buffer('深入浅出Node.js');
var client = dgram.createSocket('udp4');
client.send(message, 0, message.length, 41234, "localhost", function(err, bytes) {
  client.close();
})
```

保存为 client.js 并执行，服务器端的命令行将会有如下输出：

```javascript
$ node server.js
server listening 0.0.0.0:41234
server got: 深入浅出Node.js from 127.0.0.1:58682
```

这些参数分别为要发送的 Buffer、Buffer的偏移、Buffer的长度、目标端口、目标地址、发送完成后的回调。与 TCP 套接字的 write() 相比，send() 方法的参数列表相对复杂，但是它更灵活的地方在于可**以随意发送数据到网络中的服务器端**，而 TCP 如果要发送数据给另一个服务器端，则需要**重新通过套接字构造新的连接**。

#### 7.3 构建 HTTP 服务

Node 提供了基本的 http 和 https 模块用于 HTTP 和 HTTPS 的封装，对于其他应用层协议的封装，也能从社区中轻松找到其实现。

##### 7.3.1 HTTP

HTTP 的全称是超文本传输协议，英文写作 HyperText Transfer Protocol。HTTP 构建在 TCP 之上，属于应用层协议。**在 HTTP 的两端是服务器和浏览器，即著名的 B/S 模式**，如今精彩纷呈的 Web 即是 HTTP 的应用。

##### 7.3.2 http 模块

Node 的 http 模块包含对 HTTP 处理的封装。在 Node 中，HTTP 服务继承自 TCP 服务器（net 模块），它能够与多个客户端保存连接。

HTTP 服务与 TCP 服务模型有区别的地方在于，在开启 keepalive 后，一个 TCP 会话可以用于多次请求和响应。TCP 服务以 connection 为单位进行服务，HTTP 服务以 request 为单位进行服务。**http 模块即是将 connection 到 request 的过程进行了封装**，示意图如图 7-3 所示：

![image-20210606143626129](https://raw.githubusercontent.com/silence/blog/assets/assets/20210606143626.png)

除此之外，http 模块将连接所用套接字的读写抽象为 ServerRequest 和 ServerResponse 对象，它们分别对应请求和响应操作。在请求产生的过程中，http 模块拿到连接中传来的数据，调用二进制模块 http_parser 进行解析，在解析完请求报文的报头后，触发 request 事件，调用用户的业务逻辑。该流程的示意图如图 7-4 所示。

![image-20210606143933649](https://raw.githubusercontent.com/silence/blog/assets/assets/20210606143933.png)

3. HTTP 服务的事件

服务器也是一个 EventEmitter 实例。

- checkContinue 事件：某些客户端发送较大的数据时，并不会将数据直接发送，而是先发送一个头部带有 Expect: 100-continue 的请求到服务器，服务器将会触发 checkContinue 事件；如果没有为服务器监听这个事件，服务器将会自动响应客户端 100 Continue 的状态码，表示接受数据上传；如果不接受数据的较多时，响应客户端 400 Bad Request 拒绝客户端继续发生数据即可。需要注意的是，当该事件发生时不会触发 request 事件，两个事件之间互斥。当客户端收到 100 Continue 后重新发起请求时，才会触发 request 事件。
- upgrade 事件：当客户端要求升级连接的协议时，需要和服务器端协商，客户端会在请求头中带上 Upgrade 字段，服务器端会在接收到这样的请求时触发该事件。

##### 7.3.3 HTTP 客户端

2. HTTP 代理

如同服务器端的实现一般，http 提供的 ClientRequest 对象也是基于 TCP 层实现的，在 keepalive 的情况下，一个底层会话连接可以多次用于请求。为了重用 TCP 连接，http 模块包含一个默认的客户端代理对象 http.globalAgent。它对每个服务器端（host+port）创建的连接进行了管理，默认情况下，通过 ClientRequest 对象对同一个服务器端发起的 HTTP 请求最多可以创建 5 个连接。它的实质是一个连接池，示意图如图 7-5 所示。

![image-20210607135014582](https://raw.githubusercontent.com/silence/blog/assets/assets/20210607135014.png)

调用 HTTP 客户端同时对一个服务器发起 10 次 HTTP 请求时，其实质只有 5 个其过去了处于并发状态，**后续的请求需要等待某个请求完成服务后才真正发出。这与浏览器对同一个域名有下载连接数的限制是相同的行为**。

3. HTTP 客户端事件

- upgrade：客户端向服务器端发起 Upgrade 请求时，如果服务器端响应了 101 Switching Protocols 状态，客户端将会触发该事件。
- continue: 客户端向服务器端发起 Expect: 100-continue 头信息，以试图发送较大数据量，如果服务器端响应 100 Continue 状态，客户端将触发该事件。

#### 7.4 构建 WebSocket 服务

WebSocket 协议与 Node 之间的配合堪称完美。

- WebSocket 客户端**基于事件的编程模型与 Node 中自定义事件相差无几**。
- WebSocket **实现了客户端与服务器端之间的长连接，而 Node 事件驱动的方式十分擅长与大量的客户端保存高并发连接**。

除此之外，WebSocket 与传统 HTTP 有如下好处。

- **客户端与服务器端只建立一个 TCP 连接，可以使用更少的连接**。
- WebSocket **服务器端可以推送数据到客户端**，这远比 HTTP 请求响应模式更灵活、更高效。
- **有更轻量级的协议头，减少数据传送量**。

##### 7.4.2 Websocket 数据传输

在握手顺利完成后，当前连接将不再进行 HTTP 的交互，而是开始 WebSocket 的数据帧协议，实现客户端与服务器端的数据交换。图 7-6 为协议升级过程示意图。

![image-20210612121342181](https://raw.githubusercontent.com/silence/blog/assets/assets/20210612121342.png)

我们以客户端发送 hello world! 到服务器端，服务器端回以 yakexi 作为一个流程来研究数据帧协议的实现过程。

图 7-7 中为 Websocket 数据帧的定义，每 8 位为一列，也即 1 个字节。其中每一位都有它的意义。

![image-20210612121943576](https://raw.githubusercontent.com/silence/blog/assets/assets/20210612121943.png)

- fin：如果这个数据帧是最后一帧，这个 fin 位为 1，其余情况为 0。当一个数据没有被分为多帧时，它既是第一帧也是最后一帧。
- rsv1、rsv2、rsv3：各为 1 位长，3 个标识符用于扩展，当有已协商的扩展时，这些值可能为 1，其余情况为 0。
- opcode：长为 4 位的操作码，可以用来表示 0 到 15 的值，用于解释当前数据帧。 0 表示附加数据帧，1 表示文本数据帧， 2 表示二进制数据帧，8 表示发送一个连接关闭的数据帧，9 表示 ping 数据帧，10 表示 pong 数据帧，其余值暂时没有定义。**ping 数据帧和 pong 数据帧用于心跳检测，当一端发送 ping 数据帧时，另一端必须发送 pong 数据帧作为响应，告知对方这一端仍然处于响应状态。**
- masked：表示是否进行掩码处理，长度为 1。客户端发送给服务器端时为 1，服务器端发送给客户端时为 0。
- payload length：一个 7、7+16 或 7+64 位长的数据位，标识数据的长度，如果值在 0 ~ 125 之间，那么该值就是数据的真实长度；如果值是 126，则后面 16 位的值是数据的真实长度；如果值是 127，则后面 64 位的值是数据的真实长度。
- masking key：当 masked 为 1 时存在，是一个 32 位长的数据位，用于解密数据。
- payload data：我们的目标数据，位数为 8 的倍数。

客户端发送消息时，需要构造一个或多个数据帧协议报文。由于 hello world! 较短，不存在分割为多个数据帧的情况，又由于 hello world! 会以文本的方式发送，它的 payload length 长度为 96（12 字节 x 8 位/字节），二进制表示为 1100000。所以报文应当如下：

```javascript
fin(1) + res(000) + opcode(0001) + masked(1) + payload length(1100000) + masking key(32 位) + payload data(hello world! 加密后的二进制)
```

当以文本方式发送时，文本的编码为 utf-8，由于这里发送的不存在中文，所以一个字符占一个字节，即 8 位。

客户端发送消息后，服务器端在 data 事件中接收到这些编码数据，然后解析为相应的数据帧，再以数据帧的格式，通过掩码将真正的数据解密出来，然后触发 onmessage() 执行，如下所示：

```javascript
socket.onmessage = function (event) {
  // TODO: event.data
}
```

服务器端再回复 yakexi 的时候，剩下的事情就是无须掩码，其余相同，如下所示：

```javascript
fin(1) + res(000) + opcode(0001) + masked(0) + payload length(110000) + payload data(yakexi 的二进制)
```

**这里的行为与纯 TCP 连接的行为十分类似，近似地可以理解为 TCP 客户端套接字的 connect 事件和 data 事件。**

#### 7.5 网络服务与安全

- SSL (Secure Socket Layer) 安全套接层
- TLS (Transport Layer Security) 安全传输层协议

Node 在网络安全上提供了 3 个模块，分别为 crypto、tls、https。其中 crypto 主要用于加密解密，SHA1、MD5 等加密算法都在其中有体现，在这里我们不用再提。真正用于网络的是另外两个模块，**tls 模块提供了与 net 模块类似的功能，区别在于它建立在 TLS/SSL 加密的 TCP 连接上**。对于 https 而言，它完全与 http 模块接口一致，区别也仅在于它建立于安全的连接之上。

### 第 8 章 构建 Web 应用

#### 8.1 基础功能

##### 8.1.4 Cookie

Cookie 的处理分为如下几步。

- 服务器向客户端发送 Cookie。
- 浏览器将 Cookie 保存。
- 之后每次浏览器都会将 Cookie 发想服务器端。

相应的 Cookie 值在 Set-Cookie 字段中。它的格式与请求中的格式不太相同，规范中对它的定义如下所示：

```bash
Set-Cookie: name=value; Path=/; Expires=Sun, 23-Apr-23 09:01:35 GMT; Domain=.domain.com;
```

其中 name=value 是必须包含的部分，其余部分皆是可选参数。这些可选参数将会影响浏览器在后续将 Cookie 发送给服务器端的行为。以下为主要的几个选项。

- path 表示这个 Cookie 影响到的路径，**当前访问的路径不满足该匹配时，浏览器则不发送这个 Cookie**。
- Expires 和 Max-Age 是用来告知浏览器这个 Cookie 何时过期的，如果不设置该选项，在关闭浏览器时会丢失掉这个 Cookie。如果设置了过期时间，浏览器将会把 Cookie 内容写入到磁盘中并保存，下次打开浏览器依旧有效。**Expires 的值是一个 UTC 格式的时间字符串，告知浏览器此 Cookie 何时将过期**，**Max-Age 则告知浏览器此 Cookie 多久后过期**。前者一般而言不存在问题，但是如果服务器端的时间和客户端的时间不能匹配，这种时间设置就会存在偏差。为此，Max-Age 告知浏览器这条 Cookie 多久之后过期，而不是一个具体的时间点。
- **HttpOnly** **告知浏览器不允许通过脚本 document.cookie 去更改这个 Cookie 值**，事实上，设置 HttpOnly 之后，这个值在 document.cookie 中不可见。但是在 HTTP 请求的过程中，依然会发送这个 Cookie 到服务器端。
- Secure。**当 Secure 值为 true 时，在 HTTP 中是无效的，在 HTTPS 中才有效**，表示创建的 Cookie 只能在 HTTPS 连接中被浏览器传递到服务器端进行会话验证，如果是 HTTP 连接则不会传递该信息，所以很难被窃听到。

##### 8.1.5 Session

通过 Cookie ，浏览器和服务器可以实现状态的记录。但是 Cookie 并非是完美的，前文提及的体积过大就是一个显著的问题，最为严重的问题是 Cookie 可以在前后端进行修改，因此数据就极容易被篡改和伪造。综上所述，Cookie 对于敏感数据的保护是无效的。

为了解决 Cookie 敏感数据的问题，Session 应运而生。S**ession 的数据只保留在服务器端，客户端无法修改**，这样数据的安全性得到一定的保证，数据也无须在协议中每次都被传递。

将每个客户和服务器中的数据一一对应起来，这里有两种常见的实现方式。

- 第一种：**基于 Cookie 来实现用户和数据的映射**。

  一旦服务器端启用了 Session，它将约定一个键值作为 Session 的口令，这个值可以随意约定，比如 Connect 默认采用 connect_uid，Tomcat 会采用 jsesessionid 等。一旦服务器检查到用户请求 Cookie 中没有携带该值，它就会为之生成一个值，这个值是**唯一且不重复的值**，并设定超时时间。

  每个请求到来时，检查 Cookie 中的口令与服务器端的数据，如果过期，就重新生成。

- 第二种：通过查询字符串来实现浏览器端和服务器端数据的对应

- 还有一种比较有趣的处理 Session 的方式是利用 HTTP 请求头中的 ETag，同样对于更换浏览器和电脑后也是无效的

##### 8.1.6 缓存

为了提高性能，这里有几条关于缓存的规则。

- 添加 Expires 或 Cache-Control 到报文头中。
- 配置 ETags。
- 让 Ajax 可缓存

通常来说，POST、DELETE、PUT 这类带行为新的请求操作一般不做任何缓存，大多数缓存只应用在 GET 请求中。使用缓存的流程如图 8-1 所示。

![image-20210612193610189](https://raw.githubusercontent.com/silence/blog/assets/assets/20210612193610.png)

在第二次请求时，它将会对本地文件进行检查，如果不能确定这份本地文件是否可以直接使用，它将会发起一次条件请求。所谓条件请求，就是在普通的 GET 请求报文中，附带 If-Modified-Since 字段，如下所示：

```javascript
If-Modified-Since: Sun, 03 Feb 2013 06:01:12 GMT
```

它将询问服务器端是否有更新的版本，本地文件的最后修改时间。如果服务器端没有新的版本，只需响应一个 304 状态码，客户端就使用本地版本。如果服务器端有新的版本，就将新的内容发送给客户端，客户端放弃本地版本。

这里的条件请求采用时间戳的方式实现，但是时间戳有一些缺陷存在。

- **文件的时间戳改动但内容并不一定改动**。
- 时间戳只能精确到秒级别，更新频繁的内容将无法生效

为此 HTTP 1.1 中引入了 ETag 来解决这个问题。ETag 的全称是 Entity Tag，**由服务器端生成，服务器端可以决定它的生成规则。**如果根据文件内容生成散列值，那么条件请求将不会受到时间戳改动造成的带宽浪费。

与 If-Modified-Since/Last-Modified 不同的是，ETag 的请求和响应是 If-None-Match/ETag

最后的方案是连条件请求都不用发起，让浏览器明确地将内容缓存起来，在响应里设置 Expires 或 Cache-Control 头。

HTTP 1.0 时，在服务器端设置 Expires 可以告知浏览器要缓存文件内容。

**Expires 是一个 GMT 格式的时间字符串。浏览器在接到这个过期值后，只要本地还存在这个缓存文件，在到期时间之前它都不会再发起请求。**但是 Expires 的缺陷在于浏览器与服务器之间的时间可能不一致，这可能会带来一些问题，比如文件提前过期，或者到期后并没有被删除。在这种情况下，Cache-Control 以更丰富的形式，实现相同的功能。

```javascript
var handle = function (req, res) {
  fs.readFile(filename, function(err, file) {
    res.setHeader("Cache-Control", "max-age=" + 10 * 365 * 24 * 60 * 60 * 1000);
    res.writeHead(200, "OK");
    res.end(file);
  })
}
```

上面的代码为 Cache-Control 设置了 max-age 值，它比 Expires 优秀的地方在于，Cache-Control 能够避免浏览器端与服务器端时间不同步带来的不一致性问题，只要进行类似倒计时的方式计算过期时间即可。除此之外，Cache-Control 的值还能设置 public、private、no-cache、no-store 等能够更精细地控制缓存的选项。

由于在 HTTP 1.0 时还不支持 max-age，如今的服务器端在模块的支持下多半同时对 Expires 和 Cache-Control 进行支持。**在浏览器中如果两个值同时存在，且被同时支持时，max-age 会覆盖 Expires。**

- 清除缓存

因为浏览器是根据 URL 进行缓存，那么一旦内容有所更新时，我们就让浏览器发起新的 URL 请求，使得新内容能够被客户端更新。

大体来说，根据文件内容的 hash 值进行缓存淘汰更加高效，因为文件内容不一定随着 Web 应用的版本而更新，而内容没有更新时，版本号的改动导致的更新毫无意义，因此以文件内容形成的 hash 值更精准。

#### 8.2 数据上传

##### 8.2.3 附件上传

有一种特殊表单往往需要用户直接提交文件。特殊表单与普通表单的差异在于该表单中可以含有 file 类型的控件，以及需要指定表单属性 enctype 为 multipart/form-data。

```html
<form action="/upload" method="post" enctype="multipart/form-data">
  <label for="username">Username:</label>
  <input type="text" name="username" id="username" />
  <label for="file">FileName:</label>
  <input type="file" name="file" id="file" />
  <br />
  <input type="submit" name="submit" value="Submit" />
</form>
```

浏览器在遇到 mutipart/form-data 表单提交时，构造的请求报文与普通表单完全不同。首先它的报头中最为特殊的如下所示：

```bash
Content-Type: multipart/form-data; boundary=AaB03x
Content-Length: 18231
```

它代表本次提交的内容是由多部分构成的，其中 boundary=AaB03x指定的是每部分内容的分界符，AaB03x 是随机生成的一段字符串，**报文体的内容将通过在它前面添加 -- 进行分割，报文结束时在它前后都加上 -- 表示结束**。另外，Content-Length 的值必须确保是报文体的长度。

##### 8.2.4 数据上传与安全

1. 内存限制

要解决这个问题主要有两个方案。

- 限制上传内容的大小，一旦超过限制，停止接收数据，并响应 400 状态码。
- 通过流式解析，将数据流导向到磁盘中，Node 只保留文件路径等小数据。

2. CSRF

CSRF 的全称是 Cross-Site Request Forgery，中文的意思为跨站请求伪造。

为了详细解释 CSRF 攻击是怎样一个过程，这里以一个留言的例子来说明。假设某个网站有这样一个留言程序，提交留言的接口如下所示：

```
http://domain_a.com/guestbook
```

用户通过 POST 提交 content 字段就能成功留言。服务器端会自动从 Session 数据中判断是谁提交的数据，补足 username 和 updateAt 两个字段后向数据库中写入数据，如下所示：

```javascript
function (req, res) {
  var content = req.body.content || '';
  var username = req.session.username;
  var feedback = {
    username: username,
    content: content,
    updateAt: Date.now()
  };
  db.save(feedback, function(err) {
    res.writeHead(200);
    res.end('Ok');
  })
}
```

正常情况下，谁提交的留言，就会在列表中显示谁的信息。如果某个攻击则发现了这里的接口存在 CSRF 漏洞，那么他就可以在另一个网站（http://domain_b.com/attack） 上构造一个表单提交，如下所示：

```html
<form id="test" method="POST" action="http://domain_a.com/guestbook">
  <input type="hidden" name="content" value="vim 是这个世界上最好的编辑器" />
</form>
<script type="text/javascript">
	$(function() {
    $("#test").submit();
  })
</script>
```

这种情况下，攻击者只要引诱某个 domain_a 的登陆用户访问这个 domain_b 的网站，就会自动提交一个留言。由于在提交到 domain_a 的过程中，浏览器会将 domain_a 的Cookie 发送到服务器，尽管这个请求是来自 domain_b 的，但是服务器并不知情，用户也不知情。

以上过程就是一个 CSRF 攻击的过程。这里的示例仅仅是一个留言的漏洞，如果出现漏洞的是转账的接口，那么其危害程度可想而知。

解决 CSRF 攻击的方案有添加随机值的方式，如下所示：

```javascript
var generateRandom = function(len) {
  return crypto.randomBytes(Math.ceil(len * 3 / 4))
  	.toString('base64')
  	.slice(0, len)
}
```

也就是说，为每个请求的用户，在 Session 中赋予一个随机值，如下所示：

```javascript
var csrfToken = req.session._csrf || (req.session._csrf = generateRandom(24))
```

在做页面渲染的过程中，将这个 _csrf 值告知前端。

由于该值是一个随机值，攻击者构造出相同的随机值的难度相当大，所以我们只需要在接收端做一次校验就能轻易地识别出该请求是否为伪造的，如下所示：

```javascript
function (req, res) {
  var token = req.session._csrf || (req.session._csrf = generateRandom(24));
  
  var _csrf = req.body._csrf;
  if (token !== _csrf) {
    res.writeHead(403);
    res.end('禁止访问');
  } else {
    handle(req, res);
  }
}
```

_csrf 字段也可以存在于查询字符串或者请求头中。

#### 8.4 中间件

对于 Web 应用而言，我们希望开发者不用接触到这么多细节性的处理，为此我们引入中间件（middleware）来简化和隔离这些基础设施与业务逻辑之间的细节，让开发者能够关注在业务的开发上，以达到提升开发效率的目的。

在最早的中间件的定义中，**它是一种在操作系统上为应用软件提供服务的计算机软件**。

它既不是操作系统的一部分，也不是应用软件的一部分，它处于操作系统与应用软件之间，让应用软件更好、更方便地使用底层服务。**如今中间件的含义借指了这种封装底层细节，为上层提供更方便服务的意义，并非限定在操作系统层面**。这里要提到的中间件，就是为我们封装上文提及的所有 HTTP 请求细节处理的中间件，开发者可以脱离这部分细节，专注在业务上。

它的工作模型如图 8-4 所示。

![image-20210612231438806](https://raw.githubusercontent.com/silence/blog/assets/assets/20210612231438.png)

##### 8.4.2 中间件与性能

前文我们添加了强大的中间件组织能力，如果注意到一个现象的话，那就是我们的业务逻辑往往是在最后才执行。为了让业务逻辑提早执行，尽早响应给终端用户，中间件的编写和使用是需要一番考究的。下面是两个主要的能提升的点。

- 编写高效的中间件
- 合理利用路由，避免不必要的中间件执行。

1. 编写高效的中间件

编写高效的中间件其实就是提升单个处理单元的处理速度，以尽早调用 next() 执行后续逻辑。需要知道的事情是，一旦中间件被匹配，那么每个请求都会使该中间件执行一次，哪怕它只浪费 1 毫秒的执行时间，都会让我们的 QPS 显著下降。常见的优化方法有几种。

- **使用高效的方法**。
- **缓存需要重复计算的结果（需要控制缓存用量，避免内存溢出）**
- **避免不必要的计算。**比如 HTTP 报文体的解析，对于 GET 方法完全不需要

2. 合理使用路由

在拥有一堆高效的中间件后，并不意味着每个中间件我们都使用，合理的路由使得不必要的中间件不参与请求处理的过程。这里以一个示例来说明该问题。

假设我们这里有一个静态文件的中间件，它会对请求进行判断，如果磁盘上存在对应的文件，就响应对应的静态文件，否则就交由下游中间件处理，如下所示：

```javascript
var staticFile = function (req, res, next) {
  var pathname = url.parse(req.url).pathname;
  
  fs.readFile(path.join(ROOT, pathname), function (err, file) {
    if (err) {
      return next();
    }
    res.writeHead(200);
    res.end(file)
  })
}
```

如果我们以如下的方式注册路由：

```javascript
app.use(staticFile);
```

那么意味着对 / 路径下的所有 URL 请求都会进行判断。又由于它中间涉及到了磁盘 I/O，如果成功匹配，它的效率还行，但是如果不成功匹配，每次的磁盘 I/O 都是对性能的浪费，使 QPS 直线下降。

对于这种情况，我们需要做的是提升匹配成功率，那么就不能使用默认的 / 路径来进行匹配了，因为它的误伤率太高。给它添加一个更好的路由路径是个不错的选择，如下所示：

```javascript
app.use('/public', staticFile);
```

这样只有 /public 路径会匹配上，其余路径根本不会涉及该中间件。

#### 8.5 页面渲染

##### 8.5.1 内容响应

1. MIME

MIME 的全称是 Multipurpose Internet Mail Extensions ，从名字可看出，它最早用于电子邮件，后来也应用到浏览器中。不同的文件类型具有不同的 MIME 值，如 JSON 文件的值为 application/json、XML 文件的值为 application/xml、PDF 文件的值为 application/pdf。

2. 附件下载

在一些场景下，无论相应的内容是什么样的 MIME 值，需求中并不要求客户端去打开它，只需弹出并下载它即可。为了满足这种需求，Content-Disposition 字段应声登场。当内容只需即时查看时，它的值为 inline，当数据可以存为附件时，它的值为 attachment。**另外，Content-Disposition 字段还能通过参数指定保存时应该使用的文件名。**

```
Content-Disposition: attachment; filename="filename.ext"
```

### 第 9 章 玩转进程

#### 9.1 服务模型的变迁

##### 9.1.1 石器时代：同步

最早的服务器，其指向模型是同步的，它的服务模式是一次只为一个请求服务，所有请求都得按次序等待服务。

##### 9.1.2 青铜时代：复制进程

##### 9.1.3 白银时代：多线程

多线程所面临的并发问题只能说比多进程略好，**因为每个线程都拥有自己独立的堆栈，这个堆栈都需要占用一定的内存空间。**另外，由于一个 CPU 核心在一个时刻只能做一件事情，操作系统只能通过将 CPU 切分为时间片的方法，让线程可以较为均匀地使用 CPU 资源，**但是操作系统内核在切换线程的同时也要切换线程的上下文，当线程数量过多时，时间将会被耗用在上下文切换中。**所以在大并发量时，多线程结构还是无法做到强大的伸缩性。

##### 9.1.4 黄金时代：事件驱动

由于所有处理都在单线程上进行，影响事件驱动服务模型性能的点在于 CPU 的计算能力，它的上限决定这类服务模型的性能上限，**但它不受多进程或多线程模式中资源上限的影响，可伸缩性远比前两者高**。

#### 9.2 多进程架构

Node 提供了 child_process 模块，并且也提供了 child_process.fork() 函数供我们实现进程的复制。

我们再一次将经典的示例代码存为 worker.js 文件，如下所示：

```javascript
var http = require('http');
http.createServer(function (req, res) {
  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.end('Hello, World\n');
}).listen(Math.round(1 + Math.random()) * 1000), '127.0.0.1');
```

通过 node worker.js 启动它，将会侦听 1000 到 2000 之间的一个随机端口。

将以下代码存为 master.js，并通过 node master.js 启动它：

```javascript
var fork = require('child_process').fork;
var cpus = require('os').cpus();
for (var i = 0; i < cpus.length; i++) {
  fork('./worker.js');
}
```

这段代码代码将会根据当前机器上的 CPU 数量复制出对应 Node 进程数。在 *nix 系统下可以通过 ps aux | grep worker.js 查看到进程的数量，如下所示：

![image-20210613142427616](https://raw.githubusercontent.com/silence/blog/assets/assets/20210613142427.png)

图 9-1 就是**著名的 Master-Worker 模式，又称主从模式**。图 9-1 中的进程分为两种：主进程和工作进程。这是典型的分布式架构中用于并行处理业务的模式，具备较好的**可伸缩性和稳定性**。**主进程不负责具体的业务处理，而是负责调度或管理工作进程，它是趋向于稳定的。**工作进程负责具体的业务处理，因为业务的多种多样，甚至一项业务由多人开发完成，所以工作进程的稳定性值得开发者关注。

![image-20210613142754164](https://raw.githubusercontent.com/silence/blog/assets/assets/20210613142754.png)

通过 fork() 复制的进程都是一个独立的进程，**这个进程中有着独立而全新的 V8 实例。它需要至少 30ms 的启动时间和至少 10 MB 的内存。**尽管 Node 提供了 fork() 供我们复制进程使每个 CPU 内核都使用上，但是依然要切记 fork() 进程是昂贵的。好在 Node 通过事件驱动的方式在单线程上解决了大并发的问题，这里启动多个进程只是为了充分将 CPU 资源利用起来，而不是为了解决并发问题。

##### 9.2.1 创建子进程

child_process 模块给予 Node 可以随意创建子进程（child_process）的能力。它提供了 4 个方法用于创建子进程。

- spawn(): 启动一个子进程来执行命令。
- exec(): 启动一个子进程来执行命令，与 spawn() 不同的是其接口不同，它有一个回调函数获知子进程的状况。
- execFile(): 启动一个子进程来执行可执行文件。
- fork(): 与 spawn() 类似，不同点在于它创建 Node 的子进程只需指定要执行的 JavaScript 文件模块即可。

spawn() 与 exec()、execFile() 不同的是，**后两者创建时可以指定 timeout 属性设置超时时间，一旦创建的进程运行超过设定的时间将会被杀死。**

exec() 与 execFile() 不同的是，exec() 适合执行已有的命令，execFile() 适合执行文件。这里我们以一个寻常命令为例，node worker.js 分别用上述 4 种方法实现，如下所示：

```javascript
var cp = require('child_process');
cp.spawn('node', ['worker.js']);
cp.exec('node worker.js', function(err, stdout, stderr) {
  // some code
})
cp.execFile('worker.js', function(err, stdout, stderr) {
  // some code
})
cp.fork('./worker.js')
```

以上 4 个方法在创建子进程之后均会返回子进程对象。它们的差别可以通过表 9-1 查看。

![image-20210613143956310](https://raw.githubusercontent.com/silence/blog/assets/assets/20210613143956.png)

这里的可执行文件是指可以直接执行的文件，如果是 JavaScript 文件通过 execFile() 运行，它的首行内容必须添加如下代码：

```javascript
#!/usr/bin/env node
```

尽管 4 种创建子进程的方式有些差别，但事实上后面 3 中方法都是 spawn() 的延伸应用。

##### 9.2.2 进程间通信

在 Master-Worker 模式中，要实现主进程管理和调度工作进程的功能，需要主进程和工作进程之间的通信。对于 child_process 模块，创建好了子进程，然后与父子进程间通信是十分容易的。

在前端浏览器中，JavaScript 主线程与 UI 渲染共用同一个线程。执行 JavaScript 的时候 UI 渲染是停滞的，渲染 UI 时，JavaScript 是停滞的，两者相互阻塞。长时间执行 JavaScript 将会造成 UI 停顿不响应。为了解决这个问题，HTML5 提出了 WebWorker API。WebWorker 允许创建工作线程并在后台运行，使得一些阻塞较为严重的计算不影响主线程上的 UI 渲染。它的 API 如下所示：

```javascript
var worker = new Worker('worker.js');
worker.onmessage = function(event) {
  document.getElementById('result').textContent = event.data;
};
```

其中，worker.js 如下所示：

```javascript
var n = 1;
search: while(true) {
  n += 1;
  for (var i = 2; i <= Math.sqrt(n); i += 1)
    if (n % i === 0)
      continue search;
  // found a prime
  postMessage(n);
}
```

主线程与工作线程之间通过 onmessage() 和 postMessage() 进行通信，子进程对象

则由 send() 方法实现主进程向子进程发送数据，message 事件实现收听子进程发来的数据，与 API 在一定程度上相似。通过消息传递内容，而不是共享或直接操作相关资源，这是较为轻量和无依赖的做法。

Node 中对应示例如下所示：

```javascript
// parent.js
var cp = require('child_process');
var n = cp.fork(__dirname + './sub.js');

n.on('message', function(m) {
  console.log('PARENT go message:', m);
});

n.send({ hello: 'world' });

// sub.js
process.on('messge', function(m) {
  console.log('CHILD got message:', m);
})

process.send({ foo: 'bar' })
```

通过 fork() 或者其他 API，创建子进程之后，为了实现父子进程之间的通信，父进程与子进程之间将会创建 IPC 通道。通过 IPC 通道，父子进程之间之间才能通过 message 和 send() 传递消息。

- **进程间通信原理**

IPC 的全称是 Inter-Process Communication，即进程间通信。进程间通信的目的是为了让不同的进程能够互相访问资源并进行协调工作。实现进程间通信的技术有很多，如命名管道、匿名管道、socket、信号量、共享内存、消息队列、Domain Socket 等。Node 中实现 IPC 通道的是管道（pipe）技术。但此管道非彼管道，在 Node 中管道是个抽象层面的称呼，具体细节实现由 libuv 提供，在 Windows 下由命名管道（named pipe）实现，*nix 系统则采用 Unix Domain Socket 实现。表现在应用层上的进程间通信只有简单的 message 事件和 send() 方法，接口十分简洁和消息化。表现在应用层上的进程间通信只有简单的 message 事件和 send() 方法，接口十分简洁和消息化。图 9-2 为 IPC 创建和实现的示意图。

![image-20210613152342612](https://raw.githubusercontent.com/silence/blog/assets/assets/20210613152342.png)

**父进程在实际创建子进程之前，会创建 IPC 通道并监听它，然后才真正创建出子进程**，并通过环境变量（NODE_CHANNEL_FD）告诉子进程这个 IPC 通道的文件描述符。子进程在启动的过程中，根据文件描述符去连接这个已存在的 IPC 通道，从而完成父子进程之间的连接。图 9-3 为创建 IPC 管道的步骤示意图。

![image-20210613152736018](https://raw.githubusercontent.com/silence/blog/assets/assets/20210613152736.png)

建立连接之后的父子进程就可以自由地通信了。由于 IPC 通道是用命名管道或 Domain Socket 创建的，它们与网络 socket 的行为比较类似，属于双向通信。**不同的是它们在系统内核中就完成了进程间的通信，而不用经过实际的网络层，非常高效。**

##### 9.2.3 句柄传递

我们如果想多个进程监听同一个端口。通常的做法是让每个进程监听不同的的端口，其中主进程监听主端口（如 80），主进程对外接收所有的网络请求，再将这些请求分别代理到不同的端口的进程上。示意图如图 9-4 所示。

![image-20210613171239487](https://raw.githubusercontent.com/silence/blog/assets/assets/20210613171239.png)

由于进程每接收到一个连接，将会用掉一个文件描述符，因此代理方案中客户端连接到代理进程，代理进程连接到工作进程的过程需要用掉两个文件描述符。操作系统的文件描述符是有限的，代理方案浪费掉一倍数量的文件描述符的做法影响了系统的扩展能力。

为了解决上述这样的问题，Node 在版本 v0.5.9 引入了进程间发送句柄的功能。send() 方法除了能通过 IPC 发送数据外，还能发送句柄，第二个可选参数就是句柄，如下所示：

```javascript
child.send(message, [sendHandle]);
```

句柄是**一种可以用来标识资源的引用**，它的内部包含了指向对象的文件描述符。比如句柄可以用来标识一个服务器端 socket 对象、一个客户端 socket 对象、一个 UDP 套接字、一个管道等。

发送句柄意味着什么？在前一个问题中，我们可以去掉代理这种方案，使主进程接收到 socket 请求后，将这个 socket 直接发送给工作进程，而不是重新与工作进程之间建立新的 socket 连接来转发数据。文件描述符浪费的问题可以通过这样的方式轻松解决。来看看示例代码：

主进程代码如下所示：

```javascript
var child = require('child_process').fork('child.js');

// Open up the server object and send the handle
var server = require('net').createServer();
server.on('connection', function(socket) {
  socket.end('handled by parent\n');
})
server.listen(1337, function () {
  child.send('server', server)
})
```

子进程代码如下所示：

```javascript
process.on('message', function (m, server) {
  if (m === 'server') {
    server.on('connection', function (socket) {
      socket.end('handled by child\n');
    })
  }
})
```

这个示例中直接将一个 TCP 服务器发送给了子进程。这是看起来不可思议的事情，我们先来测试一番，看看效果如何，如下所示：

```bash
# 先启动服务器
node parent.js
```

然后新开一个命令行窗口，用上 curl 工具，如下所示：

```bash
curl "http://127.0.0.1:1337"
handled by parent
curl "http://127.0.0.1:1337"
handled by child
curl "http://127.0.0.1:1337"
handled by child
curl "http://127.0.0.1:1337"
handled by parent
```

命令行中的响应结果也是很不可思议的，这里子进程和父进程都有可能处理我们客户端发起的请求。

试试将服务发送给多个子进程，如下所示：

```javascript
// parent.js
var cp = require('child_process');
var child1 = cp.fork('child.js');
var child2 = cp.fork('child.js');

// Open up the server object and send the handle
var server = require('net').createServer();
server.on('connection', function(socket) {
  socket.end('handled by parent\n');
})
server.listen(1337, function () {
  child1.send('server', server);
  child2.send('server', server);
})
```

然后在子进程中将进程 ID 打印出来，如下所示：

```javascript
// child.js
process.on('message', function (m, server) {
  if (m === 'server') {
    server.on('connection', function (socket) {
      socket.end('handled by child, pid is ' + process.pid + '\n');
    })
  }
})
```

再用 curl 测试我们的服务，如下所示

![image-20210613190226563](https://raw.githubusercontent.com/silence/blog/assets/assets/20210613190226.png)

测试的结果仍然是每次出现的结果都可能不同。对于主进程而言，我们甚至想要它更轻量一点，那么是否将服务器句柄发送给子进程之后，就可以关掉服务器的监听，让子进程来处理请求呢？

我们对主进程进行改动：

```javascript
// parent.js
var cp = require('child_process');
var child1 = cp.fork('child.js');
var child2 = cp.fork('child.js');

// Open up the server object and send the handle
var server = require('net').createServer();
server.listen(1337, function () {
  child1.send('server', server);
  child2.send('server', server);
  // 关掉
  server.close();
})
```

然后对子进程进行改动，如下所示：

```javascript
// child.js
var http = require('http');
var server = http.createServer(function (req, res) {
  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.end('handled by child, pid is ' + process.pid + '\n');
})

process.on('message', function (m, tcp) {
  if (m === 'server') {
    tcp.on('connection', function (socket) {
      server.emit('connection', socket);
    })
  }
})
```

重新启动 parent.js 后，再次测试，如下所示：

![image-20210613190941617](https://raw.githubusercontent.com/silence/blog/assets/assets/20210613190941.png)

这样一来，所有的请求都是由子进程处理了。整个过程中，服务的过程发生了一次改变，如图 9-5 所示。

![image-20210613191202562](https://raw.githubusercontent.com/silence/blog/assets/assets/20210613191202.png)

主进程发送完句柄并关闭监听之后，成为了如图 9-6 所示的结构。

![image-20210613191345295](https://raw.githubusercontent.com/silence/blog/assets/assets/20210613191345.png)

我们神奇的发现，多个子进程可以同时监听相同端口，再没有 EADDRINUSE 异常发送了。

1. 句柄发送与还原

目前子进程对象 send() 方法可以发送的句柄类型包括如下几种。

- net.Socket。TCP 套接字
- net.Server。TCP 服务器，任意建立在 TCP 服务上的应用层服务都可以享受到它带来的好处。
- net.Native。C++ 层面的 TCP 套接字或 IPC 管道
- dgram.Socket。UDP 套接字
- dgram.Native。C++ 层面的 UDP 套接字

![image-20210613195306564](https://raw.githubusercontent.com/silence/blog/assets/assets/20210613195306.png)

值得注意的是，Node 进程之间只有消息传递，不会真正地传递对象，这种错觉是抽象封装的结果。

2. 端口共同监听

在了解了句柄传递背后的原理后，我们继续探究为何通过发送句柄后，多个进程可以监听到相同的端口而不引起 EADDRINUSE 异常。其答案也很简单，我们独立启动的进程中，TCP 服务器端 socket 套接字的文件描述符并不相同，导致监听到相同的端口时会抛出异常。

Node 底层对每个端口监听都设置了 SO_REUSEADDR 选项，这个选项的涵义是不同进程可以就相同的网卡和端口进行监听，这个服务器端套接字可以被不同的进程复用，如下所示：

```
setsockopt(tcp -> io_watcher.fd, SOL_SOCKET, SO_REUSEADDR, &on, sizeof(on))
```

由于独立启动的进程互相之间并不知道文件描述符，所以监听相同端口时就会失败。但对于 send() 发送的句柄还原出来的服务而言，它们的文件描述符是相同的，所以监听相同端口不会引起异常。

多个应用监听相同的端口时，文件描述符同一时间只能被某个进程所用。换言之就是**网络请求向服务器端发送时，只有一个幸运的进程能够抢到连接，也就是说只有它能为这个请求进行服务。这些进程服务是抢占式的。**

##### 9.2.4 小结

至此，我们介绍了创建子进程、进程间通信的 IPC 通道实现、句柄在进程间的发送和还原、端口共用等细节。通过这些基础技术，用 child_process 模块在单机上搭建 Node 集群是件相对容易的事情。因此在多核 CPU 的环境下，让 Node 进程能够充分利用资源不再是难题。

#### 9.3 集群稳定之路

我们还有些细节需要考虑

- 性能问题
- 多个工作进程的存活状态管理
- 工作进程的平滑重启
- 配置或者静态数据的动态重新载入
- 其他细节

##### 9.3.1 进程事件

除了 message 事件外，Node 还有如下这些事件。

- error：当子进程无法被复制创建、无法被杀死、无法发送消息时会触发该事件。
- exit：子进程退出时触发该事件，子进程如果是正常退出，这个事件的第一个参数为退出码，否则为 null。如果进程是通过 kill() 方法被杀死的，会得到第二个参数，它表示杀死进程时的信号。
- close: 在子进程的标准输入输出流终止时触发该事件，参数与 exit 相同。
- disconnect: 在父进程或子进程中调用 disconnect() 方法时触发该事件，在调用该方法时将关闭监听 IPC 通道。

##### 9.3.2 自动重启

接着前文的多进程架构，我们在主进程上要加入一些子进程管理的机制，比如重新启动一个工作进程来继续服务。示意图如图 9-8 所示。

![image-20210613202045953](https://raw.githubusercontent.com/silence/blog/assets/assets/20210613202046.png)

实现代码如下：

```javascript
// master.js
var fork = require('child_process').fork;
var cpus = require('os').cpus();

var server = require('net').createServer();
server.listen(1337);

var workers = {};
var createWorker = function() {
  var worker = fork(__dirname + '/worker.js');
  // 退出时重新启动新的进程
  worker.on('exit', function() {
    console.log('Worker ' + worker.pid + ' exited.');
    delete workers[worker.pid];
    createWorker();
  })
  // 句柄转发
  worker.send('server', server);
  workers[worker.pid] = worker;
  console.log('Create worker. pid: ' + worker.pid);
}

for (var i = 0; i < cpus.length; i++) {
  createWorker();
}

// 进程自己退出时，让所有工作进程退出
process.on('exit', function () {
  for (val pid in workers) {
    workers[pid].kill();
  }
})
```

测试一下上面的代码，如下所示：

![image-20210613202727816](https://raw.githubusercontent.com/silence/blog/assets/assets/20210613202727.png)

通过 kill 命令杀死某个进程试试，如下所示：

```bash
$ kill 30506
```

如果是 30506 进程退出后，自动启动了一个新的工作进程 30518，总体进程数量并没有发生改变，如下所示：

```bash
Worker 30606 exited.
Create worker. pid: 30518
```

在这个场景中我们主动杀死了一个进程，在实际业务中，可能有隐藏的 bug 导致工作进程退出，那么我们需要仔细地处理这种异常，如下所示：

```javascript
// worker.js
var http = require('http');
var server = http.createServer(function (req, res) {
  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.end('handled by child, pid is ' + process.pid + '\n');
})

var worker;
process.on('message', function(m, tcp) {
  if (m === 'server') {
    worker = tcp;
    worker.on('connection', function (socket) {
      server.emit('connection', socket);
    })
  }
})

process.on('uncaughtException', function() {
  // 停止接收新的连接
  worker.close(function() {
    // 所有已有连接断开后，退出进程
    process.exit(1);
  })
})
```

上述代码的处理流程是，一旦有未捕获的异常出现，工作进程就会立即停止接收新的连接；当所有连接断开后，退出进程。主进程在侦听到工作进程的 exit 后，将会立即启动新的进程服务，以此保证整个集群中总是有进程在为用户服务的。

1. 自杀信号

上述代码存在一个问题是要等到已有的所有连接断开后进程才退出，**在极端情况下，所有工作进程都停止接收新的连接，全处在等待退出的状态。但在等到进程完全退出才重启的过程中，所有新来的请求可能纯真没有工作进程为新用户服务的场景，这会丢掉大部分请求。**

为此需要改进这个过程，不能等到工作进程退出后才重启新的工作进程。当然也不能暴力退出进程，因为这样会导致已连接的用户直接断开。**于是我们在退出的流程中增加一个自杀（suicide）信号。工作进程在得知要退出时，向主进程发送一个自杀信号，然后才停止接收新的连接，当所有连接断开后才退出。主进程在接收到自杀信号后，立即创建新的工作进程服务。**代码改动如下：

```javascript
// worker.js 
process.on('uncaughtException', function (err) {
  process.send({ act: 'suicide' });
  // 停止接收新的连接
  worker.close(function() {
    // 所有已有连接断开后，退出进程
    process.exit(1);
  })
})
```

主进程将重启工作进程的任务，从 exit 事件的处理函数中转移到 message 事件的处理函数中，如下所示：

```javascript
var createWorker = function () {
  var worker = fork(__dirname + '/worker.js');
  // 启动新的进程
  worker.on('message', function(message) {
    if (message.act === 'suicide') {
      createWorker();
    }
  })
  worker.on('exit', function () {
    console.log('Worker ' + worker.pid + ' exited.')
    delete worker[worker.pid]
  })
  worker.send('server', server);
  workers[worker.pid] = worker;
  console.log('Create worker. pid: ' + worker.pid);
}
```

与前一种方案相比，创建新工作进程在前，退出异常进程在后。在这个可怜🥺的异常进程退出之前，总是有新的工作进程来替上它的岗位。至此我们完成了进程的平滑重启，一旦有异常出现，主进程会创建新的工作进程来为用户服务，旧的进程一旦处理完已有连接就自动断开。整个过程使得我们的应用的稳定性和健壮性大大提高。示意图如图 9-9 所示。

![image-20210613205028996](https://raw.githubusercontent.com/silence/blog/assets/assets/20210613205029.png)

这里存在问题的是有可能我们的连接是长连接，不是 HTTP 服务的这种短连接，等待长连接断开可能需要较久的时间。为此为已有连接的断开设置一个超时时间是必要的，在限定时间里强制退出的设置如下所示：

```javascript
process.on('uncaughtException', function (err) {
  process.send({ act: 'suicide' });
  // 停止接收新的连接
  worker.close(function () {
    // 所有已有连接断开后，退出进程
    process.exit(1);
  })
  // 5 秒后退出进程
  setTimeout(function () {
    process.exit(1);
  }, 5000);
})
```

进程中如果出现未能捕获的异常，就意味着有那么一段代码在健壮性上是不合格的。为此退出进程前，通过日志记录下问题所在是必须要做的事情，它可以帮我们很好地定位和追踪代码异常出现的位置，如下所示：

```javascript
process.on('uncaughtException', function(err) {
  // 记录日志
  logger.error(err);
  // 发送自杀信号
  process.send({ act: 'suicide' });
  // 停止接收新的连接
  worker.close(function() {
    // 所有已有连接断开后，退出进程
    process.exit(1);
  });
  // 5 秒后退出进程
  setTimeout(function () {
    process.exit(1);
  }, 5000)
})
```

2. 限量重启

在极端情况下，如果进程启动中就发生错误，或者启动后收到连接就发生错误，会导致工作进程频繁重启。

为了消除这种无意义的重启，比如在单位时间内规定只能重启多少次，超过限制就触发 giveup 事件，告知放弃重启工作进程这个重要事件。

giveup 事件是比 uncaughtException 更严重的异常事件。uncaughtException 只代表集群中某个工作进程退出，在整体性保证下，不会出现用户得不到服务的情况，但是这个 giveup 事件则表示集群中没有任何进程服务了，十分危险。为了健壮性考虑，我们应当在 giveup 事件中添加重要日志，并让监控系统监视到这个严重错误，进而报警等。

##### 9.3.3 负载均衡

Node 默认提供的机制是采用操作系统的抢占式策略。所谓的抢占式就是在一堆工作进程中，闲着的进程对到来的请求进行争抢，谁抢到谁服务。

但是对于 Node 来说，需要分清的是它的繁忙是由 CPU、I/O 两个部分构成的，影响抢占的是 **CPU 的繁忙度**。**对不同的业务，可能存在 I/O 繁忙，而 CPU 较为空闲的情况，这可能造成某个进程能够抢到较多请求，形成负载不均衡的情况。**

Node v0.11 中有一个新的策略叫 Round-Robin，又叫轮叫调度**。轮叫调度的工作方式是由主进程接收连接，将其依次分发给工作进程。分发的策略是在 N 个工作进程中，每次选择第 i = (i + 1) mod n 个进程来发送连接。**

Round-Robin 非常简单，可以避免 CPU 和 I/O 繁忙差异导致的负载不均衡。

##### 9.3.4 状态共享

Node 不允许在多个进程之间共享数据。但在实际业务中，往往需要共享一些数据。

1. 第三方数据存储

解决数据共享最直接、简单的方式就是通过第三方来进行数据存储。

但这种方式存在的问题是如果数据发生改变，还需要一种机制通知到各个子进程，使得它们的内部状态也得到更新。

实现状态同步的机制有两种，一种是各个子进程去向第三方进行定时轮询，示意图如图 9-10 所示。

![image-20210613212530531](https://raw.githubusercontent.com/silence/blog/assets/assets/20210613212530.png)

2. 主动通知

一种改进的方式是当数据发生更新时，主动通知子进程。当然，即使是主动通知，也需要一种机制来及时获取数据的改变。这个过程仍然不能脱离轮询，但我们可以减少轮询的进程数量，我们将这种用来发送通知和查询状态是否更改的进程叫做通知进程。为了不混合业务逻辑，可以将这个进程设计为只进行轮询和通知，不处理任何业务逻辑，示意图如图 9-11 所示。

![image-20210613213141913](https://raw.githubusercontent.com/silence/blog/assets/assets/20210613213142.png)

这种推送机制如果按进程间信号传递，在跨多台服务器时会无效，是故可以考虑采用 TCP 或 UDP 的方案。进程在启动时从通知服务处除了读取第一次数据外，还将**进程信息注册到通知服务处**。**一旦通过轮询发现有数据更新后，根据注册信息，将更新后的数据发送给工作进程**。由于不涉及太多进程去向同一地方进行状态查询，状态响应处的压力不至于太过巨大，单一的通知服务轮询带来的压力并不大，所以可以将轮询时间调整得较短，一旦发现更新，就能实时地推送到各个子进程中。

#### 9.4 Cluster 模块

前文介绍了 child_process 模块中的大多数细节，以及如何通过这个模块构建强大的单机集群。

对于本章开头提到的创建 Node 进程集群，cluster 实现起来也是很轻松的事情，如下所示：

```javascript
// cluster.js
var cluster = require('cluster');

cluster.setupMaster({
  exec: 'worker.js'
});

var cpus = require('os').cpus();
for (var i = 0; i < cpus.length; i++) {
  cluster.fork();
}
```

执行 node cluster.js 将会得到与前文创建子进程集群的效果相同。就官方文档而言，它更喜欢如下的形式作为示例：

```javascript
var cluster = require('cluster');
var http = require('http');
var numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
  // Fork workers
  for (var i = 0; i < numCPUs; i++) {
    cluster.fork();
  }
  
  cluster.on('exit', function(worker, code, signal) {
    console.log('worker ' + worker.process.pid + ' died');
  })
} else {
  // Workers can share any TCP connection
  // In this case its a HTTP server
  http.createServer(function (req, res) {
    res.writeHead(200);
    res.end('hello world\n');
  }).listen(8000)
}
```

在进程中判断是主进程还是工作进程，主要取决于环境变量中是否有 NODE_UNIQUE_ID，如下所示：

```javascript
cluster.isWorker = ('NODE_UNIQUE_ID' in process.env);
cluster.isMaster = (cluster.isWorker === false);
```

但是官方示例中忽而判断 cluster.isMaster、忽而判断 cluster.isWorker，对于代码的可读性十分差。我建议用 cluster.setupMaster() 这个 API，将主进程和工作进程从代码上完全剥离，如同 send() 方法看起来直接将服务器从主进程发送子进程那样神奇，剥离代码之后，甚至都感觉不到主进程中有任何服务器相关的代码。

通过 cluster.setupMaster() 创建子进程而不是使用 cluster.fork()，程序结构不再凌乱，逻辑分明，代码的可读性和可维护性较好。

##### 9.4.1 Cluster 工作原理

事实上 cluster 模块就是 child_process 模块和 net 模块的组合应用。cluster 启动时，**如同我们在 9.2.3 节里的代码一样，它会在内部启动 TCP 服务器，在 cluster.fork() 子进程时，将这个 TCP 服务器端 socket 的文件描述符发送给工作进程**。如果进程是通过 cluster.fork() 复制出来的，那么它的环境变量里就存在 NODE_UNIQUE_ID，如果工作进程中存在 listen() 侦听网络端口的调用，它将拿到该文件描述符，通过 SO_REUSEADDR 端口重用，从而实现多个子进程共享端口。对于普通方式启动的进程，则不存在文件描述符传递共享等事情。

对于 cluster 模块，一个主进程只能管理一组工作进程。如图 9-12 所示:

![image-20210614124003617](https://raw.githubusercontent.com/silence/blog/assets/assets/20210614124003.png)

#### 9.5 总结

尽管通过 child_process 模块可以大幅提升 Node 的稳定性，但是一旦主进程出现问题，所有子进程将会失去管理。在 Node 的进程管理之外，还需要用**监听进程数量或监听日志的方式确保整个系统的稳定性，即使主进程出错退出，也能及时得到监控警报，使得开发者可以及时处理故障。**

### 第 10 章 测试

#### 10.1 单元测试

##### 10.1.1 单元测试的意义

对于开发者而言，单元测试就是最基本的一种方式。如果开发者不自己测试代码，那必然要面对如下问题。

1. 测试工程师是否可依赖？
2. 第三方代码是否可信赖？
3. 在产品的迭代过程中，如何继续保证质量？

简单而言，编写可测试代码有以下几个原则可以遵循。

- **单一职责**。一段代码只承担一个职责，承担多个职责的需要进行解耦分离。
- **接口抽象**。通过对程序代码进行接口抽象后，我们可以针对接口进行测试，而具体代码实现的变化不影响为接口编写的单元测试。
- **层次分离**。层次分离实际上是单一职责的一种实现。在 MVC 接口的应用中，就是典型的层次分离模型，如果不分离各个层次，无法想象这个代码该如何切入测试。通过分层之后，可以逐层测试，逐层保证。

对于开发者而言，**不仅要编写单元测试，还应当编写可测试代码**。

#### 10.2 性能测试

单元测试主要用于检测代码的行为是否符合预期。在完成代码的行为检测后，还需要对已有代码的性能作出评估。

性能测试的范畴比较广泛，包括负载测试、压力测试和基准测试等。这里将只会简单介绍下基准测试。

除了基准测试，这里还将介绍如何对 Web 应用进行网络层面的性能测试和业务指标的换算。

##### 10.2.1 基准测试

基准测试要统计的就是在多少时间内支线了多少次某个方法。

##### 10.2.2 压力测试

除了可以对基本的方法进行基准测试外，通常还会对网络接口进行压力测试以判断网络接口的性能。对网络接口做压力测试需要考查的几个指标有**吞吐率**、**响应时间**和**并发数**，这些指标反映了服务器的并发处理能力。

##### 10.2.3 基准测试驱动开发

其中主要分为如下几步其流程图如图 10-9 所示。

1. 写基准测试。
2. 写/改代码。
3. 收集数据。
4. 找出问题。
5. 回到第 2 步。

![image-20210614133432683](https://raw.githubusercontent.com/silence/blog/assets/assets/20210614133432.png)

##### 10.2.4 测试数据与业务数据的转换

通常，在进行实际的功能开发之前，我们需要评估业务量，以便功能开发完成后能够胜任实际的在线业务量。如果用户量只有几个，每天的 PV 只有几十个，那么网站开发几乎不需要什么优化就能胜任。**如果 PV 上 10 万甚至百万、千万，就需要运用性能测试来验证是否能够满足实际业务需求，如果不满足，就要运用各种优化手段提升服务能力**。

假设某个页面每天的访问量为 100 万。根据实际业务情况，主要访问量大致集中在 10 个小时以内，那么换算公式就是：

QPS = PV / 10h

100w 的业务访问量换算为 QPS，约等于 27.7，即服务器需要每秒处理 27.7 个请求才能胜任业务量。

### 第 11 章 产品化

在实际的产品中，需要很多非编码相关的工作以保障项目的进展和产品的正常运行等，这些细节包括**工程化、架构、容灾备份、部署和运维等**。只有这些任务在持续性进行，才表面项目是活着的。

#### 11.1 项目工程化

##### 11.1.1 目录结构

类似现在框架的脚手架把目录结果给约定好。

##### 11.2.1 部署环境

我们将普通测试称为 stage 环境（类似于 TCE 的开发测试环境），预发布环境称为 pre-release 环境（类似于 TCE 的集成测试环境），实际的生产环境称为 product 环境，整个部署流程如图 11-3 所示。

![image-20210614134941140](https://raw.githubusercontent.com/silence/blog/assets/assets/20210614134941.png)

#### 11.3 性能

Node 产品的性能与许多因素相关，这里我们将范畴缩减到 Web 应用中来，只评估一些常见的提升性能的方法。对于 Web 应用而言，最直接有效的莫过于**动静分离、多进程架构、分布式**，其中涉及的几个拆分原则如下所示。

- 做专一的事。
- 让擅长的工具做擅长的事。
- 将模型简化。
- 将风险分离。

除此之外，缓存也能带来很大的性能提升。

##### 11.3.1 动静分离

在普通的 Web 用于中，Node 尽管也能通过中间件实现静态文件服务，但是 Node 处理静态文件的能力并不算突出。将图片、脚本、样式表和多媒体等静态文件都引导到专业的静态文件服务器上，让 Node 只处理动态请求即可。这个过程可以用 Nginx 或者专业的 CDN 来处理。图 11-4 为动静分离的示意图。

![image-20210614135725591](https://raw.githubusercontent.com/silence/blog/assets/assets/20210614135725.png)

##### 11.3.2 启用缓存

如今，Redis 或 Memcached 几乎是 Web 应用的标准配置。如果你的产品需要应对巨大的流量，启用缓存并应用好它，是系统性能瓶颈的关键。

##### 11.3.3 多进程架构

##### 11.3.4 读写分离

这主要针对数据库而言。就任意数据库而言，读取的速度远远高于写入的速度。而某些数据库在写入时为了保证数据的一致性，会进行锁表操作，这同时会影响到读取的速度。某些系统为了提升性能，通常会进行数据库的读写分离，**将数据库进行主从设计，这样读数据操作不再受到写入的影响，降低了性能的影响。**

#### 11.4 日志

与其遇见 bug 修复它，不如建立健全的排查和跟踪机制，而日志就是实现这种机制的关键。在健全的系统中，完善的日志记录最能够还原问题现场。通过记录日志来定位问题是一种成本较小的方式。**这种非结构化、轻量的记录方式容易实现，也容易扩展**。

##### 11.4.1 访问日志

访问日志一般用来记录每个客户端对应用的访问。在 Web 应用中，**主要记录 HTTP 请求中的关键数据**。一般的 Web 服务器都实现了记录访问日志的功能，只需要简单的配置即可启用。

在实际的应用场景中，可以置入一些用户信息，用以跟踪一些数据，比如某个登陆用户太过密集地访问某个页面等。

##### 11.4.2 异常日志

**异常日志通常用来记录那些意外产生的异常错误**。通过日志的记录，开发者可以根据异常信息去定位 bug 出现的具体位置，以快速修复问题。

**异常日志通常有完善的分级**，Node 中提供的 console 对象就简单地实现了这几种划分，具体如下所示：

- console.log: 普通日志。
- console.info: 普通信息。
- console.warn: 警告信息。
- console.error: 错误信息。

console 模块在具体实现时，log 与 info 方法都将信息输出给标准输出 process.stdout，**warn 与 error 方法则将信息输出到标准错误 process.stderr**。

**info 和 error 分别是 log 和 warn 的别名。**

console 对象上具有一个 Console 属性，它是 console 对象的构造函数。借助这个构造函数，我们可以实现自己的日志对象，相关代码如下：

```javascript
var info = fs.createWriteStream(logdir + '/info.log', { flag: 'a', mode: 'o666' });

var error = fs.createWriteStream(logdir + '/error.log', { flag: 'a', mode: 'o666' });

var logger = new console.Console(info, error);
```

分别调用它的 API，日志内容就能各自写入到对应的文件中，相关代码如下：

```javascript
logger.log('Hello world!');
logger.error('segment fault');
```

有了记录信息的日志 API 后，开发者需要小心的是要小心捕获每一个异常。在第 4 章中，我们提到异步调用中回调函数里的异常无法被外部捕获的问题，也提到了异步 API 编写的规范**，每个开发者应当将 API 内部发生的异常作为第一个实参传递给回调函数。对于回调函数中产生的异常，则可以不用过问，交给全局 uncaughtException 事件去捕获即可。**

在逐层次的异步 API 调用中，异常是该传递给调用方还是该立即通过日志记录，这是一个需要注意的问题。**就通常的 API 编写而言，尽量不要隐藏错误，不要通过 try/catch 块将异常捕获，然后隐藏起来不向外部调用者暴露。**这对于底层 API 的设计而言，尤为重要。事实上，日志通常是服务于业务的。我的建议是**异常尽量由最上层的调用者去捕获记录，底层调用或中间层调用中出现的异常只要正常传递给上层的调用方即可。**

我们通常需要准备一个 format() 方法来封装和格式化异常信息。如下为异常日志示例：

![1623652282494](https://raw.githubusercontent.com/silence/blog/assets/assets/20210614143751.png)

##### 11.4.3 日志与数据库

有的开发者可能对日志不太了解，会选择将一些日志写入到数据库中。数据库比日志文件好的地方在于它是结构化的数据，可以直接编写 SQL 语句进行分析，日志文件则需要再加工之后才能分析。

**但是日志文件与数据库写入在性能上处于两个级别，数据库写入过程中要经历一系列处理，比如锁表、日志等操作。写日志文件则是直接将数据写到磁盘上。为此，如果有大量的访问，可能会存在写入操作大量排队的状况，数据库的消费速度验证低于生产速度，进而导致内存泄漏等。**相比之下，**写日志是轻量的方法，将日志分析和日志记录这两个步骤分离开是较好的选择**。日志记录可以在线写，日志分析则可以借助一些同步到数据库中，通过离线分析的方式反馈出来。

##### 11.4.4 分割日志

线上业务可能访问量巨大，产生的日志也可能是大量的，上述示例只是简单地将普通日志和异常日志分开放在两个文件中，日志过多时也不便直接查看。为此，将产生的日志按日期分割是一个不错的主意。

#### 11.5 监控报警

部署好流程，记录好日志之后，应用就似乎可以自行运转了。但是应用也应当有一个监控系统。如果应用出了差错，也需要通过监控及时发现，然后恢复它正常运行。

应用的监控主要有两类，一种是**业务逻辑型的监控**，一种是**硬件型的监控**。监控主要通过定时采样来进行记录。除此之外，还要对监控的信息设置上限，**一旦出现大的波动，就需要发出警报提醒开发者**。为了较好地供开发者使用，监控到的信息一般还要通过数据可视化的方式反映出来，以便更直观地查看。

##### 11.5.1 监控

1. 日志监控

业务逻辑型的监控主要体现在日志上。

通过监控异常日志文件的变动，**将新增的异常按异常类型和数量反映出来**。某些异常与具体的某个子系统相关，监控出现的某个异常多半能反映出子系统的状态。

除了异常日志的监控外，对于访问日志的监控也能体现出实际业务的 QPS 值。观察 QPS 的表现能够**检查业务在时间上的分布**。

此外，从访问日志从也能实现 PV 和 UV 的监控。同 QPS 值一样，通过对 PV/UV 的监控，可以很好地知道应用的**使用者们的习惯、预知访问高峰**等。

2. 响应时间

响应时间也是一个需要监控的点。**一旦系统的某个子系统出现异常或者性能瓶颈，将会导致系统的响应时间变长。响应时间可以在 Nginx 一类的反向代理上监控，也可以通过应用自行产生的访问日志来监控。**监控的系统的响应时间应该是波动较小的、持续均衡的。

3. 进程监控

**监控进程一般是检查操作系统中运行的应用进程数**，比如对于采用多进程架构的 Web 应用，就需要检查工作进程的数量，如果低于预估值，就应当发出警报声。

4. 磁盘监控

磁盘监控主要是监控磁盘的用量。

5. 内存监控

监控服务器的内存使用状况，可以检查应用中是否存在内存泄漏的状况。

6. CPU 占用监控

CPU 的使用分为用户态、内核态、IOWait 等。如果用户态 CPU 使用率较高，说明**服务器上的应用需要大量的 CPU 开销**；如果内核态 CPU 使用率较高，说明**服务器花费大量时间进行进程调度或者系统调用**；IOWait 使用率则反应的是 **CPU 等待磁盘 I/O 操作**。

CPU 的使用率中，用户态小于 70%、内核态小于 35% 且整体小于 70% 时，处于健康状态。

7. CPU load 监控

CPU load 又称 CPU 平均负载，它用来描述**操作系统当前的繁忙程度**，可以简单地理解为 CPU 在单位时间内正在使用和等待使用 CPU 的平均任务数。

8. I/O 负载

I/O 负载指的主要是磁盘 I/O。反应的是磁盘上的读写情况。

9. 网络监控

网络流量监控的两个主要指标是流入流量和流出流量。

10. 应用状态监控

外部监控将会持续性地调用应用的反馈接口来检查它的监控状态。

健壮一些的状态响应则是将应用的依赖项的状态打印出来，如数据库连接是否正常、缓存是否正常等。

11. DNS 监控

对于产品的稳定性，域名 DNS 状态也需要胶乳监控。目前国内有一些免费的 DNS 监控服务，如 DNSPod 等，可以通过这些监控服务，监控自己的在线应用。

##### 11.5.2 报警的实现

- 邮件报警。如果报警系统由 Node 编写，可以调用 nodemailer 模块来实现邮件的发送。
- 短信或电话报警。

##### 11.5.3 监控系统的稳定性

#### 11.6 稳定性

为了更好的稳定性，典型的水平扩展方式就是多进程、多机器、多机房，这样的分布式设计在现在的互联网公司并不少见。

- 多机器

  一旦出现分布式，就需要考虑**负载均衡**、**状态共享**和**数据一致性**的问题。

  图 11-5 为负载均衡的示意图。

  ![image-20210614155215015](https://raw.githubusercontent.com/silence/blog/assets/assets/20210614155215.png)

  对于状态共享和数据一致性，它们与多进程的问题是一致的，具体可参加第 9 章。

- 多机房

  在容灾方面，机房与机房之间可以互为备份。由于机房与机房之间的网络复杂度再度提升，负载均衡方面需要进一步去统筹规划。此处不再展开。

- 容灾备份

  在多机房和多机器的部署结构下，十分容易通过备份的方式进行容灾，任何**一台机器或者一个机房停止了服务，都能有其余的服务器来接替新的任务**。在这个机制下，我们至少需要 4 台服务器来构建这个稳定的服务集群，如图 11-6 所示。

  ![image-20210614155646023](https://raw.githubusercontent.com/silence/blog/assets/assets/20210614155646.png)

#### 11.7 异构同存

图 11-7 为编程语言与服务之间通过网络协议进行调用的示意图。

![image-20210614155916496](https://raw.githubusercontent.com/silence/blog/assets/assets/20210614155916.png)

总之，在应用 Node 的过程中，不存在为了用它而推翻已有设计的请求。Node 能通过协议与已有的系统很好地异构同存。







