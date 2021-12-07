[TOC]

原文来源：[使用 EventEmitter2（观察者模式）构建前端应用（一）](https://github.com/livoras/blog/issues/6)

### 1. 前言

接之前的**如何使用事件巧妙巧妙地进行模块的解耦**。特意写这篇博客详细说一下。

本文主要介绍：

1. 观察者模式在前端中的体现形式
2. 事件在组件化的前端架构中的应用

#### 1.1 观察者模式在前端中的表现形式 -- 事件机制

在构建前端应用的时候免不了要和事件打交道，有些同学可能觉得事件不就是鼠标点击执行特定的函数之类的吗？

此 “事件” 非彼 “事件”。这里的“事件”，实际上是指“**观察者模式（Observer Pattern）**”在前端的一种呈现方式。所谓观察者模式可以类比博客“**订阅/推送**”，你通过 RSS 订阅了某个博客，那么这个博客有新的博文就会自动推送你；当你退订阅这个博客，那么就不会再推送给你。

用 Javascript 代码可以怎么表示这个一个场景？

```javascript
// 假设已有一个 Blog 类实现 subscribe、publish、unsubscribe 方法
var blog = new Blog();

var readerFunc1 = function (blogContent) {
  console.log(blogContent + " will be shown here.");
};
var readerFunc2 = function (blogContent) {
  console.log(blogContent + " will be shown here, too.");
};

// 读者1订阅博客
blog.subscribe(readerFunc1);
// 读者2订阅博客
blog.subscribe(readerFunc2);

// 发布博客内容，上面的两个读者的函数都会被调用
blog.publish("This is blog content.");
// 读者1取消订阅
blog.unsubscribe(readerFunc1);
// readerFunc1 函数不再调用，readerFunc2 继续调用
blog.publish("This is another blog content.");
```

可以把上面的“新文章”看成是一个事件，“订阅文章”则是“**监听**”这个事件，“发布新文章”则是“**触发**”这个事件，“取消订阅文章”就是“**取消监听**”“新文章”这个事件。

假如“**监听**”用 `on` 来表示，“**触发**”用 `emit` 来表示，“**取消监听**”用 `off` 来表示，那么上面的代码可以重新表示为：

```javascript
// 假设已有一个 Blog 类实现 on、emit、off 方法
var blog = new Blog();

var readerFunc1 = function (blogContent) {
  console.log(blogContent + " will be shown here.");
};

var readerFunc2 = function (blogContent) {
  console.log(blogContent + " will be shown here, too.");
};

// 读者1监听事件
blog.on("new post", readerFunc1);
// 读者2监听事件
blog.on("new post", readerFunc2);

// 发布博客内容，触发事件，上面的两个读者的函数都会被调用
blog.emit("new post", "This is blog content.");
// 读者1取消监听事件
blog.off("new post", readerFunc1);
// readerFunc1 函数不再调用，readerFunc2 继续调用
blog.emit("new post", "This is another blog content.");
```

这就是前端中观察者模式的一种具体的表现，使用 `on` 来监听特定的事件， `emit` 触发特定的事件，`off` 取消监听特定的事件。再举一个场景“小猫听到小狗叫就会跑”：

```javascript
var dog = new Dog();
var cat = new Cat();

dog.on("park", function () {
  cat.run();
});

dog.emit("park");
```

巧妙利用观察者模式可以让前端应用开发耦合性变得更加低，开发效率更高。可能说“变得更有趣”会显得有点不专业，但确实会变得有趣。

#### 1.2 EventEmitter2

上面可能比较疑惑的一个点就是，`on`、`emit`、`off` 函数该怎么实现？

如果要自己实现一遍也不很复杂：每个“事件名”对应的就是一个函数数组，每次 `on` 某个事件的时候就是把函数压到对应的函数数组当中；每次 `emit` 的时候相当于把事件名对应的函数数组遍历一遍进行调用；每次 `off` 的时候把目标函数从数组当中剔除。[这里](https://github.com/livoras/stereojs/blob/master/src/events.js)有个简单的实现，有兴趣的可以了解一下。

重复发明轮子的事情就不要做了，其实现成有很多 Javascript 的事件库，本文主要使用这个库来展开对观察者模式在前端应用中的讨论。但实际上，**你可以使用任何自己构建的或者第三方的事件库来实践本文所提及的应用方式。**

### 2. EventEmitter2 简介

EventEmitter 本来是 Nodejs 中自带的 `events` 模块中的一个类，可见 [Node.js 文档](https://nodejs.org/api/events.html)。可供开发者自定义事件，后来有人把它重新实现了一遍，优化了实现方式，提高了性能，新增了一些方便的 API，这就是 [EventEmitter2](https://github.com/asyncly/EventEmitter2)。当然，后来陆续出现了 [EventEmitter3](https://www.npmjs.com/package/eventemitter3)，[EventEmitter4](https://www.npmjs.com/package/eventemitter4)。可见没有女朋友的程序员也是比较无聊地只好重复发明和优化轮子。

EventEmitter 2 可以供浏览器、或者 Node.js 使用。安装过程和 API 就不在这里累述，参照官方文档即可。使用 Browserify 或者 Node.js 可以非常方便地引用 EventEmitter2，只需要 require 即可。示例：

```javascript
var EventEmitter2 = require("eventemitter2").EventEmitter2;
var emitter = new EventEmitter2();

emitter.on("Hello World", function () {
  console.log("SomeBody said: Hello world.");
});

emitter.emit("Hello World");
// 输出 SomeBody said: Hello world.
```

#### 2.1 EventEmitter2 作为父类给子类提供事件机制

但在实际应用中，很少单纯 EventEmitter 直接实例化来使用。比较多的应用场景是，为某些已有的类添加事件的功能。如上面的第一章中的 “小猫听到小狗叫就会跑” 的例子，`Cat` 和 `Dog` 类本身就有自己的类属性、方法，需要 的是为已有的 Cat、Dog 添加事件功能。这里就需要让 EventEmitter 作为其他类的父类进行继承。

```javascript
var EventEmitter2 = require("eventemitter2").EventEmitter2;

// Cat子类继承父类构造字
function Cat() {
  EventEmitter2.apply(this);
  // Cat 构造子，属性初始化等
}

// 原型继承
Cat.prototype = Object.create(EventEmitter2.prototype);
Cat.prototype.constructor = Cat;

// Cat类方法
Cat.prototype.run = function () {
  console.log("This cat is running...");
};

var cat = new Cat();
console.assert(typeof cat.on === "function"); // => true
console.assert(typeof cat.run === "function"); // => true
```

很棒是吧，这样就可以既有 EventEmitter2 的原型方法，也可以定义 Cat 自身的方法。

**这一点都不棒！**每次定义一个类都要重新写一堆啰嗦的东西，下面做个继承的改进：构建一个函数，只需要传入已经定义好的类就可以在不影响类原有功能的情况下，让其拥有 EventEmitter2 的功能：

```javascript
// Function `eventify`: Making a class get power of EventEmitter2!
// @copyright: Livoras
// @date: 2015/03/27
// All rights reserve!

function eventify(kclass) {
  if (kclass.prototype instanceof EventEmitter2) {
    console.warn("Class has been eventified!");
    return kclass;
  }

  function Tempt() {
    kclass.apply(this, arguments);
    EventEmitter2.call(this);
  }

  function Tempt2() {}
  // Tempt2 继承 EventEmitter2
  Tempt2.prototype = Object.create(EventEmitter2.prototype);
  Tempt2.prototype.constructor = EventEmitter2;

  var temptProp = Object.create(Tempt2.prototype);

  var kclassProp = klass.prototype;

  for (var attr in klassProp) {
    temptProp[attr] = klassProp[attr];
  }

  Tempt.prototype = temptProp;
  Tempt.prototype.contructor = klass;

  return Tempt;
}
```

**上面的代码的实现的原理后续画时间整理一下**，在这里只需要知道，有了 eventify 就可以很方便的给类添加 EventEmitter2 的功能，使用方法如下：

```javascript
// Dog 类的构造函数和原型方法定义
function Dog(name) {
  this.name = name;
}

Dog.prototype.park = function () {
  console.log(this.name + " parking....");
};

// 使 Dog 具有 EventEmitter2 功能
Dog = eventify(Dog);
var dog = new Dog("Jerry");

dog.on("somebody is coming", function () {
  dog.park();
});

dog.emit("somebody is coming");
// 输出 Jerry is parking....
```

如上面的代码，现在没有必要为 Dog 类重新书写继承类代码，只需要按正常的方式定义好 Dog 类，然后传入 eventify 函数即可使 Dog 获取 EventEmitter2 的功能。本文接下来的讨论会持续使用 `eventify` 函数。

### 3. EventEmitter2 在组件化的前端架构中的应用

#### 3.1 组件化的前端架构

当一个前端应用足够复杂的时候，往往需要对应用进行“**组件化**”。所谓组件化，就是把一个大的应用拆分成多个小的应用。每个“应用”具有自己独特的**结构和内容、样式和业务逻辑**，这些小的应用称为“组件”（Component）。组件的复用性一般很强，是 DRY 原则的应用典范，多个组件的嵌套、组合，构建成了一个完成而复杂的应用。

举个例子：博客的评论功能组件：

![block](https://raw.githubusercontent.com/silence/blog/assets/assets/20210412212816.jpg)

这个评论组件的功能大概如此：

- 可显示多条评论（comment）；
- 每条评论多条有自己的回复；
- 评论或者回复都会显示有用户头像
- 鼠标放到用户头像上会显示该用户的信息（类似微博的功能）。

这里可以把这个功能分好几个组件：

1. 整体评论功能作为一个组件：commentsBox
2. commentsBox 有子组件 (child component) comment 负责显示用户的评论
3. 每个 comment 组件有子组件 replay 负责显示用户对评论的回复
4. commentsBox 有子组件 user-info-card 负责显示用户的信息。

组件这样的关系可以用树的结构来表示：

![img](https://raw.githubusercontent.com/silence/blog/assets/assets/20210412213452.png)

这里要注意的是组件之间的关系一般有两种：嵌套和组合。嵌套，如，每个 commentBox 有 comment 和 user-info-card，comment 和 user-info-card 是嵌套在 commentBox 当中的，所以这两个组件和 commentBox 之间都是嵌套的关系；组合，comment 和 user-info-card 都是作为 commentBox 的子组件存在，他们两个互为兄弟，是组合的关系。**处理组件之间的嵌套和组合关系是架构层面需要解决的最重要的问题之一**，不在本文讨论范围内，故不累述。但接下来我们讨论的“**组件之间以事件的形式进行消息传递**”和这些组件之间的关系密切相关。

当开始按照上面的设计进行组件化的时候，我们首先要做的是为每个组件构建一个超类，所有的组件都应该继承这个超类：

Component.js

```javascript
eventify = require("./eventify.js");

// Component 构造函数
function Component(parent) {
  this.$el = $("...");
  this.parent = parent;
}

// Component 原型方法
Component.prototype.init = function () {};

module.exports = eventify(Component);
```

这里为了方便起见，Component 基本上面内容都没有，几乎只是一个“空”的类，而它通过 eventify 函数获得了“超能力”，所以继承 Component 的类同样就有事件的功能。

注意 Component 构造函数，每个 Component 在实例化的时候应该传入一个它所属的**父组件**的实例 `parent` ，接下来会看到，组件之间的消息通信可以通过这个实例来完成。而 `$el` 可以看作是该组件所负责的 HTML 元素。

#### 3.2 父子、兄弟组件之间的消息传递

现在把注意力放在 commentsBox、comment、user-info-card 三个组件上，暂且忽略 reply。

目前要实现的功能是：鼠标放到 comment 组件的用户头像上，就会显示用户信息。要把这个功能完成大概是这个一个事件流程：comment 组件监听用户鼠标放在头像上的交互事件，然后通过 `this.parent` 向父组件（commentsBox）传递该事件（`this.parent` 就是 commentsBox），**commentBox 获取到该事件以后触发一个事件给 user-info-card，user-info-card 可以通过 `this.parent` 监听到该事件，显示用户信息**。

```javascript
// comment-component.js
// 从 Component 类中继承获得 Comment 类
// ...

// 原型方法
Comment.prototype.init = function () {
  var that = this;
  this.$el.find("div.avatar").on("mouseover", function () {
    // 这里的 that.parent 相当于父组件 CommentsBox，在 Comment 组件被实例化的时候传入
    that.parent.emit("comment:user-mouse-on-avatar", this.userId);
  });
};
```

上述代码为当用户把鼠标放到用户头像的时候触发一个事件 `comment:user-mouse-on-avatar`，这里需要注意的是，通过 `组件名：事件名` 给这样的事件命名方式可以区分事件的来源组件或目标组件，是一种比较好的编程习惯。

```javascript
// comments-box-component.js
// 从 Component 类中继承获得 CommentsBox 类
// ...

// 原型方法
CommentsBox.prototype.init = function () {
  var that = this;
  // 这里接受到来自 Comment 组件的事件
  this.on("comment:user-mouse-on-avatar", function (userId) {
    // 把这个事件传递给 user-info-card 组件
    that.emit("user-info-card:show-user-info", userId);
  });
};
```

上述代码中 commentsBox 获取到来自 comment 组件的 `comment:user-mouse-on-avatar` 事件，由于 user-info-card 组件也同时拥有 commentsBox 的实例，所以 commentsBox 可以通过触发自身的事件 `user-info-card:show-user-info` 来给 user-info-card 组件传递事件。再一次注意这里的事件名， `user-info-card:` 前缀说明这个事件是由 user-info-card 组件所接收的。

```javascript
// user-info-card-component.js
// 从 Component 类中继承获得 UserInfoCard 类
// ...

// 原型方法
UserInfoCard.prototype.init = function () {
  var that = this;
  this.parent.on("user-info-card:show-user-info", function (userId) {
    // 通过 ajax 获取用户数据
    $.ajax({
      user: "/users" + userId,
      method: "GET",
    }).success(function (data) {
      // 渲染用户信息
      that.render(data);
      // 显示信息
      that.show();
    });
  });
};
```

上述代码中，user-info-card 组件通过 `this.parent` 获取到来自其父组件 （也就是 commentsBox ）的事件 `user-info-card:show-user-info`，并且得到所传入的用户 id；然后通过 ajax 向服务器发送用户 id，请求用户数据渲染页面数据然后显示。

这样，消息就通过事件机制从 comment 到达了它的父组件 commentsBox，然后通过 commentsBox 到达它的兄弟组件 user-info-card。完成了一个父子组件之间、兄弟之间的消息传递过程：

![img](https://raw.githubusercontent.com/silence/blog/assets/assets/20210413114727.png)

按照这种消息传递方式的事件有四种类型：

1. `this.parent.emit`，触发父组件的事件，由父组件监听，相当于告知父组件自己所发生的事情。
2. `this.parent.on`，监听父组件的事件，由父组件触发，相当于接收处理来自父组件的指令。
3. `this.emit`，触发自己的事件，由子组件监听，相当于向某个子组件发送命令。
4. `this.on`，监听自己的事件，由子组件触发，相当于接受处理来自子组件的事件。

每个组件只要 hold 住一个其父组件实例，就可以完成：

1. 和父组件直接进行消息通信
2. 通过父组件和自己的兄弟组件间接进行消息通信

两个功能。

#### 3.3 使用事件总线（eventbus）进行跨组件消息传递

现在可以把注意力放到 reply 组件上，reply 作为 comment 的子组件，负责显示这条评论下的回复。类似地，它有回复者的用户头像，鼠标放上去以后也可以显示用户的信息。

user-info-card 是 commentsBox 的子组件，reply 是 comment 的子组件；user-info-card 和 reply 既不是父子也不是兄弟节点关系，reply 无法按照上面的方式比较直接地把事件传递给它; reply 的鼠标放到头像上的事件需要先传递给其父组件 comment，然后经过 comment 传递给 commentsBox，最后通过 commentsBox 传递给 user-info-card 组件。如下：

![img](https://raw.githubusercontent.com/silence/blog/assets/assets/20210413115831.png)

看起来好像比较麻烦，reply 离它根组件 commentsBox 高度为二，嵌套了两层。假设 reply 嵌套了很多层，那么事件的传递就类似浏览器的事件冒泡一样，需要先冒泡到根节点 commentsBox，再由根节点把事件发送给 user-info-card。

如果要真的这样写会带来相当大的维护成本，当组件之间的交互方式更改了甚至只是单单修改了事件名，中间层的负责事件转发的都需要把代码重新修改。**而且，这些负责转发的组件需要维护和自己业务逻辑并不相关的逻辑，违反单一职责原则**。

解决这个问题的方式就是：**提供一个组件之间共享的事件对象 eventbus，可以负责跨组件之间的事件传递**。所有的组件都可以从这个总线上触发事件，也可以从这个总线上监听事件。

![img](https://raw.githubusercontent.com/silence/blog/assets/assets/20210413120415.png)

common/eventbus.js

```javascript
var EventEmitter2 = require("eventemitter2").EventEmitter2;

// eventbus 是一个简单的 EventEmitter2 对象
module.exports = new EventEmitter2();
```

那么 reply 组件和 user-info-card 就可以通过 eventbus 进行之间的信息交换，在 reply 组件中：

```javascript
// reply.js
// 从 Component 类中继承获得 Reply 类
// ...

eventbus = require("../common/eventbus.js");

// 原型方法
Reply.prototype.init = function () {
  var that = this;
  this.$el.find("div.avatar").on("mouseover", function () {
    // 触发 eventbus 上的事件 user-info-card:show-user-info
    eventbus.emit("user-info-card:show-user-info", that.userId);
  });
};
```

在 user-info-card 组件当中：

```javascript
// user-info-card-component.js
// 从 Component 类中继承获得 UserInfoCard 类
// ...

eventbus = require("../common/eventbus.js");

// 原型方法
UserInfoCard.prototype.init = function () {
  var that = this;

  // 原来的逻辑不变
  this.parent.on("user-info-card:show-user-info", getUserInfoAndShow);

  // 新增获取 eventbus 的事件
  eventbus.on("user-info-card:show-user-info", getUserInfoAndShow);

  function getUserInfoAndShow(userId) {
    // 通过 ajax 获取用户数据
    $.ajax({
      user: "/users/" + userId,
      method: "GET",
    }).success(function (data) {
      // 渲染用户信息
      that.render(data);
      // 显示信息
      that.show();
    });
  }
};
```

这样 user-info-card 和就跨越了组件嵌套组合的关系，直接进行组件之间的信息事件的交互。

![img](https://raw.githubusercontent.com/silence/blog/assets/assets/20210413151258.png)

#### 3.4 问题就来了

那么问题就来了：

1. 既然 eventbus 这么方便，为什么不所有组件都往 eventbus 上发送事件，这样不就不需要组件的事件转发，方便多了吗？
2. 什么时候使用 eventbus 进行事件传递，什么时候通过组件转发事件？

如果所有的组件都往 eventbus 上 post 事件，那么就会带来 eventbus 上事件的维护的困难；我们可以类比一下 JavaScript 里面的全局变量，假如所有函数都不自己维护局部变量，而都使用全局变量会带来什么问题？想想都觉得可怕。既然这个事件交互只是在局部组件之间交互的，那么就尽量不要把它 post 到 eventbus，eventbus 上的事件应该尽量少，越少越好。

那什么时候使用 eventbus 上的事件？这里给出一个原则：当组件嵌套了三层以上的时候，带来局部事件转发维护困难的时候，就可以考虑祭出 eventbus 。而在实际当中很少会出现三层事件传播这种情况，可以保持 eventbus 事件的简洁。（按照这个原则上面的 reply 是不需要使用 eventbus 的，但是为了阐述 eventbus 而使用，这点要注意。）
