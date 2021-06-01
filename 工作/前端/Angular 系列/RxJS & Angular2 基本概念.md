[TOC]

本文为书籍 《Angular 2 从 0 到 1》和 《揭秘 Angular （第二版）》的重点摘抄

### 序：前端风云

本章主要介绍各种前端技术背景、作用及 JavaScript 的发展历程。最后，介绍现今前端技术的理念和趋势。

#### 故事的起点

1995 年，任职于网景公司的 Brendan Eich 为 Netscape Navigator 2.0 创造出了 JavaScript 。

1997 年 6 月，ECMA 的第 39 号技术专家委员会（Technical Committee 39，简称 TC39）制定了第 1 版的 ECMA-262 标准，它定义了 ECMAScript （简称 ES）这门全新的脚本语言。接着在 1999 年，带着多种新特性的 ECMAScript 3 与 IE 5 正式发布，同年年末 HTML 4.01 也闪耀登场。在种种的新机遇下，前端领域逐渐发展起来。

#### AJAX 王者归来

2004 年 4 月 1 日，Google 公司发布了 Gmail。在 Gmail 中，首次使用了与服务器高度互动的 JavaScript 脚本，实现了更好的局部刷新效果，让交互体验更接近常规软件。后来在 2005 年的一篇文章中将这些技术命名为 AJAX （Asynchronous JavaScript And XML），它也成了现在 Web 开发的标准做法。

AJAX 的出现不仅仅是提升了加载速度那么简单，而是让前端可以承担更多的工作，为前后端开发的解耦创造了可能。

#### 工具库的流行

2006 年 jQuery 发布。jQuery 以其巧妙的接口封装、简洁的链式写法和高效的选择器实现，再加上丰富的插件体系，不需要关注不同浏览器的接口差异问题，大大提升了前端开发的生产力。在一段时间里很多前端工程师的招聘要求中都提及需要熟悉甚至精通 jQuery，可见其使用的广泛性与重要性。

伴随着各种 DOM 操作与模板引擎的出现，再加上相应的 UI 组件库的普及，前端社区内也出现了各类前端架构化的尝试与小范围的实践。不少公司的项目也由原先后端主导的模式转向富前端化，将更多的逻辑交由前端来实现，而后端仅提供更为底层的数据处理与部署运维。

#### 百家争鸣

AngularJS 诞生的 2009 年，在前端史上无疑是意义非凡的一年。

在这一年里，ECMAScript 5 发布，这宣告了 ECMAScript 4 的最终“流产”。ECMAScript 4 的“流产”事件却也为后续的 ECMAScript 6 的定案做了铺垫。同年，技术社区内发布了一项名为 CommonJS 的规范，解决了 JavaScript 模块化的问题，而后衍生出了 AMD （代表库 RequireJS）、CMD（代表库 Sea.js）等模块化规范，使得前端模块化越来越流行。也在同一年，Ryan Dahl 基于 CommonJS 规范和 Google V8 引擎创造出了 Node.js。它是服务端 JavaScript 的运行环境，可以让前端工程师更简单地进行后端开发。Node.js 的出现，也让大量前端自动化工具如 Grunt、Gulp、Webpack 等陆续被创造出来。2012 年发布的 TypeScript，身为 JavaScript 的超集，更是把强类型特性带到了前端社区，通过类型检测更容易发现问题。TypeScript 紧跟着 ECMAScript 标准的实现，吸引了大量粉丝，火爆度逐日递增。

#### 走进前端新时代

2014 年 9 月，HTML 5 发布，标志着大量功能强大的接口特性被确定下来。使用 React Native，前端工程师可以进行移动端开发，使用 Electron ，前端工程师可以进行桌面开发，使用 Node.js技术，前端工程师可以进行 同构 （Universal）服务器端应用开发。由此前端工程师的工作能力范畴被放大，市场价值增加了很多。

伴随着前端社区的快速迭代发展，在 2015 年 6 月 17 日，ECMAScript 6 正式发布。此新标准从开始制定到最终发布历时 15 年，带来了大量的新特性与语法糖，**同年 TC39 决定以后每年都将当前已经标准化的特性发布出来，并以年份进行版本命名，所以 ECMAScript 6 也被称为 ECMAScript 2015。**

#### Typescript 

##### 基本类型

1. boolean
2. number
3. string
4. array
5. **tuple**
6. **enum**
7. any
8. null 和 undefined
9. void类型
10. never 类型 （返回值为 never 的函数可以是永远无法执行到终点的情况）

### 第一章：认识 Augular

#### 第一个组件

Angular 提倡的文件命名方式是这样的：`组件名称.component.ts`，组件的 HTML 模板命名为：`组件名称.component.html`，组件的样式文件命名为：`组件名称.component.css`，大家在编码中尽量遵循 Google 的官方建议。

我们新生成的 Login 组件源码如下

```typescript
import { Component, OnInit } from '@angular/core';

// @Component 是 Angular 提供的装饰器函数，用来描述 Component 的元数据
// 其中 selector 是指这个组件的在 HTML 模板中的标签是什么
// template 是嵌入 （inline）的 HTML 模板，如果使用单独文件可用 templateUrl
// styles 是嵌入（inline）的 CSS 样式，如果使用单独文件可用 styleUrls
@Component({
  selector: 'app-login',
  tempate: `
		<p>
			login Works!
		</p>
	`
})
export class LoginComponent implements OnInit {
  constructor() { }
  
  ngOnInit() { }
}
```

那么这个组件建成后，我们怎么使用呢？注意上面的代码中 `@Component` 装饰修饰配置中的 `selector: 'app-login'`，这意味着我们可以在其他组件的 template 中使用 `<app-login></app-login>` 来引用我们的这个组件。

