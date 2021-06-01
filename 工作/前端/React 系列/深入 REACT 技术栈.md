[TOC]

本文为对书籍 《深入 REACT 技术栈》的一些重点摘抄

### 第2章 漫谈 React

#### 2.1 事件系统

在 React 底层，主要对合成事件做了两件事：事件委派和自动绑定。

1. **事件委派**

在使用 React 事件前，**一定要熟悉它的事件代理机制。它并不会把事件处理函数直接绑定到真实的节点上，而是把所有事件绑定到结构的最外层，使用一个统一的事件监听器，这个事件监听器上维持了一个映射来保存所有组件内部的事件监听和处理函数。当组件挂载或卸载时，只是在这个统一的事件监听器上插入或删除一些对象；当事件发生时，首先被这个统一的事件监听器处理，然后在映射里找到真正的事件处理函数并调用**。这样做简化了事件处理和回收机制，效率也有了很大的提升。

2. **自动绑定**

在 React 组件中，每个方法的上下文都会指向该组件的实例，即自动绑定 this 为当前组件。而且 React 还会对这种引用进行缓存，以达到 CPU 和内存的最优化。在使用 ES6 classes 或者纯函数时，这种自动绑定就不复存在了，我们需要手动实现 this 的绑定。

#### 2.4 组件间通信

​	一般情况下，组件之间的通信尽可能保存简洁。如果说程序中出现多级传递或跨级传递时，那么首先要重新审视一下是否有更合理的方式。Pub/Sub 模式的实现的过程非常容易理解，即**利用全局对象来保存事件，用广播的方式去处理事件**。这种常规的设计方法在软件开发中处处可见，但这种模式带来的问题就是**逻辑关系混乱**。

​	在上述几种通信模式中，跨级通信往往是反模式的典型案例。对于应用开发来说，应该尽力避免仅仅通过例如 Pub/Sub 实现的设计思路，加入强依赖与约定来进一步梳理流程是更好的方法，这将在第 4 章再深入讨论。

#### 2.5 组件间抽象

在 React 组件的构建过程中，常常有这样的场景，有一类功能需要被不同的组件公用，此时就涉及抽象的话题。在不同的设计理念下，有许多抽象方法，而针对 React，我们重点讨论两种：mixin 和高阶组件。

##### 2.5.1 mixin

1. 使用 mixin 的缘由

在 Ruby 中，include 关键词即是 mixin，是将一个模块混入到一个另一个模块中，或是一个类中。为什么编程语言要引入这样一种特性呢？事实上，包括 C++ 等一些年龄较大的 OOP 语言，它们都有一个强大但危险的多重继承特性。现代语言为了权衡利弊，大都舍弃了多重继承，只采用单继承，但单继承字啊实现抽象时有诸多不便之处。为了弥补缺失，Java 引入了接口（interface），其他一些语言则引入了像 mixin 的技巧，方法虽然不同，但都是为创造一种类似 多重继承 的效果，事实上说它是组合更为贴切。

2. 封装 mixin 方法

看到这里，我们已经知道了广义的 mixin 方法的作用，现在试着自己封装一个 mixin 方法来感受一下：

```javascript
const mixin = function(obj, mixins) {
  // 拷贝 obj 里面的属性
  const newObj = obj;
  // 继承 obj 的 prototype
  newObj.prototype = Object.create(obj.prototype)
  // 将 mixins 里面的方法拷贝到 newObj 的 prototype 上
  for (let prop in mixins) {
    if (mixins.hasOwnProperty(prop)) {
      newObj.prototype[prop] = mixins[prop]
    }
  }
  
  return newObj;
}

const BigMixin = {
  fly: () => {
    console.log('I can fly')
  }
}

const Big = function() {
  console.log('new big')
}

const FlyBig = mixin(Big, BigMixin)

const flyBig = new FlyBig(); // => 'new big'
flybig.fly(); // => 'I can fly'
```

对于广义的 mixin 方法，就是用赋值的方式将 mixin 对象里的方法都挂载到原对象上，来实现对对象的混入。

4. ES6 Classes 与 decorator

Core-decorators 库为开发者提供了一些实用的 decorator，其中实现了我们正想要的 `@mixin`。下面解读一下其核心实现：

```javascript
import { getOwnPropertyDescriptors } from './private/utils';

const { defineProperty } = Object;

function handleClass(target, mixins) {
  if (!mixins.length) {
    throw new SyntaxError(`@mixin() class ${target.name} requires at least one mixin as an argument`);
  }
  
  for (let i = 0, l = mixins.length; i < l; i++) {
    // 获取 mixins 的 attributes 对象
    const descs = getOwnPropertyDescriptors(mixins[i])
    
    // 批量定义 mixins 的 attributes 对象
    for (const key in descs) {
      if (!(key in target.prototype)) {
        defineProperty(target.prototype, key, descs[key])
      }
    }
  }
}

export default function mixin(...mixins) {
  if (typeof mixins[0] === 'function') {
    return handleClass(minxins[0], [])
  }else {
    return (target) => {
      return handleClass(target, mixins)
    }
  }
}
```

可以看到，源代码十分简单，它将每一个 mixin 对象的方法都叠加到 target 对象的原型上以达到 mixin 的目的。这样，就可以用 `@mixin` 来做多个重用模块的叠加了。比如：

```javascript
import React, { Component } from 'react';
import { mixin } from 'core-decorators';

const PureRender = {
  shouldComponentUpdate() {}
}

const Theme = {
  setTheme() {}
};

@mixin(PureRender, Theme)
class MyComponent extends Component {
  render() {}
}
```

细心的你应该已经发现了这个 mixin 与 createClass 中的 mixin 的区别。上述实现中，mixin 的逻辑和最早实现的简单逻辑很相似，之前直接给对象的 prototype  属性赋值，但这里用了 `getOwnPropertyDescriptor` 和 `defineProperty` 这两个方法， 有什么区别呢？

事实上，这样实现的好处在于 `defineProperty` 这个方法，也就是定义与赋值的区别，定义是对已有的定义，赋值则是覆盖已有的定义。所以说前者并不会覆盖已有方法，但后者会。

5. mixin 的问题