现在我们打开 `hello-augular/src/app/app.component.html` 加入我们的组件引用

```html
<h1>
  {{title}}
</h1>
<app-login></app-login>
```

#### 一些基础概念

##### 什么是模块？

简单来说**模块就是提供相对独立的功能的功能块**，每块聚焦于一个特定业务领域。Angular 内建的很多库是以模块形式提供的，比如 FormsModule 封装了表单处理，HttpModule 封装了 Http 的处理等等。

每个 Angular 应用至少有一个根模块类 --- 根模块，**我们通过引导根模块来启动应用**。按照约定，根模块的类名称叫做 AppModule，被放在 `app.module.ts` 文件中。我们这个例子的根模块位于 `hello-angular/src/app/app.module.ts`

```typescript
import { BrowserModule } from '@augular/platform-browser';
import { NgModule } from '@angular/core';
import { FormsModule } from '@angular/forms';
import { HttpModule } from '@angular/http';

import { AppComponent } from './app.component';
import { LoginComponent } from './login/login.component';

@NgModule({
  declaratoins: [
    AppComponent,
    LoginComponent
  ],
  imports: [
    BrowserModule,
    FormsModule,
    HttpModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

@NgModule 装饰器用来为模块定义元数据。declarations 列出了应用中的顶层组件，包括引导性组件 AppComponent 和我们刚刚创建的 LoginComponent。**在 module 里面声明的组件在 module 范围内都可以直接使用，也就是说在同一 module 里面的任何 Component 都可以在其模板文件中直接使用声明的组件**，就像我们在 AppComponent 的模板末尾加上 `<app-login></app-login>` 一样。

imports 引入了 3 个辅助模块：

- BrowserModule 提供了运行在浏览器中的应用所需要的关键服务（Service）和指令（Directive），这个模块所有需要在浏览器中跑的应用都必须引用；
- FormsModule 提供了表单处理和双向绑定等服务和指令
- HttpModule 提供 Http 请求和响应的服务

providers 列出会在此模块中“注入”的服务（Service），关于依赖注入会在后面的章节中详细解释。

bootstrap 指明哪个组件为引导性组件（本案例中的 AppComponent）。当 Angular 引导应用时，它会在 DOM 中渲染这个引导性组件，并把结果放进 index.html 的该组件的元素标签中（本案例中的 app-root）。

##### 引导过程

Angular 2 通过在 main.ts 中引导 AppModule 来启动应用。针对不同的平台，Angular 提供了很多引导选项。下面的代码是通过**即时（JiT - Just In Time）编译器动态引导**，一般在进行开发调试时，默认采用这种方式。

```typescript
// main.ts
import './polyfills.ts';

// 连同 Angular 编译器一起发布到浏览器端
import { platformBrowserDynamic } from '@angular/platform-browser-dynamic';
import { enableProdMode } from '@angular/core';
import { enviroment } from './environment/environment';
import { AppModule } from './app/';

if (environment.production) {
  enableProdMode();
}
// Angular 编译器在浏览器中编译并引导该应用
platformBrowserDynamic().bootstrapModule(AppModule);
```

另一种方式是**使用预编译器（AoT - Ahead-Of-Time）进行静态引导，静态方案可以生成更小、启动更快的应用**，建议优先使用它，特别是在移动设备或高延迟网络下。使用 static 选项，Angular 编译器作为构建流程的一部分提前运行，生成一组类工厂。它们的核心就是 AppModuleNgFactory。引导预编译的 AppModuleNgFactory 的语法和动态引导 AppModule 类的方式很相似。

```typescript
// 不把编译器发布到浏览器
import { platformBrowser } from '@angular/platform-browser';

// 静态编译器会生成一个 AppModule 的工厂 AppModuleNgFactory;
import { AppModuleNgFactory } from './app.module.ngfactory';

// 引导 AppModuleNgFactory
platfromBrowser().bootstrapModuleFactory(AppModuleNgFactory);
```

#### 小结

Angular 的各个组成部分以组件作为桥梁关联起来，对组件的深度剖析将作为本书深入讲解的开篇；之后是与组件密切相关的模板章节，它囊括了 Angular 大部分内置的模板语法，仔细阅读完后可对 Angular 的视图特性有一个系统的认识；接着，指令、服务、依赖注入及路由这四个相对比较独立的概念会分别开辟新章节讲述；最后会加入测试这个特殊章节，测试是保证应用质量的关键环节，该章节会详细介绍 Angular 的各个部分是如何实施自动化测试的。

### 第二章：组件

#### 组件化标准

W3C 为了统一组件化的标准方式，提出了 Web Component 标准。通过标准化的非侵入方式封装组件，每个组件都包含自己的 HTML、CSS、JavaScript 代码，并且不会对页面上的其它组件产生影响。

Web Component 标准包括如下四个重要的概念。

- 自定义元素：这个特性允许开发者创建自定义的 HTML 标签和元素，每个元素都有属于自己的脚本和样式。
- 模板：模板允许开发者使用 \<ng-template> 标签预先定义一些内容，但并不随页面加载而渲染，而是可以在运行时使用 JavaScript 来初始化它。
- Shadow DOM：通过 Shadow DOM 可以在文档流中创建一些完全独立于其它元素的 DOM 子树，这个特性可以让开发者开发一个独立组件，并且不会干扰其他 DOM 元素。
- HTML 导入：一种在 HTML 文档中引入其他 HTML 文档的方法，用于导入 Web Component 的机制。

### 第十章 依赖注入

#### 依赖注入介绍

控制反转的概念最早由 Martin Fowler 于 2004 年提出，是针对面向对象设计不断复杂化而提出的一种设计原则，是一种利用面向对象编程法则来降低应用程序耦合的设计模式。IoC 强调的是对代码引用的控制权由调用方转移到外部容器，在运行时通过某种方式（如：反射）注入进来，实现了控制的反转，这大大降低了服务类之间的耦合度。依赖注入是最常用的一种实现 IoC 的方式，也是本章将要讲解的内容。另一种是依赖查找，读者可自行在网上了解，本章不再赘述。

在依赖注入模式中，**应用组件无需关注所依赖对象的创建或初始化过程，可以认为框架已经初始化好了，开发者只管调用即可**。依赖注入有利于应用程序中各模块之间的解耦，使得代码更容易维护。这种优势可能一开始体现不出来，但随着项目复杂度的增加，当各模块、组件、第三方服务等相互调用更频繁时，依赖注入的优点就体现的淋漓尽致。开发者可以专注于所依赖对象的消费，无须关注这些依赖的生成过程，这无疑将大大提升开发效率。

接下来通过一个机器人的例子来加深了解依赖注入带来的好处。示例代码如下：

```typescript
// 不使用依赖注入的示例

export class Robot {
  public head: Head;
	public arms: Arms;
	constructor() {
    this.head = new Head();
    this.arms = new Arms();
  }
  // 移动机器人
	move() {}
}
```

一个 Robot 类会包含 Head、Arms。那么上面的代码存在什么问题呢？Robot 在它自身的构造函数中创建并引用了 Head 和 Arms 的实例，仔细分析就会发现，这种做法使得 Robot 类的扩展性差且难以测试，具体说明如下。

- 拓展性差

  Robot 类通过 Head 和 Arms 创建了自己需要的组件，即头部和胳膊。试想一下，如果 Head 类的构造函数需要一个参数呢？此时没其他办法，只能通过 this.head = new Head(theNewParameter) 的方式修改 Robot 类；或者如果需要给 Robot 类 换一个带有无线通信功能的头（HeadWithWireless），只能将 Robot 引用的 Head 修改为 HeadWithWireless。

- 难以测试

  当需要测试 Robot 类时，需要考虑 Robot 类隐藏的其它依赖。比如 Head 组件本身是否依赖其他组件，且它依赖的组件是否也依赖另一个组件；另外，Head 组件的实例是否发送了异步请求到服务器等。正是因为不能控制 Robot 的隐藏依赖，所以 Robot 很难被测试。

怎样才能使 Robot 类变得易于扩展且易于测试呢？这里用依赖注入的设计模式改造一下 Robot 的构造函数。示例代码如下：

```typescript
// 使用依赖注入的示例
export class Robot {
  public head: Head;
	public arms: Arms;
  constructor(public head: Head, public arms: Arms) {
    this.head = head;
    this.arms = arms;
  }
  
  // 移动机器人
  move() {}
}
```

这里把依赖对象作为参数传给构造函数，在 Robot 类中不再创建 Head 和 Arms。当创建 Robot 实例时，只需把创建好的 Head 和 Arms 实例传给它的构造函数就可以了。示例代码如下：

```javascript
var robot = new Robot (new Head(), new Arms());
```

到这一步，就实现了 Robot 类与 Head 类及 Arms 类的解耦，开发者可以将任何 Head 和 Arms 实例注入到 Robot 类的构造函数中，它们之间的注入关系如图所示。

![image-20210520202530903](https://raw.githubusercontent.com/silence/blog/assets/assets/20210520202531.png)

依赖注入的一个典型应用场景就是测试，使用依赖注入方式编写的代码，测试人员在做场景覆盖测试时，基本上不需要修改被测试程序，只需要将依赖对象注入到被测试程序即可。这里以测试 Robot 组件为例，将 Head 和 Arms 的 mock 对象传入 Robot 类的构造函数中。示例代码如下：

```typescript
class MockHead extends Head {
  head = '头部';
}

class MockArms extends Arms {
  arms = '胳膊'
}

var robot = new Robot(new MockHead(), new MockArms())
```

依赖注入通过注入服务的方式替代了在组件里初始化所依赖的对象，从而避免了组件之间的紧耦合。但是这还不够，在使用 Robot 类时还需要手动创建 Head、Arms 及 Robot 的实例，为了减少重复操作，这里可以通过创建一个 Robot 的工厂类来解决。示例代码如下：

```typescript
// Robot 工厂类，这并不是依赖注入的做法，仅作为引出后面内容的示例
import { Head, Arms, Robot } from './robot';

export class RobotFactory {
  createRobot() {
    let robot = new Robot(this.createHead(), this.createArms());
    return robot;
  }
  
  createHead() {
    return new Head();
  }
  