我们认可 mixin 给组件开发带来抽象的好处，但随着大量使用 mixin，它的问题也渐渐暴露出来。Dan Abramov 是最早提出这个问题的人，它总结了 mixin 最大的一些问题。

- 破坏了原有组件的封装

  我们知道 mixin 方法会混入方法，给原有组件带来新的特性，比如 mixin 中有一个 renderList 方法，给我们带来了渲染 List 的能力，但它也可能带来了新的 state 和 props，这意味着组件有一 些“不可见”的状态需要我们去维护，但我们在使用的时候并不清楚。此外，renderList 中的方 法会有调用组件中的方法，但很可能被其他 mixin 截获，带来很多不可知。

  另外，mixin 也有可能去依赖其他的 mixin，这样会建立一个 mixin 的依赖链，当我们改动其 中一个 mixin 的状态时，很可能会直接影响其他的 mixin。解决方法是可以约定好输入和输出。 但不幸的是，mixin 是平面结构，所有方法都在同一个环境中，我们没法做到很好的约定。

- 命名冲突

  刚才也提到了，mixin 是平面结构，那么不同 mixin 中的命名在不可知的情况，重用的情况 是不可控的。尤其是像 handleChange 这样常见的名字，我们不能在两个 mixin 中同时使用，也不 能在自己的组件中使用这个名字的方法。

  尽管我们可以通过更改名字来解决，但遇到第三方引用，或已经引用了几个 mixin 的情况下， 总是要花一定的成本去解决冲突。

- 增加复杂性

  在过去写 mixin 的时候，是不是常遇到这样的情形:我们设计一个组件，引入名为 PopupMixin 的 mixin，这样就给组件引进了 PopupMixin 生命周期方法，还有 hidePopup()、startPopup() 等 方法。当我们再引入HoverMixin时，将有更多的方法被引进，比如 handleMouseEnter()、handleMouseLeave()、isHovering()方法。当然，我们可以进一步抽象出 TooltipMixin，将两个整合在 一起，但我们发现它们都有 componentDidUpdate 方法。

  几个月后，再去看组件的实现时，会发现代码已经没法维护，它的逻辑已经复杂到难以理解。 写 React 组件时，我们首先考虑的往往是单一的功能、简洁的设计和逻辑。当加入功能的时候， 可以继续控制组件的输入和输出。如果说因为复杂性，我们不断加入新的状态，那么组件肯定会 因此变得非常难以维护。

  针对这些困扰，React 社区提出了新的方式来取代 mixin，那就是高阶组件。

##### 2.5.2 高阶组件

下面我们再来讨论一下高阶组件与 mixin 的不同之处，如下图所示：