  createArms() {
    return new Arms();
  }
}
```

上述代码只有三个方法，比较好维护，但是随着代码量的增加，维护这些代码就会变得很棘手。幸运的是，Angular 的依赖注入框架（Dependency Injection Framework）替开发者解决了这个问题。有了它，开发者就不用去关心需要定义哪些依赖，以及把这些依赖注入给谁了。因为依赖注入提供了注入器（Injector），它会帮助开发者创建所需要的类实例。

```javascript
var injector = new Injector();
var robot = injector.get(Robot);
```

有了注入器，Robot 就不需要知道如何创建它所依赖的 Head 和 Arms 了，用户也不需要知道如何生产一个 Robot，同时也不需要维护一个巨大的工厂类了。

通过 Robot 例子的学习，读者对依赖注入应该有了一个基本的认识 --- **依赖注入有利于各种组件之间的解耦，代码易于维护，提升了开发效率**。

### 番外：Rx -- 隐藏在 Angular 中的利剑

Rx (Reactive Extension -- 响应式扩展 http://reactivex.io ) 最近在各个领域都非常火。其实 Rx 这个货是微软在好多年前针对 C# 写的一个开源类库，但好多年都不温不火，一直到 Netflix 针对 Java 平台做出了 RxJava 版本后才在开源社区热度飞书蹿升。

那么什么叫响应式编程呢？这里引用一下 Wikipedia 的解释：

> In computing, reactive programming is a programming paradigm oriented around data flows and the propagation of change. This means that it should be possible to express static or dynamic data flows with ease in the programming languages used, and that the underlying execution model will automatically propagate changes through the data flow.

> 翻译：在计算领域，响应式编程一种面向数据流和变化传播的编程范式。这意味着可以在编程语言中很方便地表达静态或动态的数据流，而相关的计算模型会自动将变化的值通过数据流进行传播。

这都说的什么啊？没关系，概念永远是抽象的，我们来举几个例子。比如说在传统的编程中 a = b + c，表示将表达式的结果赋给 a，而之后改变 b 或 c 的值不会影响 a。但在响应式编程中，a 的值会随着 b 或 c 的更新而更新。

![image-20210514165403884](https://raw.githubusercontent.com/silence/blog/assets/assets/20210514165403.png)

那么用响应式编程方法写出来就是这个样子，可以看到随着 b 和 c 变化 a 也会随之变化。

![image-20210514165501824](https://raw.githubusercontent.com/silence/blog/assets/assets/20210514165501.png)

看出来一些不一样的思维方式了吗？响应式编程**需要描述数据流**，而不是单个点的数据变量，我们需要把数据的每个变化汇聚成一个数据流。如果说传统编程方式是基于离散的点，那么**响应式编程就是线**。

上面的代码虽然很短，但体现出 Rx 的一些特点

1. Lamda 表达式，对，就是那个看上去像箭头的东西 => 。你可以把它想象成一个数据流的指向，我们从箭头左方取得数据流，在右方做一系列处理后或者输出成另一个数据流或者做一些其他对于数据的操作。
2. 操作符：这个例子中的 from，zip 都是操作符。Rx 中有太多的操作符，从大类上讲分为：创建类操作符、变换类操作符、过滤类操作符、合并类操作符、错误处理类操作符、工具类操作符、条件型操作符、数学和聚集类操作符、连接型操作符等等。

#### Rx 再体验

还是从例子开始，我们逐渐的来熟悉 Rx。

在 RxJS 领域一般在 Observable 类型的变量后面加上 $ 标识这是一个“流变量”（由英文 Stream 得来，Observable 就是一个 Stream，所以用 \$ 标识），不是必须的，但是属于约定俗成。

```javascript
let todo = document.getElementById('todo');
let input$ = Rx.Observable.fromEvent(todo, 'keyup');
input$.subscribe(input => console.log(input.target.value));
```

在 Outupt 窗口中应该可以看到一个文本输入框，在这个输入框中输入任意你要试验的字符，观察 Console

![image-20210514170558350](https://raw.githubusercontent.com/silence/blog/assets/assets/20210514170558.png)

这几行代码很简单：首先我们得到 HTML 中 id 为 todo 的输入框对象，然后定义一个观察者对象将 todo 这个输入框的 keyup 事件转换成一个数据流，最后订阅这个数据流并在 console 中输出我们接收到的 input 事件的值。我们从这个例子中可以观察到几个现象：

1. 数据流：你每次在输入框输入时都会有新的数据被推送过来。本例中，你会发现连续输入“1，2，3，4”，在 console 的输出是 “1，12，123，1234”，也就是说每次 keyup 事件我们都得到了完整的输入框中的值。而且这个数据流是无限的，只要我们不停止订阅，它就会一直在那里待命。
2. 我们观察的是 todo 上发生的 keyup 这个事件，那如果我一直按着某个键不放会怎么样呢？你的猜测是对的，一直按着的时候，数据流没有更新，直到你抬起按键为止（你看到截图里面有 2 条一模一样的含有多个 5 的数据是因为我用的 Surface Pro 截图时的快捷键也被捕获了，但由于是控制键所以文字内容没有改变）

![image-20210514171827663](https://raw.githubusercontent.com/silence/blog/assets/assets/20210514171827.png)

如果观察的足够仔细的话，你会发现 console 中输出的值其实是 input.target.value，我们观察的对象其实是 id 为 todo 的这个对象上发生的 keyup 事件 `Rx.Observable.fromEvent(todo, 'keyup')`。那么其实在订阅的代码段中的 input 其实是 keyup 事件才对。好，我们看看到底是什么，将 console.log(input.target.value) 改写成 console.log(input)，看看会怎样呢？是的，我们得到的确实是 KeyboardEvent

![image-20210514172208477](https://raw.githubusercontent.com/silence/blog/assets/assets/20210514172208.png)

**不太过瘾**？那么我们再来做几个小练习，首先将diamante改成下面的样子，其实不用我讲，你应该也可以猜得到，这是要过滤出 `keyCode = 32` 的事件，keyCode 是 Ascii 码，那么这就是要把空格滤出来。

```javascript
let todo = document.getElementById('todo');
let input$ = Rx.Observable.fromEvent(todo, 'keyup');
input$
	.filter(ev => ev.keyCode === 32)
	.subscribe(ev => console.log(ev.target.value));