![](https://raw.githubusercontent.com/silence/blog/assets/assets/20210508153947.png)

图 2-1 其实已经很清晰地表达了 mixin 与高阶组件的不同之处。简单来说，高阶组件符合函 数式编程思想。对于原组件来说，并不会感知到高阶组件的存在，只需要把功能套在它之上就可 以了，从而避免了使用 mixin 时产生的副作用。

#### 2.6 组件性能优化

##### 2.6.3 Immutable

1. Immutable Data

   **Immutable Data 就是一旦创建，就不能再更改的数据。**对 Immutable 对象进行修改、添加或删除操作，都会返回一个新的 Immutable 对象。Immutable 实现的原理是持久化的数据结构 （persistent data structure），也就是使用旧数据创建新数据时，要保证旧数据同时可用且不变。**同时为了避免深拷贝把所有节点都复制一遍带来的性能损耗，Immutable 使用了结构共享（structural sharing），即如果对象树中一个节点发生变化，只修改这个节点和受它影响的父节点，其它节点则进行共享。**

2. **Immutable 的优点**

   Immutable 的优点有如下几点。

   - **降低了“可变”带来的复杂度**。可变数据耦合了 time 和 value 的概念，造成了数据很难被回溯。比如：

     ```javascript
     function touchAndLog(touchFn) {
       let data = { key: 'value' };
       touchFn(data);
       console.log(data.key);
     }
     ```

     在不查看 touchFn 的代码的情况下，因为不确定方法对 data 做了什么，我们是不可能知道结果是什么。但如果 data 是不可变的呢，你会很肯定地知道打印的结果是 value。

   - **节省内存**。Immutable 使用结果共享尽量复用内存。没有被引用的对象会被垃圾回收：

     ```javascript
     import { Map } from 'immutable';
     
     let a = Map({
       select: 'users',
       filter: Map({ name: 'Cam' })
     });
     
     let b = a.set('select', 'people');
     
     a === b; // => false
     
     a.get('filter') === b.get('filter'); // => true
     ```

     上面 a 和 b 共享了没有变化的 filter 节点。

   - **撤销/重做，复制/粘贴，甚至时间旅行这些功能做起来都是小菜一碟**。因为每次数据都是不一样的，那么只要把这些数据放到一个数组里存储起来，想回退到哪里，就拿出对应的数据，这很容易开发出撤销及重做这两种功能。

   - **并发安全**。传统的并发非常难做，因为要处理各种数据不一致的问题，所以“聪明人”发明了各种锁来解决问题。但使用了 Immutable 之后，数据天生是不可变的，**并发锁就不再需要了**。然而现在并没有用，因为 Javascript 还是单线程运行的。

   - **拥抱函数式编程。**Immutable 本身就是函数式编程中的概念。只要输入一致，输出必然一致，这样开发的组件更易于调试和组装。

3. 使用 Immutable 的缺点

   **容易与原生对象混淆是使用 Immutable 的过程中遇到的最大的问题。**

   虽然 Immutable 尽量把 API 设计的原生对象类似，但还是很难区分到底是 Immutable 对象还是原生对象。

   当使用第三方库的时候，一般需要使用原生对象，同样容易忘记转换对象。下面给出一些办法来避免类似问题的发生：

   - 使用 FlowType 或 Typescript 静态类型检查工具；
   - 约定变量命名规则，如所有 Immutable 类型对象以 $$ 开头；
   - 使用 Immutable.fromJS 而不是 Immutable.Map 或 Immutable.List 来创建对象，这样可以避免 Immutable 对象和原生对象间的混用。

#### 2.8 自动化测试

测试可以让项目保持健壮，在后期维护和拓展的过程中，减少犯错的几率。当项目发布时，代码能通过所有测试也代表所覆盖到的场景全部通过。

#### 2.9 小结

之前我们也讲到，对于数组或对象类型的 props 而言，优化的最直接手段就是使用 Immutable。经过测试，Tabs 组件大大减少了无意义的渲染次数。

### 第3章 解读 React 源码

本章会通过分析 **React 15.0** 的源码，深入 Virtual DOM 内部的实现机制和原理，让我们一步步揭开 Virtual DOM 的神秘面纱，探索其内部的精彩世界！

#### 3.1 初探 React 源码

在深入分析 React 源码之前，我们先大致了解一下 React 源码的组织结构，如图 3-1 所示。

![](https://raw.githubusercontent.com/silence/blog/assets/assets/20210508180415.png)

在 React 源码中，每个文件的命名风格属于字面与含义可相互解释，整体的代码结构按照 `addons`、`isomorphic`、`renderers`、`shared`、`core`、`test` 进行组织。

- **addons**:包含一系列的工具方法插件，如 PureRenderMixin、CSSTransitionGroup、Fragment、 LinkedStateMixin 等。
- **isomorphic**:包含一系列同构方法。
- **shared**:包含一些公用或常用方法，如 Transaction、CallbackQueue 等。
- **test**:包含一些测试方法等。
- **core/tests**:包含一些边界错误的测试用例。
- **renderers**:是 React 代码的核心部分，它包含了大部分功能实现，此处对其进行单独分析。

renders 分为 dom 和 shared 目录。

1. **dom**:包含 client、server 和 shared。

   - **client**:包含 DOM 操作方法(如 findDOMNode、setInnerHTML、setTextContent 等)以 及事件方法，结构如图 3-2 所示。这里的事件方法主要是一些非底层的实用性事件方法， 如 事 件 监 听 ( ReactEventListener )、 常 用 事 件 方 法 ( TapEventPlugin 、 EnterLeave- EventPlugin)以及一些合成事件(SyntheticEvents 等)。

     ![](https://raw.githubusercontent.com/silence/blog/assets/assets/20210508181001.png)

   - **server**:主要包含服务端渲染的实现和方法(如 ReactServerRendering、ReactServer- RenderingTransaction 等)。

   - **shared**:包含文本组件(ReactDOMTextComponent)、标签组件(ReactDOMComponent)、 DOM 属性操作(DOMProperty、DOMPropertyOperations)、CSS 属性操作(CSSProperty、 CSSPropertyOperations)等。

2. **shared**:包含 event 和 reconciler。

   - **event**:包含一些更为底层的事件方法，如事件插件中心(EventPluginHub)、事件注册(EventPluginRegistry)、事件传播(EventPropagators)以及一些事件通用方法。

     React 自定义了一套通用事件的插件系统，该系统包含事件监听器、事件发射器、事件插件中心、点击事件、进/出事件、简单事件、合成事件以及一些事件方法，如图 3-3 所示。

     ![](https://raw.githubusercontent.com/silence/blog/assets/assets/20210508184947.png)

   - **reconciler**:称为协调器，它是最为核心的部分，包含 **React 中自定义组件的实现** (ReactCompositeComponent)、**组件生命周期机制**、**setState 机制**(ReactUpdates、ReactUpdateQueue)、**DOM diff 算法**(ReactMultiChild)等重要的特性方法。

React 也能够实现 Virtual DOM 的批处理更新，当操作 Virtual DOM 时，不会马上生成真实的 DOM，而是会将一个事件循环（event loop）内的两次数据更新进行合并，这样就使得 React 能够在事件循环的结束之前完全不用操作真实的 DOM。例如，多次进行节点内容 A => B，B => A 的变化，React 会将多次数据更新合并为 A => B => A，即 A => A，任务数据并没有更新，因此 UI 也不会发生任何变化。如果通过手动控制，这种逻辑通常是及其复杂的。

#### 3.2 Virtual DOM 模型

Virtual DOM 中的节点称为 ReactNode，它分为 3 种类型 ReactElement、ReactFragment 和 ReactText。其中，ReactElement 又分为 ReactComponentElement 和 ReactDOMElement。

下面是 ReactNode 中不同类型节点所需要的基础元素：

```typescript
type ReactNode = ReactElement | ReactFragment | ReactText;

type ReactElement = ReactComponentElement | ReactDOMElement;

type ReactDOMElement = {
  type: string,
  props: {
    children: ReactNodeList,
    className: string;
    etc.
  }
  key: string | boolean | number | null,
  ref: string | null
};

type ReactComponentElement<TProps> = {
  type: ReactClass<TProps>,
  props: TProps,
  key: string | boolean | number | null,
  ref: string | null
};

type ReactFragment = Array<ReactNode | ReactEmpty>;

type ReactNodeList = ReactNode | ReactEmpty;

type ReactText = string | number;

type ReactEmpty = null | undefined | boolean;
```

##### 3.2.5 自定义组件

React 的主要思想是通过构建可复用组件来构建用户界面。**所谓组件，其实就是有限状态机（FSM），通过状态渲染对应的界面，且每个组件都有自己的生命周期，它规定了组件的状态和方法需要在哪个阶段改变和执行**。

**有限状态机，表示有限个状态以及在这些状态之间的转移和动作等行为的模型。一般通过状态、事件、转换和动作来描述有限状态机**。状态机能够记住目前所处的状态，可以根据当前的状态做出相应的决策，并且可以在进入不同的状态时做不同的操作。状态机将复杂的关系简单化，利用这种自然而直观的方式可以让代码更容易理解。

React 正是利用这一概念，通过管理状态来实现对组件的管理。例如，某个组件有显示和隐藏两个状态，通常会设计两个方法 show() 和 hide() 来实现切换，而 React 只需要设置状态 `setState({ showed: true/false })` 即可实现。同时，React 还引入了组件的生命周期这个概念。通过它，就可以实现组件的状态机控制，从而达到 “生命周期 => 状态 => 组件” 的和谐画面。

#### 3.4 解密 setState 机制

##### 3.4.1 setState 异步更新

React 初学者常会写出 `this.state.value = 1` 这样的代码，这样是完全错误的写法。

> 注意：绝对不要直接修改 this.state，这不仅是一种低效的做法，而且很有可能会被之后的操作替换。

`setState` 通过一个队列机制实现 state 更新。当执行 setState 时，会将需要更新的 state 合并后放入状态队列，而不会立刻修改更新 this.state，队列机制可以高效地批量更新 state。如果不通过 setState 而直接修改 this.state 的值，那么该 state 将不会被放入状态队列中，当下次调用 setState 并对状态队列进行合并时，将会忽略之前直接被修改的 state，而造成无法预知的错误。

因此，应该使用 setState 方法来更新 state，同时 React 也正是利用状态队列机制实现了 setState 的异步更新，避免频繁地重复更新 state。相关代码如下：

```javascript
// 将新的 state 合并到状态更新队列中
var nextState = this._processPendingState(nextProps, nextContext);

// 根据更新队列和 shouldComponentUpdate 的状态来判断是否需要更新组件
var shouldUpdate = this._pendForceUpdate || !inst.shouldComponentUpdate || inst.shouldComponentUpdate(nextProps, nextState, nextContext);
```

##### 3.4.2 setState 循环调用风险

当调用 `setState` 时，实际上会执行 `enqueueSetState` 方法，并对 `partialState` 以及 `_pendingStateQueue` 更新队列进行合并操作，最终通过 `enqueUpdate` 执行 `state` 更新。

而 `performUpdateIfNecessary` 方法会获取 `_pendingElement`、`_pendingStateQueue`、`_pendingForceUpdate`，并调用 `receiveComponent` 和 `updateComponent` 方法进行组件更新。

如果在 `shouldComponentUpdate` 或 `componentWillUpdate` 方法中调用 `setState`，此时 `this._pendingStateQueue != null`，则 `performUpdateIfNecessary` 方法就会调用 `updateComponent` 方法进行组件更新，但 `updateComponent` 方法又会调用 `shouldComponentUpdate` 和 `componentWillUpdate` 方法，因此造成循环调用，使得浏览器内存占满后崩溃，如图 3-14 所示。

<img src="https://raw.githubusercontent.com/silence/blog/assets/assets/20210509172448.png" alt="a"  />

接下来我们来看 setState 的源码：

```javascript
// 更新 state
ReactComponent.prototype.setState = function(partialState, callback) {
  this.updater.enqueueSetState(this, partialState);
  
  if (callback) {
    this.updater.enqueueCallback(this, callback, 'setState');
  }
}

enqueueSetState: function(publicInstance, partialState) {
  var internalInstance = getInternalInstanceReadyForUpdate(publicInstance, 'setState');
  
  if (!internalInstance) {
    return;
  }
  
  // 更新队列合并操作
  var queue = internalInstance._pendingStateQueue || (internalInstance._pendingStateQueue = [])
  
  queue.push(partialState);
  enqueueUpdate(internalInstance);
},
  
// 如果存在 _pendingElement、_pendingStateQueue 和 _pendingForceUpdate，则更新组件
  performUpdateIfNecessary: function(transaction) {
    if (this._pendingElement != null) {
      ReactReconciler.receiveComponent(this, this._pendingElement), transaction, this._context;
    }
    
    if (this._pendingStateQueue !== null || this._pendingForceUpdate) {
      this.updateComponent(transaction, this._currentElement, this._currentElement, this._context, this._context);
    }
  }
```

##### 3.4.3 setState 调用栈

既然 setState 最终是通过 enqueueUpdate 执行 state 更新，那么 enqueueUpdate 到底是如何更新 state 的呢？

首先，看看下面这个问题，你是否能够正确回答呢？

```javascript
import React, { Component } from 'react';

class Example extends Component {
  constructor() {
    super();
    this.state = {
      val: 0
    };
  }
  
  componentDidMount() {
    this.setState({ val: this.state.val  + 1 });
    // 第 1 次输出
    console.log(this.state.val);
    
    this.setState({ val: this.state.val + 1 });
    // 第 2 次输出
    console.log(this.state.val);
    
    setTimeout(() => {
      this.setState({ val: this.state.val + 1 });
      // 第 3 次输出
      console.log(this.state.val);
      
      this.setState({ val: this.state.val + 1 });
      // 第 4 次输出
      console.log(this.state.val);
    })
  }
  
  render() {
    return null;
  }
}
```

上述代码中，4 次 console.log 打印出来的 val 分别是：0、0、2、3。

假如结果与你心中的答案不完全相同，那么你应该会感兴趣 enqueueUpdate 到底做了什么？

图 3-15 是一个简化的 setState 调用栈，注意其中核心的状态判断。

![image-20210510143659716](https://raw.githubusercontent.com/silence/blog/assets/assets/20210510143659.png)

enqueueUpdate 的代码如下：

```javascript
function enqueueUpdate(component) {
  ensureInjected();
  
  // 如果不处于批量更新模式
  if (!batchingStrategy.isBatchingUpdates) {
    batchingStrategy.batchedUpdateds(enqueueUpdate, component);
    return;
  }
  // 如果处于批量更新模式，则将该组件保存在 dirtyComponents 中
  dirtyComponents.push(component);
}
```

如果 isBatchingUpdates 为 true，则对所有队列中的更新执行 batchedUpdateds 方法，否则只把当前组件（即调用了 setState 的组件）放入 dirtyComponents 数组中。例子中 4 次 setState 调用的表现之所以不同，这里逻辑判断起了关键作用。

那么 batchingStrategy 究竟做什么呢？其实它只是一个简单的对象，定义了一个 isBatchingUpdates 的布尔值，以及 batchedUpdates 方法：

```javascript
var ReactDefaultBatchingStrategy = {
  isBatchingUpdates: false,
  
  batchedUpdates: function(callback, a, b, c, d, e) {
    var alreadyBatchingUpdates = ReactDefaultBatchingStrategy.isBatchingUpdates;
    ReactDefaultBatchingStrategy.isBatchingUpdates = true;
    
    if(alreadyBatchingUpdates) {
      callback(a, b, c, d, e);
    } else {
      transaction.perform(callback, null, a, b, c, d, e);
    }
  }
}
```

值得注意的是，batchedUpdates 方法中有一个 transaction.perform 调用，这是本章后续要介绍的核心概念 --- 事务（transaction）。

##### 3.4.4 初识事务

事务源码中有一幅图，形象地解释了它的作用，如图 3-16 所示。

![image-20210510145733092](https://raw.githubusercontent.com/silence/blog/assets/assets/20210510145733.png)

事务就是将需要执行的方法使用 wrapper 封装起来，再通过事务提供的 perform 方法执行。而在 perform 之前，先执行所有 wrapper 中的 initialize 方法，执行完 perform 之后（即执行 method 方法后）再执行所有的 close 方法。一组 initialize 及 close 方法称为一个 wrapper。从图 3-16 中可以看出，事务支持多个 wrapper 叠加。

到实现上，事务提供了一个 mixin 方法供其他模块实现自己需要的事务。而要使用事务的模块，除了需要把 mixin 混入自己的事务实现中外，还要额外实现一个抽象的 getTransactionWrappers 接口。这个接口用来后去所有需要封装的前置方法（initialize）和收尾方法（close），因此它需要返回一个数组的对象，每个对象分别有 key 为 initialize 和 close 方法。

下面是一个简单使用事务的例子：

```javascript
var Transaction = require('./Transaction');

// 我们自己定义的事务
var MyTransaction = function() {
  // ...
};

Object.assign(MyTransaction.prototype, Transaction.Mixin, {
  getTransactionWrappers: function() {
    return [{
      initialize: function() {
        console.log('before method perform');
      },
      close: function() {
        console.log('after method perform');
      }
    }]
  }
})

var transaction = new MyTransaction();
var testMethod = function() {
  console.log('test');
}
transaction.perform(testMethod);

// 打印的结果如下：
// before method perform
// test 
// after method perform
```

当然，在 React 中还做了异常处理等工作，这里就不详细展开了。如果你有兴趣，可以继续翻看源码。

##### 3.4.5 解密 setState

说了这么多，事务到底是怎么导致前面所述的 setState 的各种不同表现的呢？

这里我们先要了解事务跟 setState 的不同表现有什么关系。首先，我们把 4 次 setState 简单归类，前两次属于一类，因为它们在同一次调用栈中执行，setTimeout 中的两次属于另一类，原因同上。下面我们分别看看这两类 setState 的调用栈，如图 3-17 和图 3-18 所示。

![image-20210510151843518](https://raw.githubusercontent.com/silence/blog/assets/assets/20210510151843.png)

![image-20210510151859412](https://raw.githubusercontent.com/silence/blog/assets/assets/20210510151859.png)

很明显，在 componentDidMount 中直接调用的两次 setState，其调用栈更加复杂；而 setTimeout 中调用的两次 setState，其调用栈则简单很多。**下面重点看看第一类 setState 的调用栈，有没有发现什么？没错，就是 batchedUpdates 方法，原来早在 setState 调用前，已经处于 batchedUpdates 执行的事务中了。**

**那这次 batchedUpdate 方法，又是谁调用的呢？让我们往前再追溯一层，原来是 ReactMount.js 中的 _renderNewRootComponent 方法。也就是说，整个将 React 组件渲染到 DOM 中的过程就处于一个大的事务中。**

接下来的解释就顺理成章了，**因为在 componentDidMount 中调用 setState 时，batchingStrategy 的 isBatchingUpdates 已经被设为 true，所以两次 setState 的结果并没有立即生效，而是被放进了 dirtyComponents 中。这也解释了两次打印 this.state.val 都是 0 的原因，因为新的 state 还没有被应用到组件中。**

再反观 setTimeout 中的两次 setState，因为没有前置的 batchedUpdate 调用，所以 batchingStrategy 的 isBatchingUpdates 标志位是 false，也就导致了新的 state 马上生效，没有走到 dirtyComponents 分支。也就是说，setTimeout 中第一次执行 setState 时，this.state.val 为 1，而 setState 完成后打印时 this.state.val 变成了 2。第二次的 setState 同理。

前面介绍事务时，也提到了其在 React 源码中的多处应用，像 initialize、perform、close、closeAll、notifyAll 等方法出现在调用栈中，都说明当前处于一个事务中。

### 第4章 认识 Flux 架构模式

#### 4.2 MV* 与 Flux

##### 4.2.1 MVC/MVVM

3. MVC 乍一看似乎没有特别值得诟病的地方，但是它存在一个致命的缺点，这个缺点在你的项目越来越大、逻辑越来越复杂的时候就非常明显，那就是混乱的数据流动方式，如图 4-3 所示。

   ![image-20210510202758809](https://raw.githubusercontent.com/silence/blog/assets/assets/20210510202758.png)

4. 解决方案

   如果渲染函数只有一个，统一放在 Controller 中，每次更新重渲染页面，这样的话，任何数据的更新都只用调用重渲染就行，并且数据和当前页面的状态是唯一确定的。这样又要保证数据的流动清晰，不能出现交叉分路的情况。

   然而重渲染会带来严重的性能与用户体验问题。重渲染和局部渲染各有好坏，对 MVC 来说这是一个两难的选择，无法做到鱼和熊掌兼得。

##### 4.2.2 Flux 的解决方案

与 React 相同，Flux 同样由一群 Facebook 工程师提出，它的名字是拉丁语的 Flow。Flux 的提出主要是针对现有前端 MVC 框架的局限总结出来的一套基于 dispatcher 的前端应用架构模式。如果用 MVC 的命名习惯，它应该叫 ADSV (Action Dispatcher Store View)。

那么 Flux 是如何解决 MVC 存在的问题呢？正如其名，Flux 的核心思想就是**数据和逻辑永远单向流动**。其模型图如图 4-4 所示。

![image-20210510203321786](https://raw.githubusercontent.com/silence/blog/assets/assets/20210510203321.png)

在介绍 React 的时候，我们也提到它推崇的核心也是单向数据流，Flux 中单向数据流则是在整体架构上的延伸。在 Flux 应用中，数据从 action 到 dispatcher，再到 store，最终到 view 的路线是单向不可逆的，各个角色之间不会像前端 MVC 模式中那样存在交错的连线。

然而想要做到单向数据流，并不是一件容易的事情。好在 Flux 的 dispatcher 定义了严格的规则来限定我们对数据的修改操作。同时，store 中不能暴露 setter 的设定也强化了数据修改的纯洁性，保证了 store 的数据确定应用唯一的状态。

再使用 React 作为 Flux 的 view，虽然每次 view 的渲染都是**重渲染**，但并不会影响页面的性能，因为重渲染的是 Virtual DOM，并由 PureRender 保障从重渲染到局部渲染的转换。意味着完全不用关心渲染上的性能问题，增、删、改的渲染都和初始化渲染一样快。

### 第5章 深入 Redux 应用架构

#### 5.2 Redux middleware

>"It provides a third-party extension point between dispatching an action, and the moment it reaches the reducers."

这是 Dan Abramov 对 middleware 的描述。它提供了一个分类处理 action 的机会。在 middleware 中，你可以检阅每一个流过的 action，挑选出特定类型的 action 进行相应的操作，给你一次改变 action 的机会。

##### 5.2.1 middleware 的由来

图 5-2 表达的是 Redux 中一个简单的同步数据流动场景，点击 button 后，在回调中分发一个 action，reducer 收到 action 后，更新 state 并通知 view 重新渲染。单向数据流，看着没什么问题。但是，如果需要打印每一个 action 信息来调试，就得去改 dispatch 或者 reducer 实现，使其具有打印日志的功能。又比如，点击 button 后，需要先去服务端请求数据，只有等数据返回后，才能重新渲染 view，此时我们希望 dispatch 或 reducer 拥有异步请求的功能。再比如，需要异步请求数据返回后，打印一条日志，再请求数据，再请求数据，再打印日志，再渲染。

![image-20210510211137248](https://raw.githubusercontent.com/silence/blog/assets/assets/20210510211137.png)

面对多样的业务场景，单纯地修改 dispatch 或 reducer 的代码显然不具有普适性，我们需要的是可以组合的、自由插拔的插件机制，这一点 Redux 借鉴了 Koa （它是用于构建 Web 应用的 Node.js 框架）里的 middleware 的思想，详情可查阅*附录A*。另外，Redux 中 reducer(*出自《React.js 小书》: 又名 stateChanger，一个用来改变状态的纯函数*) 更关心的是**数据的转化逻辑**，所以 middleware 就是为了增强 dispatch 而出现的。

图 5-3 展示了应用 middleware 后 Redux 处理事件的逻辑，每一个 middleware 处理一个相对独立的业务需求，通过串联不同的 middleware 实现变化多样的功能。那么，后续我们就来讨论 middleware 是怎么写的，以及 Redux 是如何让 middleware 串联起来的。

![image-20210510212136485](https://raw.githubusercontent.com/silence/blog/assets/assets/20210510212136.png)

##### 5.2.2 理解 middleware 机制

Redux 提供了 applyMiddleware 方法来加载 middleware，该方法的源码如下：

```javascript
// './compose'
// compose 函数接受一个 [f1, f2, ... fn] 的数组入参
// compose([f1, f2, ..., fn])(arg) 的执行结果就是 f1(f2(...fn-1(fn(arg)))) 这样串联执行起来的
function compose(...funcs) {
  return arg => funcs.reduceRight((composed, f) => f(composed), arg)
}

import compose from './compose';

export default function applyMiddleware(...middlewares) {
  // 这里的 next 一般传参为 Redux 里的 createStore 方法
  return (next) => (reducer, initialState) => {
    let store = next(reducer, initialState);
    let dispatch = store.dispatch;
    let chain = [];
    
    var middlewareAPI = {
      getState: store.getState,
      dispatch: (action) => dispatch(action),
    };
    
    // 给每个 middleware 传参 middlewareAPI 即 store
    // 并将它们的返回结果赋值给 chain => [f1, f2, ...fn]
    // 最简单的 f 为 (next) => (action) => { next(action) }
    chain = middlewares.map(middleware => middleware(middlewareAPI));
    // fn 执行完后变成 (action) => { store.dispatch(action) }
    // fn-1 执行时 上一步执行完成的结果成为新的 store.dispatch 函数
    // 一直执行到 第一个 f1 返回一个 (action) => { store.dispatch(action) }
    // 但此时 所有中间件的 方法 都已经执行完了
    dispatch =  compose(...chain)(store.dispatch);
    
    return {
      ...store,
      // 覆盖 store 里的 store.dispatch 方法
      dispatch,
    }
  }
}
```

applyMiddleware 的代码虽然只有二十多行，却非常精炼。

然后再来看 logger middleware 的实现：

```javascript
export default store => next => action => {
  console.log('dispatch:', action);
  next(action);
  console.log('finish:', action);
}
```

接下来，我们就分 4 步来深入解析 middleware 的运行原理。

1. **函数式编程思想设计**

middleware 的设计有点特殊，是一个层层包裹的匿名函数，这其实是函数式编程中的 currying，它是一种使用匿名单参数函数来实现多参数函数的方法。applyMiddleware 会对 logger 这个 middleware 进行层层调用，动态地将 store 和 next 参数赋值。

currying 的 middleware 结构的好处主要有以下两点。

- **易串联**：currying 函数具有**延迟执行**的特性，通过不断 currying 形成的 middleware 可以**累积参数**，再配合组合（compose）的方式，很容易形成 pipeline 来处理数据流。
- **共享 store**：在 applyMiddleware 执行的过程中，store 还是旧的，但是因为闭包的存在，applyMiddleware 完成后，所有的 middleware 内部拿到的 store 是最新且相同的。

另外，我们会发现 applyMiddleware 的结构也是一个多层 currying 的函数。借助 compose，applyMiddleware 可以用来和其他插件加强 createStore 函数：

```javascript
import { createStore, applyMiddleware, compose } from 'Redux';
import rootReducer from '../reducers';
import DevTools from '../containers/DevTools';

const finalCreateStore = compose(
  // 在开发环境中使用的 middleware
	applyMiddleware(d1, d2, d3),
  // 它会启动 Redux DevTools
  DevTools.instrument()
)(createStore);
```

2. **给 middleware 分发 store**

通过如下方式创建一个普通的 store:

```javascript
let newStore = applyMiddleware(mid1, mid2, mid3, ...)(createStore)(reducer, null);
```

上述代码执行完毕后，applyMiddleware 方法陆续获得了 3 个参数，第一个是 middlewares 数组 [mid1, mid2, mid3, ...]，第二个是 Redux 原生的 createStore，最后一个是 reducer。然后，我们可以看到 applyMiddleware 利用 createStore 和 reducer 创建了一个 store。而 store 的 getState 方法和 dispatch 方法又分别被直接和间接地赋值给 middlewareAPI 变量 `store`:

```javascript
const middlewareAPI = {
  getState: store.getState,
  dispatch: (action) => dispatch(action)
}

chain = middlewares.map(middleware => middleware(middlewareAPI))
```

然后，让每个 middleware 带着 middlewareAPI 这个参数分别执行一遍。执行完后，获得 chain 数组 [f1, f2, ..., fx, fn]，它保存的对象是第二个箭头函数返回的匿名函数。**因为是闭包，每个匿名函数都可以访问相同的 store，即 middlewareAPI**。

> 说明：middlewareAPI 中的 dispatch 为什么要用匿名函数包裹呢？
>
> 我们用 applyMiddleware 是为了改造 dispatch，所以 applyMiddleware 执行完后，dispatch 是变化了的，而 middlewareAPI 是 applyMiddleware 执行中分发到各个 middleware 的，所以必须用匿名函数包裹 dispatch，这样只要 dispatch 更新了，middlewareAPI 中的 dispatch 应用也会发生变化。

3. **组合串联 middleware**

这一层只有一行代码，却是 applyMiddleware 精华所在：

```javascript
dispatch = compose(...chain)(store.dispatch);
```

其中 compose 是函数式编程中的组合，它将 chain 中的所有匿名函数 [f1, f2, ..., fn] 组装成一个新的函数，即新的 dispatch。当新的 dispatch 执行时，[f1, f2, ..., fn]，从右到左依次执行。Redux 中的 compose 的实现是下面这样的，当然实现方式并不唯一：

```javascript
function compose(...funcs) {
  return arg => funcs.reduceRight((composed, f) => f(composed), arg)
}
```

compose(...funcs) 返回的是一个匿名函数，其中 funcs 就是 chain 数组。当调用 reduceRight 时，依次从 funcs 数组的右端取一个函数 fx 拿来执行，fx 的参数 composed 就是前依次 fx+1 执行的结果，而第一次执行的 fn (n 代表 chain 的长度) 的参数 arg 就是 store.dispatch。所以，当 compose 执行完后，我们得到的 dispatch 是这样的，假设 n = 3:

```javascript
dispatch = f1(f2(f3(store.dispatch)))
```

这时调用新 dispatch，每一个 middleware 就依次执行了。

4. **在 middleware 中调用 dispatch 会发生什么**

经过 compose 后，所有的 middleware 算是串联起来了。可是还有一个问题，在分发 store 时，我们提到过每个 middleware 都可以访问 store，即 middlewareAPI 这个变量，也可以拿到 store 的 dispatch 属性。那么，在 middleware 中**调用 store.dispatch()** 会发生什么，和**调用 next()** 有区别吗？现在我们来说明两者的不同：

```javascript
// 调用 next
const logger = store => next => action => {
  console.log('dispatch:', action);
  next(action);
  console.log('finish:', action);
}

// 调用 store.dispatch
const logger = store => next => action => {
   console.log('dispatch:', action);
   store.dispatch(action);
   console.log('finish:', action);
}
```

在分发 store 时我们解释过，middleware 中 store 的 dispatch 通过匿名函数的方式和最终 compose 结束后的新 dispatch 保持一致，所以，**在 middleware 中调用 store.dispatch() 和在其它任何地方调用的效果一样**。而在 middleware 中调用 next()，效果是进入下一个 middleware，如图 5-4 所示。

![image-20210511164728863](https://raw.githubusercontent.com/silence/blog/assets/assets/20210511164729.png)

正常情况下，如图 5-4 左图所示，**当我们分发一个 action 时，middleware 通过 next(action) 一层层处理和传递 action 直到 Redux 原生的 dispatch。如果某个 middleware 使用 store.dispatch(action) 来分发 action，就发生了如图 5-4 右图所示的情况，这相当于重新来一遍。**假如这个 middleware 一直简单粗暴地调用 store.dispatch(action)，就会形成无限循环了。那么 store.dispatch(action) 的用武之地在哪里呢？

假如我们需要发送一个异步请求到服务端获取数据，成功后弹出一个自定义的 message。这里我们用到了 Redux Thunk:

```javascript
const thunk = store => next => action => typeof action === 'function'
? action(store.dispatch, store.getState)
: next(action)
```

Redux Thunk 会判断 action 是否是函数。如果是，则执行 action，否则继续传递 action 到下一个 middleware。针对于此，我们设计了以下 action:

```javascript
const getThenShow = (dispatch, getState) => {
  const url = 'http://xxx.json';
  
  fetch(url)
  	.then((response) => {
    	dispatch({
      	type: 'SHOW_MESSAGE_FOR_ME',
        message: response.json(),
    	})
  	})
  	.catch(() => {
    	dispatch({
        type: 'FETCH_DATA_FAIL',
        message: 'error'
      })
  	})
}
```

这时候只要在应用中调用 store.dispatch(getThenShow)，Redux Thunk 就会执行 getThenShow 方法。getThenShow 会先请求数据，如果成功，分发一个显示 message 的 action；否则，分发一个请求失败的 action。而这里的 dispatch 就是通过 Redux Thunk middleware 传递进来的。

在 middleware 中使用 dispatch 的场景一般是接受到一个定向 action，这个 action 并不希望到达原生的分发 action，往往用在异步请求的需求里。在下一节中，我们会详细讨论如何在 Redux 中实现异步流。

#### 5.3 Redux 异步流

曾经前端的革新是以 Ajax 的出现为分水岭，现代应用中绝大部分页面渲染会以异步流的方式进行。我们还记得在 Flux 中，并没有定义在哪里发异步请求，那么 Redux 是如何解决这个问题的呢？

##### 5.3.1 使用 middleware 简化异步请求

在这一节中，我们通过介绍最常用的 3 个 middleware 来介绍 Redux 怎样发异步请求。

1. redux-thunk

我们试想，如果要发异步请求，在 Redux 定义中，最合适的位置是在 action creator 中实现。但我们之前了解到的 action 都是同步的情况，那么怎样让 action 支持异步情况呢？

这里引入了 redux-thunk middleware。首先我们需要知道什么是 thunk？其实在学习 Node.js 时，已经接触并熟悉 Thunk 函数了。比如：

```javascript
fs.readFile(fileName, callback);

var readFileThunk = Thunk(filename);
readFileThunk(callback);

var Thunk = function(filename) {
  return function(callback) {
    return fs.readFile(fileName, callback)
  }
}
```

**Thunk 函数实现上就是针对多参数的 currying 以实现对函数的惰性求值。任何函数，只要参数有回调函数，就能写成 Thunk 函数的形式。**

我们再来看看 **redux-thunk 的源代码**：

```javascript
function createThunkMiddleware(extraArgument) {
  return ({ dispatch, getState }) => next => action => {
    if (typeof action === 'function') {
      return action(dispatch, getState, extraArgument);
    }
    
    return next(action);
  }
}
```

我们很清楚地看到，当 action 为函数的时候，我们并没有调用 next 或 dispatch 方法，而是返回 action 的调用。这里的 action 即为一个 Thunk 函数，以达到将 dispatch 和 getState 参数传递到函数内的作用。

了解 redux-thunk 的原理后，这里我们模拟请求一个天气的异步请求。action 通常可以这么写：

```javascript
function getWeather(url, params) {
  return (dispatch, getState) => {
		fetch(url, params) .then(result => {
			dispatch({
				type: 'GET_WEATHER_SUCCESS', payload: result,
			}); 
    })
			.catch(err => { 
      	 dispatch({
					type: 'GET_WEATHER_ERROR',
					error: err, 
         });
			});
  	};
}
```

我们顺利地把同步 action 变成了异步 action。

尽管我们利用 Thunk 可以完成各种复杂的异步 action，但是对于某些复杂但是又有规律的场景，抽离出更合适的、目标更明确的 middleware 来解决会是更好的方案，而异步请求绝对是其一。

2. redux-promise

我们发现，异步请求其实都是利用 promise 来完成的，那么为什么不通过抽象 promise 来解决异步流的问题呢？

这里再引入 redux-promise middleware，然后通过源码来分析一下它是怎么做的：

```javascript
import { isFSA } from 'flux-standard-action';

function isPromise(val) {
  return val && typeof val.then === 'function';
}

export default function promiseMiddleware({ dispatch }) {
  return next => action => {
    if (!isFSA(action)) {
      return isPromise(action)
      	? action.then(dispatch)
        : next(action)
    }
    
    return isPromise(action.payload)
    	? action.payload.then(
    		result => dispatch({ ...action, payload: result }),
      	error => {
          dispatch({ ...action, payload: error, error: true })
          return Promise.reject(error);
        }
    	)
    	: next(action)
  }
}
```

redux-promise 兼容了 FSA 标准，也就是说将返回的结果保存在 payload 中。实现过程非常容易理解，即判断 action 或 action.payload 是否为 promise，如果是，就执行 then，返回的结果再发送一次 dispatch。

我们利用 ES7 的 async 和 await 语法，可以简化上述异步过程：

```javascript
const fetchData = (url, params) => fetch(url, params);

async function getWeather(url, params) {
  const result = await fetchData(url, params);
  
  if (result.error) {
    return {
      type: 'GET_WEATHER_ERROR',
      error: result.error,
    }
  }
  
  return {
    type: 'GET_WETHER_SUCCESS',
    payload: result,
  }
}
```

### 第6章 单元测试

#### 6.1 单元测试的原则

从不同的角度，可将测试划分为如下不同的种类：

1. 从人工操作还是写代码来操作的角度，可以分为手工测试和自动化测试；
2. 从**是否需要考虑系统的内部设计角度**，可以分为白盒测试和黑盒测试；
3. 从测试对象的级别，可以分为单元测试、集成测试和端到端测试；
4. 从测试验证的系统特性，又可以分为功能测试、性能测试和压力测试。

单元测试代码一般都由编写对应功能代码的开发者来编写，开发者提交的单元测试代码应该保持一定的覆盖率，而且必须永远能够运行通过。可以说，单元测试是保证代码质量的第一道防线。

既然说到单元测试，就不得不说测试驱动开发（Test Driven Development，TDD），**有的开发者对测试驱动开发奉为神器，严格实践先写单元测试测试用例后写功能代码**，而且单元测试也保证其他开发者不会因为失误破坏原有的功能；也有开发者对测试驱动开发不以为然，因为写单元测试的时间是写功能代码的好几倍，当需求发生改变的时候，除了要维护功能代码，还要维护测试代码，苦不堪言。

这里我们也不要争论测试驱动开发是否有必要，我们只需要正视一点，那就是单元测试应该是让开发者的工作更轻松更高效，而不是成为开发过程中的包袱。

每个开发团队会根据自身特点决定是否要求测试驱动开发，也可以设定恰当的单元测试覆盖阈值，不过要注意以下几点：

首先，即使单元测试覆盖率达到了 100%，也不表示程序是没有 bug 的，实现高质量的软件有多方面要求，单元测试只是手段之一，**不要对单元测试覆盖率有太过偏执的要求**。

另外，程序架构的可测试性非常重要，开发者不喜欢写单元测试代码一个很主要的原因就是发现单元测试“太难写”，比如为了写一个单元测试要写太多的模仿对象（Mock），涉及复杂的流程难以全部条件分支。

要克服单元测试“太难写”的问题，就需要架构能把程序拆分成足够小到方便测试的部分，只要每个小的部分被验证能够正确地各司其职，组合起来能够完成整体功能，那么开发者编写的单元测试就可以专注于测试各个小的部分就行，这就是更高的“可测试性”。