```

结果我们看到了，按 123456789 都没有反应，直到按了空格

![image-20210514172705494](https://raw.githubusercontent.com/silence/blog/assets/assets/20210514172705.png)

你可能一直在好奇，我们最终只对输入框的值有兴趣，能不能数据流只传值过来呢？当然可以，使用 map 这个变换类操作符就可以完成这个转换了。

```javascript
let todo = document.getElementById('todo');
let input$ = Rx.Observable.fromEvent(todo, 'keyup');
input$
	.map(ev => ev.target.value)
	.subscribe(value => console.log(value));
```

map 这个操作符做的事情就是允许你对原数据流中的每一个元素应用一个函数，然后返回并形成一个新的数据流，这个数据流中的每一个元素都是原来的数据流中的元素应用函数后的值。比如下面的例子，对于原数据流中的每个数据应用一个函数 10 * x，也就是扩大了 10 倍，形成一个新的数据流。

![image-20210514173327813](https://raw.githubusercontent.com/silence/blog/assets/assets/20210514173327.png)

#### 常见操作

最常见的两个操作符我们上面已经了解了，我们继续再来认识新的操作符。类似 `.map(ev => ev.target.value)` 的场景太多了，以至于 rxjs 团队搞出来一个专门的操作符来应对，这个操作符就是 pluck。这个操作符专业从事从一系列嵌套的属性中把值提取出来形成新的流。比如上面的例子可以改写成下面的代码，效果是一样的。那么如果其中某个属性为空怎么办？这个操作符负责返回一个 undefined 作为值加入流中。

```javascript
let todo = docuement.getElementById('todo');
let input$ = Rx.Observable.fromEvent(todo, 'keyup');
input$
	.pluck('target', 'value')
	.subscribe(value => console.log(value));
```

下面我们稍微给我们的页面加点料，除了输入框再加一个按钮

```html
<!DOCTYPE html>
<html>
<head>
	<meta charset="utf-8">
	<meta name="viewport" content="width=device-width">
	<title>JS Bin</title>
	<script src="https://unpkg.com/@reactivex/rxjs@5.0.0-beta.7/dist/global/Rx.umd.js"></script>
</head>
<body>
	<input id="todo" type="text"/>
	<button id="addBtn">Add</button> </body>
</html>
```

在 Javascript 中我们同样方法得到按钮的 DOM 对象以及声明对此按钮点击事件的观察者：

```javascript
let addBtn = document.getElementById('addBtn');
let buttonClick$ = Rx.Observable.fromEvent(addBtn, 'click')
														  .mapTo('clicked');
```

由于点击事件没有什么可见的值，所以我们利用一个操作符叫 mapTo 把对应的每次点击转换成字符 clicked。其实它也是一个 map 的简化操作。

![image-20210514175317143](https://raw.githubusercontent.com/silence/blog/assets/assets/20210514175317.png)

##### 合并类操作符

###### combineLatest 操作符

既然现在我们已经有了两个流，应该试验一下合并类操作符了，先来试试 combineLatest，我们合并了按钮点击事件的数据流，并且返回一个对象，这个对象有两个属性，第一个是按钮事件数据流的值，第二个是文本输入事件数据流的值。也就是说应该是类似 `{ ev: 'clicked', input: '1' }` 这样的结构。

```javascript
Rx.Observable.combineLatest(buttonClick$, input$, (ev, input) => {
  return {
    ev: ev,
    input: input
  }
})
	.subscribe(value => console.log(value))
```

那看看结果如何，在文本输入框输入 1，没反应，再输入 2，还是没反应

![image-20210517195115594](https://raw.githubusercontent.com/silence/blog/assets/assets/20210517195115.png)

那我们点击一下按钮试试，这回有结果了，但有点没明白为什么是 12，输入的数据流应该是：`1，12，...` 但那个 1 怎么丢了呢？

![image-20210517195254942](https://raw.githubusercontent.com/silence/blog/assets/assets/20210517195255.png)

再来文本框输入 3，4 看看，这回倒是都出来了

![image-20210517195349708](https://raw.githubusercontent.com/silence/blog/assets/assets/20210517195349.png)

我们来解释一下 combineLatest 的机制就会明白了，如下图所示，上面的 2 条线是 2 个源数据流（我们分别叫它们源 1 和 源 2 吧），经过 combineLatest 操作符后产生了最下面的数据流（我们称它为结果流）。

当源 1 的数据流发射时，源 2 没有数据，这时候结果流也不会有数据产生，当源 2 发射第一个数据（图中 A）后，combineLatest 操作符做的处理是，把 A 和 源 1 的最近产生的数据（图中 2）组合在一起，形成结果流的第一个数据（图中 2A）。当源 2 产生第二个数据（图中 B）时，源 1 这时没有新的数据产生，那么还是用源 1 中最新的数据（图中 2）和源 2 中最新的数据（图中 B）组合。

也就是说 combineLatest 操作符其实是在组合 2 个源数据流中选择最新的 2 个数据进行配对，如果其中一个源之前没有任何数据产生，那么结果流也不会产生数据。

![image-20210517210530537](https://raw.githubusercontent.com/silence/blog/assets/assets/20210517210530.png)

讲到这里，有童鞋会问，原理是明白了，但什么样的实际需求会需要这个操作符呢？其实有很多，我这里只举一个小例子，现在健身这么热，比如说我们做一个简单的 BMI 计算器，BMI 的计算公式是：体重（公斤）/ 身高（米）。那么我们在页面给出两个输入框和一个用于显示结果的 div:

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width">
<title>JS Bin</title>
<script src="https://unpkg.com/@reactivex/rxjs@5.0.0-beta.7/dist/global/Rx.umd.js"></script>
</head>
<body>
Weight: <input type="number" id="weight"> kg <br/>
Height: <input type="number" id="height"> cm <br/>
Your BMI is <div id="bmi"></div> </body>
</html>
```

那么在 JS 中，我们想要达成的结果是只有两个输入框都有值的时候才能开始计算 BMI，这时你发现 combineLatest 的逻辑不要太顺溜啊。

```javascript
let weight = document.getElementById('weight');
let height = document.getElementById('height');
let bmi = document.getElementById('bmi');

let weight$ = Rx.Observable.fromEvent(weight, 'input').pluck('target', 'value');

let height$ = Rx.Observable.fromEvent(height, 'input').pluck('target', 'value');

let bmi$ = Rx.Observable.combineLatest(weight$, height$, (w, h) => w/(h*h/100/100));

bmi$.subscribe(b => bmi.innerHTML = b);
```

![image-20210517211318150](https://raw.githubusercontent.com/silence/blog/assets/assets/20210517211318.png)

###### zip 操作符

除了 combineLatest，Rxjs 还提供了多个合并类的操作符，我们再试验一个 zip 操作符。zip 和 combineLatest 非常像，但重要的区别点在于 zip 严格的需要多个源数据流中的每一个的相同顺序的元素配对。

比如说还是上面的例子，zip 要求源 1 的第一个数据和源 2 的第一个数据组成一对，产生结果流的第一个数据；源 1 的第二个数据和源 2 的第二个数据组成一对，产生结果流的第二个数据。而 combineLatest 不需要等待另一个源数据流产生数据，只要有一个产生，结果流就会产生。

![image-20210517212301420](https://raw.githubusercontent.com/silence/blog/assets/assets/20210517212301.png)

**zip 这个词在英文中有拉链的意思，记住这个有助于我们理解这个操作符**，就像拉链一样，它需要拉链两边的齿一一对应。从效果角度上讲，这个操作符有减缓发射速度的作用，因为它会等待合并序列中最慢的那个。

下面我们还是看个例子，在我写第七章的使用 Bing Image API 变换背景时，我最开始的想法是取得图片数组后，把这个数组中的元素每隔一段时间发送出去一个，这样组件端就不用关心图片变化的逻辑，只要服务发射一个地址，我就加载就行了。我就是用 zip 来实现的，我们在这个逻辑中有 2 个源数据流：基于一个数组生成的数据流以及一个时间间隔数据流。前者的发射速度非常快，后者则速度均匀，我们希望后者的速度对齐前者，以达到每隔一段时间发射前者的数据的目的。

```javascript
yieldByInterval(items, time) {
  return Observable.from(items).zip(
  	Observable.interval(time),
    (item, index) => item
  )
}
```

为了更好的让大家体会，我改写一个纯 javascript 版本，可以在 JSBin 上面直接跑的，它的本质逻辑和上面讲的相同：

```javascript
let greetings = ['Hello', 'How are you', 'How are you doing'];
let time = 3000;
let item$ = Rx.Observable.from(greetings);
let interval$ = Rx.Observable.interval(time);

Rx.Observable.zip(
	item$,
  interval$,
  (item, index) => {
    return {
      item,
      index
    }
  }
).subscribe(result => {
  console.log('item: ' + result.item + ' index: ' + result.index + '  at ' + new Date())
});
```

我们看到结果如下图所示，每隔 3000 毫秒，数组中的欢迎文字被输出一次。

![image-20210517213347556](https://raw.githubusercontent.com/silence/blog/assets/assets/20210517213347.png)

这两个操作符应该是 Rx 中最常用的 2 个合并类操作符了。其他操作符大家可以去 RxJS 官网查看，注意不是所有的操作符 RxJS 都有。

##### 创建类操作符

通常来讲，Rx 团队不鼓励新手自己从 0 开始创建 Observable，因为状态太复杂，会遗漏一些问题。Rx 鼓励的是通过已有的大量创建类转换操作符来去建立 Observable。我们其实之前已经见过一些了，包括 from 和 fromEvent。

###### from 操作符

from 可以支持从数组、类似数组的对象、Promise、iterable 对象或类似 Observable 的对象（其实这个主要指 ES 2015 中的 Observable）来创建一个 Observable。

这个操作符应该是可以创建 Observable 的操作符中最常使用的一个，因为它几乎可以把任何对象转换成 Observable。

```javascript
var array = [10, 20, 30];
var result$ = Rx.Observable.from(array);
result$.subscribe(x => console.log(x));
```

![image-20210517213947852](https://raw.githubusercontent.com/silence/blog/assets/assets/20210517213947.png)

###### fromEvent 操作符

这个操作符专门为事件转成 Observable 而制作的，非常强大且方便。对于前端来说，这个方法用于处理各种 DOM 中的事件再方便不过了。

```javascript
var click$ = Rx.Observable.fromEvent(docuement, 'click');
click$.subscribe(x => console.log(x));
```

![image-20210517214237073](https://raw.githubusercontent.com/silence/blog/assets/assets/20210517214237.png)

###### fromEventPattern 操作符

我们经常会遇到一些已有的代码，这些代码和类库往往不受我们的控制，无法重构或代价太大。我们需要在这种情况下可以利用 Rx 的话，就需要大量的可以从原有的代码中可以转换的方法。addXXXHandler 和 removeXXXHandler 就是大家以前经常使用的一种模式，那么在 Rx 中也提供了对应的方法可以转换，那就是

```javascript
function addClickHandler(handler) {
  document.addEventListener('click', handler)
}

function removeClickHandler(handler) {
  document.removeEventListener('click', handler);
}

var click$ = Rx.Observable.fromEventPattern(
	addClickHandler,
  removeClickHandler
);

click$.subscribe(x => console.log(x));
```

![image-20210517214840863](https://raw.githubusercontent.com/silence/blog/assets/assets/20210517214840.png)

###### defer 操作符

defer 是直到有订阅者之后才创建 Observable，而且它为每个订阅者都会这样做，也就是说其实每个订阅者都是接收到自己的单独数据流序列。

![image-20210518113116100](https://raw.githubusercontent.com/silence/blog/assets/assets/20210518113116.png)

```javascript
Rx.Observable.defer(() => {
  let result = doHeavyJob();
  return result ? 'success' : 'failed';
})
	.subscribe(x => console.log(x))

function doHeavyJob() {
  setTimeout( function() { console.log('doing something') }, 2000 )
  return true
}
```

###### Interval 操作符

Rx 提供内建的可以创建和计时器相关的 Observable 方法，第一个是 Interval，它可以在指定时间间隔发送整体的自增长序列。

![image-20210518114101300](https://raw.githubusercontent.com/silence/blog/assets/assets/20210518114101.png)

例如下面代码，我们每隔 500 毫秒发送一个整数，这个数列是无穷的，我们取前三个好了：

```javascript
let source = Rx.Observable.interval(500 /* ms */).take(3);

let subscription = source.subscribe(function (x) {
  console.log('Next: ' + x);
}, function (err) {
  console.log('Error: ' + err);
}, function () {
  console.log('Completed');
})
```

那么输出是

![image-20210518114453003](https://raw.githubusercontent.com/silence/blog/assets/assets/20210518114453.png)

这里大家可能注意到我们没有采用箭头的方式，而是用传统的写法，写了 `function (x) {...}`，哪种方式其实都可以，箭头函数会更简单些。

另一个需要注意的地方是，在 subscribe 方法中我们多了 2 个参数：一个处理异常，一个处理完成。Rx 认为所有的数据流会有三个状态：next，error 和 completed。这三个函数就是分别处理这三种状态的，当然如果我们不写某个状态的处理，也就意味着我们认为此状态不需要特别处理。而且有些序列是没有 completed 状态的，因为是无限序列。本例中，如果我们去掉 `.take(3)` 那么 completed 是永远无法触发的。

###### Timer 操作符

下面我们来看看 Timer，一共有 2 种形式的 Timer，**一种是指定时间后返回一个序列中只有一个元素（值为 0 ）的 Observable**。

```javascript
// 这里指定一开始的 delay 时间
// 也可以输入一个 Data，比如： "2016-12-31 20:00:00"
// 这样变成了在指定的时间触发
let source = Rx.Observable.timer(2000);

let subscription = source.subscribe(
	x => console.log('Next: ' + x),
  err => console.log('Error: ' + err),
  () => console.log('Completed')
)
```

![image-20210518115543486](https://raw.githubusercontent.com/silence/blog/assets/assets/20210518115543.png)

第二种 Timer 很类似于 Interval。除了第一个参数是一开始的延迟时间，第二个参数是间隔时间，也就是说，**在一开始的延迟时间后，每隔一段时间就会返回一个整数序列**。这个和 Interval 基本一样，除了 Timer 可以指定什么时间开始（延迟时间）。

```javascript
var source = Rx.Observable.timer(2000, 100).take(3);

var subscription = source.subscribe(
	x => console.log('Next: ' + x),
  err => console.log('Error: ' + err),
  () => console.log('Completed')
)
```

![image-20210518120008793](https://raw.githubusercontent.com/silence/blog/assets/assets/20210518120008.png)

当然还有其他创建类的操作符，大家可以去 RxJS 官网查阅自行试验一下。

##### 过滤类操作符

之前我们见过好几个过滤类操作符：filter，distinct，take 和 debounce。

###### filter 操作符

Filter 操作只允许数据流中满足其 predicate 测试的元素发射出去，这个 predicate 函数接受 3 个参数：

1. 原始数据流元素
2. 索引，这个是指该元素在源数据流中的位置（从 0 开始）
3. 源 Observable 对象

如下的代码将 0 - 5 中的偶数过滤出来：

```javascript
let source = Rx.Observable.range(0, 5)
	.filter(function(x, idx, obs) {
    return x % 2 === 0;
  });

let subscription = source.subscribe(
	x => console.log('Next: ' + x),
  err => console.log('Error: ' + err),
  () => console.log('Completed')
)
```

![image-20210518131220337](https://raw.githubusercontent.com/silence/blog/assets/assets/20210518131220.png)

###### debounceTime 操作符

对于一些发射频率比较高的数据流，我们有时会想给它安个“整流器”。比如在一个搜索框中，输入一些字符后希望出现一些搜索建议，这是个非常好的功能，很多时候可以减少用户的输入。

但是由于这些搜索建议需要联网完成数据的传递，如果太频繁操作的话，对于用户的数据流量和服务器的性能承载都是有副作用的。所以我们一般希望在用户连续快速输入时不去搜索，而是等待有相对较长的间隔时再去搜索。

下面的代码**从输入上做了这样的一个“整流器”，滤掉了间隔时间小于 400 毫秒的输入事件（输入本身不受影响），只有用户出现较明显的停顿时才把输入值发射出来**。

```javascript
let todo = document.getElementById('todo');
let input$ = Rx.Observable.fromEvent(todo, 'keyup');

input$.debounceTime(400).subscribe(input => console.log(input.target.value));
```

快速输入 “12345”，在这种情况下得到的是一条数据

![image-20210518132048948](https://raw.githubusercontent.com/silence/blog/assets/assets/20210518132049.png)

但如果不应用 debounceTime，我们得到 5 条记录

![image-20210518132128907](https://raw.githubusercontent.com/silence/blog/assets/assets/20210518132129.png)

其它的过滤类操作符也很有趣，比如 **Distinct 就是可以把重复的元素过滤掉**，**skip 就可以跳过几个元素**等等，可以自行研究，这里就不一一举例了。

Rx 的操作符实在太多了，我只能列举一些较常见的给大家介绍一下，其他的建议大家去官方文档学习。

#### Angular 2 中的内建支持

Angular 2 中对于 Rx 的支持是怎么样的呢？先试验一下吧，简单粗暴的一个组件模板页面

```html
<p>
  {{clock}}
</p>
```

和在组件中定义一个简单粗暴的成员变量

```javascript
import { Component } from '@angular/core';

import { Observable } from 'rxjs/Observable';
import 'rxjs/add/observable/internal';

@Component({
  selector: 'app-playground',
  templateUrl: './playground.component.html',
  styleUrls: ['./playground.component.css']
})
export class PlaygroundComponent {
  clock = Observable.interval(1000);

	constructor() {}
}
```

搞定！打开浏览器，显示了一个 `[object Object]`，**晕倒**。

![image-20210518132913484](https://raw.githubusercontent.com/silence/blog/assets/assets/20210518132913.png)

当然经过前面的学习，我们知道 Observable 是个异步数据流，我们可以把代码改写一下，在订阅方法中去赋值就一切 ok 了。

```typescript
import { Component } from '@angular/core';

import { Observable } from 'rxjs/Observable';
import 'rxjs/add/observable/interval';

@Component({
  selector: 'app-playground',
  templateUrl: './playground.component.html',
  styleUrls: ['./playground.component.css']
})
export class PlaygroundComponent {
  clock: number;
	
  constructor() {
    Observable.interval(1000).subscribe(value => this.clock = value)
  }
	
}
```

![image-20210518133336943](https://raw.githubusercontent.com/silence/blog/assets/assets/20210518133337.png)

但是这样做还是有一个问题，我们加入一个 do 操作符，在每次订阅前去记录就会发现一些问题。当我们离开页面再回来，每次进入都会创建一个新的订阅，但原有的没有释放。

```javascript
Observable.interval(1000)
	.do(_ => console.log('observable created'))
	.subscribe(value => this.clock = value);
```

观察 console 中在 'observable created' 之前的数字和页面显示的数字，大概是页面每增加 1 ，console 的数字增加 2，这说明我们后面运行着 2 个订阅。

![image-20210518133749370](https://raw.githubusercontent.com/silence/blog/assets/assets/20210518133749.png)

原因是我们没有在页面销毁时取消订阅，那么我们利用生命周期的 onDestroy 来完成这一步：

```javascript
import { Component, OnDestroy } from '@angular/core';

import { Observable } from 'rxjs/Observable';
import { Subscription } from 'rxjs/Subscription';
import 'rxjs/add/observable/interval';

@Component({
  seletor: 'app-playground',
  templateUrl: './playground.component.html',
  styleUrls: ['./playground.component.css']
})
export class PlaygroundComponent implements OnDestroy {
  clock: number;
	subscription: Subscription;

	constructor() {
    this.subscription = Observable.interval(1000)
    .do(_ => console.log('observable created'))
    .subscribe(value => this.clock = value)
  }

	ngOnDestroy() {
    if(this.subscription !== undefined)
      this.subscription.unsubscibe();
  }
}
```

现在再来观察，同样进入并离开再进入页面后，页面每增加 1 ，console 也会增加 1。

![image-20210518134508006](https://raw.githubusercontent.com/silence/blog/assets/assets/20210518134508.png)

###### Async 管道

现在看起来还是挺麻烦的，有没有更简单的方法呢？答案当然是肯定的：Angular 2 提供一个管道叫：async，有了这个管道，我们无需管理琐碎的取消订阅，以及订阅了。

让我们回到最开始的简单粗暴版本，模板文件稍微改下一下

```html
<p>
  {{ clock | async }}
</p>
```

这个 `| async` 是什么东东？async 是 Angular 2 提供的一种转换器，叫管道（Pipe）。

每个应用开始的时候差不多都是一些简单任务：获取数据、转换它们，然后把它们显示给用户。一旦取到数据，我们可以把它们原始值的结果直接显示。但这种做法很少能有好的用户体验。比如，几乎每个人都更喜欢简单的日期格式，几月几号星期几，而不是原始字符串格式 --- Fri Apr 15 1988 00:00:00 GMT-0700 (Pacific Daylight Time)。通过管道我们可以把不友好的值转换成友好的值显示在页面中。

Angular 内置了一些管道，比如 DatePipe、UpperCasePipe、LowerCasePipe、CurrencyPipe 和 PercentPipe。它们全都可以直接用在任何模板中。Async 管道也是内置管道之一。

当然这样在页面写完管道后，我们的组件也回归了简单粗暴版本：

```javascript
import { Component, OnDestroy } from '@angular/core'; 

import { Observable } from 'rxjs/Observable';
import 'rxjs/add/observable/interval';

@Component({
	selector: 'app-playground',
	templateUrl: './playground.component.html',
  styleUrls: ['./playground.component.css']
})
export class PlaygroundComponent {
	clock = Observable.interval(1000).do( _ =>console.log('observable created'));
  constructor() { }
}
```

现在打开浏览器，看一下页面的效果

![image-20210518135520097](https://raw.githubusercontent.com/silence/blog/assets/assets/20210518135520.png)



 

