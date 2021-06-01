本文为 《Webpack 实战 - 入门、进阶与调优》的重点摘抄

[TOC]

### 第一章 Webpack 简介

#### 1.1 何为 Webpack

Webpack 是一个开源的 JavaScript 模块打包工具，其最核心的功能是**解决模块之间的依赖，把各个模块按照特定的规则和顺序组织在一起，最终合并为一个 JS 文件（有时会有多个，这里讨论的只是最基本的情况）。**这个过程就叫做**模块打包**。

你可以把 Webpack 理解为一个模块处理工厂。我们把源代码交给 Webpack，由它去进行加工、拼装处理，产出最终的资源文件，等待送往用户。

#### 1.2 为什么需要 Webpack 

说回 Webpack，既然它解决的最主要的问题是模块打包，那么为了更好地阐述 Webpack 的作用，我们必须先谈谈模块。

##### 1.2.2 JavaScript 中的模块

随着技术的发展，JavaScript 已经不仅仅用来实现简单的表单提交等功能，引入多个 script 文件到页面中逐渐成为一种常态，但我们发现这种做法有很多**缺点**。

- 需要手动维护 JavaScript 的加载顺序。**页面的多个 script 之间通常会有依赖关系，但由于这种依赖关系是隐式的，除了添加注释以外很难清晰地指明谁依赖了谁**，这样当页面中加载的文件过多时就很容易出现问题。

- **每一个 script 标签都意味着需要向服务器请求一次静态资源**，在 HTTP2 还没出现的时期，建立连接的成本是很高的，过多的请求会严重拖慢网页的渲染速度。

- 在每个 script 标签中，**顶层作用域即全局作用域，如果没有任何处理而直接在代码中进行变量或函数声明，就会造成全局作用域的污染**。

模块化则解决了上述的所有问题。

- **通过导入和导出语句我们可以清晰地看到模块间的依赖关系**，这点在后面会做详细的介绍。

- 模块可以借助工具来进行打包，**在页面中只需要加载合并后的资源文件，减少了网络开销。**

- **多个模块之间的作用域是隔离的，彼此不会有命名冲突。**

从 2009 年开始，JavaScript 社区开始对模块化进行不断的尝试，并依次出现了 AMD、CommonJS、CMD 等解决方案。但这些都只是由社区提出的，并不能算语言本身的特性。而在 2015 年，ECMAScript 6.0 (ES6) 正式定义了 JavaScript 模块标准，使这门语言在诞生了 20 年之后终于拥有了模块这一概念。

ES6 模块标准目前已经得到了大多数现代浏览器的支持，但在实际应用方面还需要等待一段时间。主要有以下几点原因：

- 无法使用 code splitting 和 tree shaking（Webpack 的两个特别重要的特性，之后的章节会介绍）。
- 大多数 npm 模块还是 CommonJS 的形式，而浏览器并不支持其语法，因此这些包没有办法直接拿来用。
- 仍然需要考虑个别浏览器及平台的兼容性问题。

那么，如何才能让我们的工程在使用模块化的同时也能正常运行在浏览器中呢？这就到了模块打包工具出场的时候了。

##### 1.2.3 模块打包工具

模块打包工具（module bundler）的任务就是解决模块间的依赖，使其打包后的结果能运行在浏览器上。它的工作方式主要分为两种：

- 将存在依赖关系的模块安装特定规则合并为**单个 JS 文件**，一次全部加载进页面中。
- 在页面初始化时加载一个入口模块，其它模块**异步地进行加载**。

##### 1.2.4 为什么选择 Webpack 

对比同类模块打包工具，Webpack 具备以下几点优势。

1. Webpack 默认**支持多种模块标准**，包括 AMD、CommonJS，以及最新的 ES6 模块，而其他工具大多只能支持一到两种。这对于一些同时使用多种模块标准的工程非常有用，Webpack 会帮我们处理好不同类型模块之间的依赖关系。
2. Webpack 有完备的代码分割（code splitting）解决方案。从字面意思去理解，它可以**分割打包后的资源，首屏只加载必要的部分，不太重要的功能放到后面动态地加载**。这对于资源体积较大的应用来说尤为重要，可以有效地减小资源体积，提升首页渲染速度。
3. Webpack 可以处理各种类型的资源。除了 JavaScript 以外，Webpack 还可以处理**样式、模板、甚至图片等**，而开发者需要做的仅仅是导入它们。比如你可以从 JavaScript 文件导入一个 CSS 或者 PNG，而这一切最终都可以由第 4 章讲到的 **loader** 来处理。
4. Webpack 拥有**庞大的社区支持**。除了 Webpack 核心库以外，还有无数开发者来为它编写周边插件和工具，绝大多数的需求你都可以直接找到已有解决方案，甚至会有多个解决方案供你挑选。

### 第二章 模块打包

#### 2.1 CommonJS

##### 2.1.1 模块

CommonJS 中规定每个文件是一个模块。将一个 JavaScript 文件直接通过 script 标签插入页面中与封装成 CommonJS 模块最大的不同在于，前者的顶层作用域是全局作用域，在进行变量及函数声明时会污染全局环境；**而后者会形成一个属于模块自身的作用域，所有的变量及函数只有自己能访问，对外是不可见的**。请看下面的例子：

```javascript
// calculator.js
var name = 'calculator.js';

// index.js
var name = 'index.js';
require('./calculator.js');
console.log(name); // => index.js
```

这里有两个文件，在 index.js 中我们通过 CommonJS 的 require 函数加载 calculator.js。运行之后控制台结果是 "index.js"，这说明 calculator.js 中的变量声明并不会影响 index.js，可见每个模块是有拥有各自的作用域的。

##### 2.1.2 导出

导出是一个模块向外暴露自身的唯一方式。在 CommonJS 中，通过 module.exports 可以导出模块中的内容，如：

```javascript
module.exports = {
  name: 'calculator',
  add: function(a, b) {
    return a + b;
  }
}
```

CommonJS 模块内部会有一个 module 对象用于存放当前模块的信息，可以理解成在每个模块的最开始定义了以下对象：

```javascript
var module = {...};
// 模块自身逻辑
module.exports = {...};             
```

module.exports 用来指定该模块要对外暴露哪些内容，在上面的代码中我们导出了一个对象，包含 name 和 add 两个属性。为了书写方便，CommonJS 也支持另一种简化的导出方式 -- 直接使用 exports。

```javascript
exports.name = 'calculator';
exports.add = function(a, b) {
  return a + b;
}
```

在实现效果上，这段代码和上面的 module.exports 没有任何不同。其内在机制是将 exports 指向了 module.exports，而 module.exports 在初始化时是一个空对象。我们可以简单地理解为，CommonJS 在每个模块的首部默认添加了以下代码：

```javascript
var module = {
  exports: {}
}
var exports = module.exports;
```

因此，为 exports.add 赋值相当于字啊 module.exports 对象上添加了一个属性。

在使用 exports 时要注意一个问题，即**不要直接给 exports 赋值，否则会导致其失效**。如：

```javascript
exports = {
  name: 'calculator'
}
```

上面代码中，由于对 exports 进行了赋值操作，使其指向了新的对象，module.exports 却仍然是原来的空对象，因此 name 属性并不会被导出。

另一个在导出时容易犯的错误是不恰当地把 module.exports 与 exports 混用。

```javascript
exports.add = function(a, b) {
  return a + b;
};
module.exports = {
  name: 'calculator'
}
```

**上面的代码先通过 exports 导出了 add 属性，然后将 module.exports 重新赋值为另外一个对象。这会导致原本拥有 add 属性的对象丢失了，最后导出的只有 name。**

另外，要注意导出语句不代表模块的末尾，在 module.exports 或 exports 后面的代码依旧会照常执行。比如下面的 console 会在控制台上打出 "end"：

```javascript
module.exports = {
  name: 'calculator'
}
console.log('end')
```

在实际使用中，为了提高可读性，不建议采取上面的写法，而是应该将 module.exports 及 exports 语句放在模块的末尾。

##### 2.1.3 导入

在 CommonJS 中使用 require 进行模块导入。如：

```javascript
// calculator.js
module.exports = {
  add: function(a, b) {
    return a + b;
  }
}
// index.js
const calculator = require('./calculator.js');
const sum = calculator.add(2, 3);
console.log(sum); // 5
```

我们在 index.js 中导入了 calculator 模块，并调用了它的 add 函数。

当我们 require 一个模块时会有两种情况：

- require 的模块是第一次被加载。这时会首先执行该模块，然后导出内容。
- require 的模块**曾被加载过。这时该模块的代码不会再次执行，而是直接导出上次执行后得到的结果**。

我们前面提到，模块会有一个 module 对象用来存放其信息，这个对象中有一个属性 loaded 用于记录该模块是否被加载过。它的值默认为 false，当模块第一次被加载和执行过后会置为 true，后面再次加载时检查到 module.loaded 为 true，则不会再次执行模块代码。

有时我们加载一个模块，不需要获取其导出的内容，只是想要通过执行它而产生某种作用，比如把它的接口挂在全局对象上，此时直接使用 require 即可。

```javascript
require('./task.js');
```

另外，require 函数可以**接收表达式**，借助这个特性我们可以动态地指定模块加载路径。

```javascript
const moduleNames = ['foo.js', 'bar.js'];
moduleNames.forEach(name => {
  require('./' + name);
})
```

#### 2.2 ES6 Module

##### 2.2.1 模块

请看下面的例子，我们将前面的 calculator.js 和 index.js 使用 ES6 的方式进行了改写。

```javascript
// calculator.js
export default {
  name: 'calculator',
  add: function(a, b) {
    return a + b;
  }
}

// index.js
import calculator from './calculator.js';
const sum = calculator.add(2, 3);
console.log(sum);
```

ES6 Module 也是将每个文件作为一个模块，每个模块拥有自身的作用域，不同是导入、导出语句。**import 和 export 也作为保留关键字在 ES6 版本中加入了进来**（CommonJS 中的 module 并不属于关键字）。

ES6 Module 会自动采用严格模式，这在 ES5（ECMAScript 5.0）中是一个可选项。以前我们可以通过选择是否在文件开始时加上 "use strict" 来控制严格模式，**在 ES6 Module 中不管开头是否有 "use strict"，都会采用严格模式。如果将原本是 CommonJS 的模块或任何未开启严格模式的代码改写为 ES6 Module 要注意这点。**

##### 2.2.3 导入

导入变量的效果相当于在当前作用域下声明了这些变量，并且不可对其进行更改，也就是所有导入的变量都是只读的。

#### 2.3 CommonJS 与 ES6 Module 的区别

##### 2.3.1 动态与静态

CommonJS 与 ES6 Module 最本质的区别在于前者对模块依赖的解决是 "动态的"，而后者是 "静态的"。在这里 "动态" 的含义是，**模块依赖关系的建立发生在代码运行阶段；而 "静态" 则是模块依赖关系的建立发送在代码编译阶段。**

让我们先看一个 CommonJS 的例子：

```javascript
// calculator.js
module.exports = { name: 'calculator' };
// index.js
const name = require('./calculator.js').name;
```

在上面上面介绍 CommonJS 的部分时我们提到过，当模块 A 加载模块 B 时（在上面的例子中是 index.js 加载 calculator.js），会执行 B 中的代码，并将其 module.exports 对象作为 require 函数的返回值进行返回。**并且 require 的模块路径可以动态指定，支持传入一个表达式，我们甚至可以通过 if 语句判断是否加载某个模块。因此，在 CommonJS 模块被执行前，并没有办法确定明确的依赖关系，模块的导入、导出发生在代码的运行阶段。**

同样的例子，让我们再对比看下 ES6 Module 的写法：

```javascript
// calculator.js
export const name = 'calculator';
// index.js
import { name } from './calculator.js';
```

ES6 Module 的导入、导出语句都是声明式的，它不支持导入的路径是一个表达式，并且导入、导出语句必须位于模块的顶层作用域（比如不能放在 if 语句中）。因此我们说，ES6 Module 是一种静态的模块结构，在 ES6 代码的编译阶段就可以分析出模块的依赖关系。它相比于 CommonJS 来说具备以下几点优势：

- 死代码检测和排除。我们可以用静态分析工具检测出那些模块没有被调用过。比如，在引入工具类库时，工程中往往只用到了其中一部分组件或接口，但有可能会将其代码完整地加载进来。未被调用到的模块代码永远不会被执行，也就成为了死代码。**通过静态分析可以在打包时去掉这些未曾使用过的模块，以减少打包资源体积。**
- 模块变量类型检查。JavaScript 属于动态类型语音，不会在代码执行前检查类型错误（比如对一个字符串类型的值进行函数调用）。**ES6 Module 的静态模块结构有助于确保模块之间传递的值或接口类型是正确的。**
- 编译器优化。在 CommonJS 等动态模块系统中，无论采用哪种方式，本质上导入的都是一个对象，**而 ES6 Module 支持直接导入变量，减少了引用层级，程序效率更高。**

##### 2.3.2 值拷贝与动态映射

在导入一个模块时，对于 CommonJS 来说获取的是一份**导出值的拷贝**；而在 ES6 Module 中则是**值的动态映射**，并且这个映射是**只读**的。

上面的话直接理解起来可能比较困难，首先让我们来看一个例子，了解一下什么是 CommonJS 中的值拷贝。

```javascript
// calculator.js
var count = 0;
module.exports = {
  count: count,
  add: function(a, b) {
    count += 1;
    return a + b;
  }
}

// index.js 
var count = require('./calculator.js').count;
var add = require('./calculator.js').add;

console.log(count);
// => 0 (这里的 count 是对 calculator.js 中 count 值的拷贝)
add(2, 3);
console.log(count);
// => 0 (calculator.js 中变量值的改变不会对这里的拷贝值造成影响)

count += 1;
console.log(count);
// => 1 (拷贝的值可以更改)
```

index.js 中的 count 是对 calculator.js 中 count 的一份**值拷贝**，因此在调用 add 函数时，虽然更改了原本 calculator.js 中 count 的值，但是并不会对 index.js 中导入时创建的副本造成影响。

另一方面，在 CommonJS 中允许对导入的值进行更改。我们可以在 index.js 更改 count 和 add，将其**赋予新值**。同样，由于是值的拷贝，**这些操作不会影响 calculator.js 本身**。

下面我们使用 ES6 Module 将上面的例子进行改写：

```javascript
// calculator.js
let count = 0;
const add = function(a, b) {
  count += 1;
  return a + b;
};
export { count, add };

// index.js
import { count, add } from './calculator.js';

console.log(count);
// => 0 (对 calculator.js 中 count 值的映射)
add(2, 3);
console.log(count);
// => 1 (实时反映 caculator.js 中 count 值的变化)

// count += 1; // 不可更改，会抛出 SyntaxError: "count" is read-only
```

我们不可以对 ES6 Module 导入的变量进行更改，可以将这种映射关系理解为一面镜子，从镜子里，我们可以实时观察到原有的事物，但是并不可以操纵镜子中的影像。

##### 2.3.3 循环依赖

循环依赖是指 模块A 依赖于 模块B ，同时 模块B 依赖于 模块A 。比如下面这个例子：

```javascript
// a.js
import { foo } from './b.js';
foo();

// b.js
import { bar } from './a.js''
bar();
```

如何处理循环依赖是开发者必须要面对的问题。我们首先看一下在 CommonJS 中循环依赖的例子。

```javascript
// foo.js
const bar = require('./bar.js');
console.log('value of bar:', bar);
module.exports = "This is foo.js";

// bar.js
const foo = require('./foo.js');
console.log('value of foo:', foo);
module.exports = "This is bar.js";

// index.js
require('./foo.js');
```

理想状态下我们希望二者都能导入正确的值，并在控制台输出。

```javascript
value of foo: This is foo.js
value of bar: This is bar.js
```

而当我们运行上面的代码时，实际输出却是：

```javascript
value of foo: {}
value of bar: This is bar.js
```

为什么 foo 的值会是一个空对象呢？让我们从头梳理一下代码的实际执行顺序。

1. index.js 导入了 foo.js，此时开始执行 foo.js 中的代码。
2. foo.js 的第一句导入了 bar.js，这时 foo.js 不会继续向下执行，而是进入了 bar.js 内部。
3. 在 bar.js 中又对 foo.js 进行了 require，这里产生了循环依赖。**需要注意的是，执行权并不会再交回 foo.js，而是直接取其导出值，也就是 module.exports。但由于 foo.js 未执行完毕，导出值在这时为默认的空对象**，因此当 bar.js 执行到打印语句时，我们看到控制台中的 value of foo 就是一个空对象。
4. bar.js 执行完毕，将执行权交回 foo.js。
5. foo.js 从 require 语句继续向下执行，在控制台打印出 value of bar（这个值是正确的），整个流程结束。

由上面可以看出，尽管循环依赖的模块均被执行了，但模块导入的值并不是我们想要的。因此在 CommonJS 中，若遇到循环依赖我们没有办法得到预想中的结果。

我们再从 Webpack 的实现角度来看，将上面的例子打包后，bundle 中有这样一段代码非常重要：

```javascript
// The require function
function __webpack_require__(moduleId) {
  if (installedModules[moduleId]) {
    return installedModules[moduleId].exports;
  }
  // Create a new module (and put it into the cache)
  var module = installedModules[moduleId] = {
    i: moduleId,
    1: false,
    exports: {}
  }
}
```

当 index.js 引用了 foo.js 之后，相当于执行了这个 \__webpack_require__ 函数，初始化了一个 module 对象并放入了 installedModules 中。当 bar.js 再次引用 foo.js 时，**又执行了该函数，但这次是直接从 installedModules 里面取值，此时它的 module.exports 是一个空对象。**这就解释了上面在第 3 步看到的对象。

接下来让我们使用 ES6 Module 的方式重写上面的例子。

```javascript
// foo.js 
import bar from './bar.js';
console.log('value of bar:', bar);
export default "This is foo.js";

// bar.js
import foo from './foo.js';
console.log('value of foo:', foo);
export default "This is bar.js";

// index.js
import foo from './foo.js';
```

执行结果如下：

```javascript
value of foo: undefined
value of bar: This is bar.js
```

上面我们谈到，在导入一个模块时，CommonJS 获取到的是值的拷贝，ES6 Module 则是动态映射，那么我们能否利用 ES6 Module 的特性使其支持循环依赖呢？请看下面的例子：

```javascript
// index.js
import foo from './foo.js';
foo('index.js');

// foo.js
import bar from './bar.js';
function foo(invoker) {
  console.log(invoker + 'invokes foo.js');
  bar('foo.js');
}
export default foo;

// bar.js
import foo from './foo.js';
let invoked = false;
function bar(invoker) {
  if (!invoked) {
    invoked = true;
    console.log(invoker + 'invokers bar.js');
    foo('bar.js');
  }
}
export default bar;
```

上面代码的执行结果如下：

```javascript
index.js invokes foo.js
foo.js invokes bar.js
bar.js invokes foo.js
```

可以看到，foo.js 和 bar.js 这一对循环依赖的模块均获取到了正确的导出值。

#### 2.4 加载其他类型模块

##### 2.4.1 非模块化文件

假如我们引入的非模块化文件是以隐式全局变量声明的方式暴露其接口的，则会发生问题。如：

```javascript
// 通过在顶层作用域声明变量的方式暴露接口
var calculator = {
  // ...
}
```

由于 Webpack 在打包时会为**每一个文件包装一层函数作用域来避免全局污染**，上面的代码将无法把 calculator 对象挂在全局，**因此这种以隐式全局变量声明需要格外注意**。

##### 2.4.2 AMD

AMD 是英文 Asynchronous Module Definition（异步模块定义）的缩写，它是由 JavaScript 社区提出的**专注于支持浏览器端模块化的标准**。从名字就可以看出它与 CommonJS 和 ES6 Module 最大的区别在于它加载模块的方式是异步的。下面的例子展示了如何定义一个 AMD 模块。

```javascript
define('getSum', ['calculator'], function(math) {
  return function (a, b) {
    console.log('sum: ' + calculator.add(a, b));
  }
})
```

在 AMD 中使用 define 函数来定义模块，它可以接受 3 个参数。第 1 个参数是当前模块的 id，相当于模块名；第 2 个参数是当前模块的依赖，比如上面我们定义的 getSum 模块需要引入 calculator 模块作为依赖，第 3 个参数用来描述模块的导出值，**可以是函数或对象。如果是函数则导出的是函数的返回值；如果是对象则直接导出对象本身**。

和 CommonJS 类似，AMD 也使用 require 函数来加载模块，只不过采用异步的形式。

```javascript
require(['getSum'], function(getSum) {
  getSum(2, 3)
})
```

require 的第一个参数指定了加载的模块，第二个参数是当加载完成后执行的回调函数。

通过 AMD 这种形式定义模块的好处在于其模块加载是非阻塞性的，当执行到 require 函数时并不会停下来去执行被加载的模块，而是继续执行 require 后面的代码，这使得模块加载操作并不会阻塞浏览器。

尽管 AMD 的设计理念很好，但与同步加载的模块标准相比其语法要更加冗长。另外其异步加载的方式并不如同步显得清晰，并且容易造成回调地域（callback hell）。在目前的实际应用中已经用得越来越少，大多数开发者还是会选择 CommonJS 或 ES6 Module 的形式。

##### 2.4.3 UMD

我们已经介绍了很多的模块形式，CommonJS、ES6 Module、AMD 及非模块化文件，在很多时候工程中会用到其中两种形式甚至更多。有时对于一个库或者框架的开发者来说，如果面向的使用群体足够庞大，就需要考虑支持各种模块形式。

严格来说，UMD 并不能说是一种模块标准，不如说它是**一组模块形式的集合**更准确。UMD 的全称是 Universal Module Definition，也就是通用模块标准，它的目标是使一个模块能运行在各种环境下，不论是 CommonJS、AMD，还是非模块化的环境（当时 ES6 Module 还未被提出）。

请看下面的例子：

```javascript
// calculator.js
(function(global, main) {
  // 根据当前环境采取不同的导出方式
  if (typeof define === 'function' && define.amd) {
    // AMD
    define(...);
  } else if(typeof exports === 'object') {
      // CommonJS
      module.exports = ...;
    } else {
      // 非模块化环境
      global.add = ...;
    }
})(this.function() {
   // 定义模块主体
   return {...}
   })
```

可以看出，UMD 其实就是根据当前全局对象中的值判断目前处于哪种模块环境。当前环境是 AMD，就以 AMD 的形式导出；当前环境是 CommonJS，就以 CommonJS 的形式导出。

需要注意的问题是，UMD 模块一般都是最先判断 AMD 环境，也就是全局下是否有 define 函数，而通过 AMD 定义的模块是无法使用 CommonJS 或 ES6 Module 的形式正确引入的。在 Webpack 中，由于它同时支持 AMD 及 CommonJS，也许工程中的所有模块都是 CommonJS，而 UMD 标准却发现当前有 AMD 环境，并使用了 AMD 方式导出，这会使得模块导入时出错。当需要这样做时，我们可以更改 UMD 模块中判断的顺序，使其以 CommonJS 的形式导出即可。

##### 2.4.4 加载 npm 模块

每一个 npm 模块都有一个入口。当我们加载一个模块时，实际上就是加载该模块的入口文件。这个入口被维护在模块内部 package.json 文件的 main 字段中。

比如对于前面的 lodash 模块来说，它的 package.json 内容如下：

```json
// ./node_modules/underscore/package.json
{
  "name": "lodash",
  // ......
  "main": "lodash.js"
}
```

当加载该模块时，实际上加载的是 node_modules/lodash/lodash.js。

除了直接加载模块以外，我们也可以通过 <package_name>/<path\> 的形式单独加载模块内部的某个 JS 文件。如：

```javascript
import all from 'lodash/fp/all.js';
console.log('all', all);
```

这样，Webpack 最终只会打包 node_modules/lodash/fp/all.js 这个文件，而不会打包全部的 lodash 库，通过这种方式可以减小打包资源的体积。

#### 2.5 模块打包原理

面对工程中成百上千个模块，Webpack 究竟是如何将它们有序地组织在一起，并按照我们预想的顺序运行在浏览器上的呢？下面我们将从原理上进行深究。

还是使用前面 calculator 的例子。

```javascript
// index.js
const calculator = require('./calculator.js');
const sum = calculator.add(2, 3);
console.log('sum', sum);

// calculator.js
module.exports = {
  add: function (a, b) {
    return a + b;
  }
}
```

上面的代码经过 Webpack 打包后将会成为如下的形式（为了易读性这里只展示代码的大体结构）：

```javascript
// 立即执行匿名函数
(function(modules) {
  // 模块缓存
  var installedModules = {};
  // 实现 require
  function __webpack_require__(moduleId) {
    // ...
  }
  // 执行入口模块的加载
  return __webpack_require__(__webpack_require__.s = 0);
})({
  // modules: 以 key-value 的形式储存所有被打包的模块
  0: function(module, exports, __webpack_require__) {
    // 打包入口
    module.exports = __webpack_require__("3qiv");
  },
  "3qiv": function(module, exports, __webpack_require__) {
    // index.js 内容
  },
  "jkzz": function(module, exports) {
    // calculator.js 内容
  }
})
```

这是一个最简单的 Webpack 打包结果（bundle），但已经可以清晰地展示出它是如何将具有依赖关系的模块串联在一起的。上面的 bundle 分为以下几个部分：

- 最外层立即执行匿名函数。它用来包裹整个 bundle，并构成自身的作用域。
- installedModules 对象。**每个模块只在第一次被加载的时候执行，之后其导出值就被存储到这个对象里面，当再次被加载的时候直接从这里取值，而不会重新执行。**
- __webpack_require\_\_ 函数。对模块加载的实现，**在浏览器中可以通过调用 \_\_webpack_require\_\_(module_id) 来完成模块导入。**
- modules 对象。工程中所有产生了依赖关系的模块都会以 key-value 的形式放在这里。key 可以理解为一个模块的 id，由数字或者一个很短的 hash 字符串构成；value 则是由一个匿名函数包裹的模块实体，匿名函数的参数则赋予了**每个模块导出和导入的能力**。

接下来让我们看看一个 bundle 是如何在浏览器中执行的。

1. 在最外层的匿名函数中会初始化浏览器执行环境，包括定义 installedModules 对象、\_\_webpack_require\_\_ 函数等，为模块的加载和执行做一些准备工作。
2. 加载入口模块。**每个 bundle 都有且只有一个入口模块**，在上面的示例中，index.js 是入口模块，在浏览器中会从它开始执行。
3. 执行模块代码。如果执行到了 module.exports 则记录下模块的导出值；如果中间遇到 require 函数（准确地说是 \_\_webpack_require\_\_），则会暂时交出执行权，进入 \_\_webpack_require\_\_ 函数体内进行加载其它模块的逻辑。
4. 在 \_\_webpack_require\_\_ 中会判断即将加载的模块是否存在于 installedModules 中。如果存在则直接取值，否则回到第 3 步，执行该模块的代码来获取导出值。
5. 所有依赖的模块都已执行完毕，最后执行权又回到入口模块。当入口模块的代码执行到结尾，也就意味着整个 bundle 运行结束。

不难看出，第 3 步和第 4 步是一个递归的过程。Webpack 为每个模块创造了一个可以导出和导入模块的环境，但本质上并没有修改代码的执行逻辑，因此代码执行的顺序与模块加载的顺序是完全一致的，这就是 Webpack 模块打包的奥秘。

#### 2.6 本章小结

本章我们介绍了 JavaScript 模块，包括主流模块标准的定义，以及在 Webpack 中是如何进行模块打包的。

CommonJS 和 ES6 Module 是目前使用较为广泛的模块标准。它们的主要区别在于前者建立**模块依赖**关系是在**运行时**，后者是在**编译时**；在模块导入方面，CommonJS 导入的是**值拷贝**，ES6 Module 导入的是**只读的变量映射**；ES6 Module 通过其静态特性可以进行**编译过程中的优化**，并且具备**处理循环依赖的能力**。

下一章我们将介绍资源的输入输出，包括资源的处理流程、Webpack 中 chunk、bundle 等概念，以及如何针对不同的场景配置打包的入口和出口。

### 第三章 资源输入输出

#### 3.1 资源处理流程

在一切流程的最开始，我们需要指定一个或多个入口（entry），也就是告诉 Webpack 具体从源码目录下的哪个文件开始打包。如果把工程中的各个模块的依赖关系当作一棵树，那么入口就是这棵依赖树的根，如图 3-1 所示。

![image-20210529201402251](https://raw.githubusercontent.com/silence/blog/assets/assets/20210529201402.png)

这些存在依赖关系的模块会在打包时被封装为一个 chunk。本书后面的部分会经常提及 chunk，这里先让我们解释一下这个概念。

chunk 字面的意思是代码块，在 Webpack 中可以理解成**被抽象和包装过后的一些模块**。它就像一个装着很多文件的文件袋，里面的文件就是各个模块，Webpack 在外面加了一层包裹，从而形成了 chunk。根据具体配置不同，一个工程打包时可能会产生一个或多个 chunk。

从上面我们已经了解到，Webpack 会从入口文件开始检索，并将具有依赖关系的模块生成一棵树，最终得到一个 chunk。由这个 chunk 得到的打包产物我们一般称之为 bundle。entry、chunk、bundle 的关系如图 3-2 所示。

![image-20210529202037512](https://raw.githubusercontent.com/silence/blog/assets/assets/20210529202037.png)

在工程中可以定义多个入口，每一个入口都会产生一个结果资源。比如我们工程中有两个入口 src/index.js 和 src/lib.js，在一般情形下会打包生成 dist/index.js 和 dist/lib.js，因此可以说，entry 和 bundle 存在着对应关系，如图 3-3 所示。

![image-20210529202312051](https://raw.githubusercontent.com/silence/blog/assets/assets/20210529202312.png)

在一些特殊情况下，一个入口也可能产生多个 chunk 并最终生成多个 bundle，本书后面的章节会对这些情况进行更深入的介绍。

#### 3.2 配置资源入口

##### 3.2.3 实例

2. 提取 vendor

vendor 的意思是 "供应商"，在 Webpack 中 vendor 一般指的是工程所使用的库、框架等第三方模块集中打包而产生的 bundle。

由于 vendor 仅仅包含第三方模块，这部分不会经常变动，因此可以有效地利用客户端缓存，在用户后续请求页面时会加快整体的渲染速度。

#### 3.4 本章小结

本章我们介绍了资源的输入输出流程，以及相关的配置项 context、entry、output。

在配置打包入口时，context 相当于路径前缀，entry 是入口文件路径。单入口的 chunk name 不可更改，多入口的话则必须为每一个 chunk 指定 chunk name。

当第三方依赖较多时，我们可以用提取 vendor 的方法将这些模块打包到一个单独的 bundle 中，以更有效地利用客户端缓存，加快页面渲染速度。

path 和 publicPath 的区别在于 path 指定的是资源的输出位置，而 publicPath 指定的是间接资源的请求位置。

下一章我们会介绍 Webpack “一切皆模块” 的思想，以及预处理器 loader 的使用。可以说，是 loader 赋予了 Webpack 无尽的想象力。

### 第四章 预处理器

#### 4.1 一切皆模块

所有这些静态资源都是模块，我们可以像加载一个 JS 文件一样去加载它们，如在 index.js 中加载 style.css:

```javascript
// index.js
import './style.css';
```

对于刚开始接触 Webpack 的人来说，可能会认为这个特性很神奇，甚至会觉得不解：从 JS 中加载 CSS 文件具有怎样的意义呢？从结果来看，其实和之前并没有什么差别，这个 style.css 可以被打包并生成输出资源目录下，对 index.js 文件也不会产生实质性的影响。这句引用的实际意义是**描述了 JS 文件与 CSS 文件之间的依赖关系。**

![image-20210529205409226](https://raw.githubusercontent.com/silence/blog/assets/assets/20210529205409.png)

我们以关系图的方式来表示一个日历组件引用资源文件在使用 Webpack 前后依赖关系图的对比。

可以看到在使用了 Webpack 后，组件的 JS 和 SCSS 作为一个整体被页面引入进来，这样就更加清晰地描述了资源之间的关系。

我们知道，模块是具有**高内聚**性及**可复用**性的结构，通过 Webpack “一切皆模块” 的思想，我们可以将模块的这些特性应用到每一种静态资源上，从而设计和实现出更加健壮的系统。

#### 4.2 loader 概述

每个 loader 本质上都是一个函数。在 Webpack 4 之前，函数的输入和输出都必须为字符串；在 Webpack 4 之后，loader 也同时支持抽象语法树（AST）的传递，通过这种方法来减少重复的代码解析。用公式表达 loader 本质则为以下形式：

```javascript
output = loader(input)
```

这里的 input 可能是工程源文件的字符串，也可能是上一个 loader 转化后的结果，包括转化后的结果（也是字符串类型）、source map，以及 AST 对象；output 同样包含这几种信息，转化后的文件字符串、source map、以及 AST。如果这是最后一个 loader，结果将直接被送到 Webpack 进行后续处理，否则将作为下一个 loader 的输入向后传递。

loader 可以是链式的。如在工程中编译 SCSS 时，我们可能需要如下 loader:

```javascript
Style 标签 = style-loader(css-loader(sass-loader(SCSS)))
```

为了更好阐述 loader 是如何工作的，下面来看一下 loader 的源码结构：

```javascript
module.exports = function loader(content, map, meta) {
  var callback = this.async();
  var result = handler(content, map ,meta);
  callback(
  	null, // or error
    result.content, // 转换后的内容
    result.map, // 转换后的 source-map
    result.meta // 转换后的 AST
  )
}
```

从上面代码可以看出，loader 本身就是一个函数，在该函数中对接收到的内容进行转换，然后返回转换后的结果（可能包含 source map 和 AST 对象）。

#### 4.3 loader 的配置

##### 4.3.4 更多配置

1. exclude 与 include 

   exclude 和 include 同时存在时，exclude 的优先级更高。

2. resource 与 issuer

   看一个例子：

   ```javascript
   // index.js
   import './style.css';
   ```

   在 Webpack 中，我们认为加载模块是 resource，而加载者是 issuer。如上面的例子中，resource 为 `/path/of/app/style.css`，issuer 是 `/path/of/app/index.js`。如果想要对 issuer 加载者也增加条件限制，则要额外写一些配置，如果我们只想让 /src/pages 目录下的 JS 可以引用 CSS，应该如何设置呢？请看下面的例子：

   ```javascript
   rules: [
     {
       test: /\.css$/,
       use: ['style-loader', 'css-loader'],
       exclude: /node_modules/,
       issuer: {
         test: /\.js$/,
         include: '/src/pages/',
       }
     }
   ]
   ```

   只有 `src/pages/` 目录下面的 JS 文件引用 CSS 文件，这条规则才会生效；如果不是 JS 文件引用的 CSS（比如 JSX 文件），或者是别的目录的 JS 文件引用 CSS，则规则不生效。

3. enforce

   enforce 的值为 “pre” ，代表它将在所有正常 loader 之前执行，这样可以保证其检测的代码不是被其他 loader 更改过的。类似的，如果某一个 loader 是需要在所有 loader 之后执行的，我们也可以指定其 enforce 为 “post”。

#### 4.4 常用 loader 介绍

##### 4.4.1 babel-loader

在配置 babel-loader 时有一些需要注意的地方。请看下面的例子：

```javascript
rules: [
  {
    test: /\.js$/,
    exclude: /node_modules/,
    use: {
      loader: 'babel-loader',
      options: {
        cacheDirectory: true,
        presets: [[
          'env', {
            modules: false
          }
        ]]
      }
    }
  }
]
```

- 对于 babel-loader 本身我们添加了 cacheDirectory 配置项，它会启用缓存机制，**在重复打包未改变过的模块时防止二次编译**，同样也会加快打包速度。
- **由于 @babel/preset-env 会将 ES6 Module 转化为 CommonJS 的形式，这会导致 Webpack 中的 tree-shaking 特性失效**（关于 tree-shaking 会在第 8 章详细介绍）。将 @babel/preset-env 的 modules 配置项设置为 false 会禁用模块语句的转化，而将 ES6 Module 的语法交给 Webpack 本身处理。

##### 4.4.2 ts-loader 

ts-loader 与 babel-loader 的性质类似，它是用于连接 Webpack 与 Typescript 的模块。

##### 4.4.3 html-loader 

html-loader 用于将 HTML 文件转化为字符串并进行格式化，这使得我们可以把一个 HTML 片段通过 JS 加载进来。

##### 4.4.4 handlebars-loader

handlebars-loader 用于处理 handlebars 模板

##### 4.4.5 file-loader 

file-loader 用于打包文件类型的资源，并返回其 publicPath。

##### 4.4.6 url-loader 

url-loader 与 file-loader 作用类似，唯一的不同在于**用户可以设置一个文件大小的阈值**，当大于该阈值时与 file-loader 一样返回 publicPath，而小于该阈值时则返回文件 base64 形式编码。

##### 4.4.7 vue-loader

vue-loader 用于处理 vue 组件

### 第五章 样式处理

先对 plugins 做一个简要介绍。plugins 用于接收一个插件数组，我们可以使用 Webpack 内部提供的一些插件，也可以加载外部插件。Webpack 为插件提供了各种 API，使其**可以在打包的各个环节中添加一些额外任务**。

##### 5.1.3 mini-css-extract-plugin

说到 mini-css-extract-plugin 的特性，最重要的就是它支持按需加载 CSS，以前在使用 extract-text-webpack-plugin 的时候我们是做不到这一点的。举个例子，a.js 通过 import() 函数异步加载了 b.js，b.js 里面加载了 style.css，那么 style.css 最终只能被同步加载（通过 HTML 的 link 标签）。但是现在 mini-css-extract-plugin 会单独打包出一个 0.css （假设使用默认配置），这个 CSS 文件将由 a.js 通过动态插入 link 标签的方式加载。

按需加载指的是，通过动态插入 script 标签的方式加载脚本 （动态插入 link 标签的方式加载样式。

### 第七章 生产环境配置

##### 7.4.1 source map 原理

在使用 source map 之前，让我们先介绍一下它的工作原理。Webpack 对于工程源代码的每一步处理都有可能会改变代码的位置、结构，甚至是所处文件，因此每一步都需要生成对应的 source map。若我们启用了 devtool 配置项，source map 就会跟随源代码一步步被传递，直到生成最后的 map 文件。这个文件默认就是打包后的文件名加上 .map，如：bundle.js.map。

在生成 mapping 文件的同时，bundle 文件中会追加上一句注释来标识 map 文件的位置。如：

```javascript
// bundle.js
(function() {
  // bundle 的内容
})();
// # sourceMappingURL = bundle.js.map
```

当我们打开了浏览器的开发者工具时，map 文件会同时被加载，这时浏览器会使用它来对打包后的 bundle 文件进行解析，分析出源代码的目录结构和内容。

map 文件有时会很大，但是不用担心，只要不打开开发者工具，浏览器是不会加载这些文件的，因此对于普通用户来说并没有影响。

### 第八章 打包优化

首先重述一条软件工程领域的经验 --- **不要过早优化**，在项目的初期不要看到任何优化点就拿来加到项目中，这样不但增加了复杂度，优化的效果也不会太理想。一般是当项目发展到一定规模后，性能问题随之而来，这时再去分析然后对症下药，才有可能达到理想的优化效果。

#### 8.1 HappyPack

##### 8.1.1 工作原理

在打包过程中有一项非常耗时的工作，就是使用 loader 将各种资源进行转译处理。最常见的包括使用 babel-loader 转译 ES6+ 语法和 ts-loader 转译 TypeScript。

这里的问题在于 Webpack 是单线程的，假设一个模块依赖于几个其他模块，Webpack 必须对这些模块逐个进行转译。虽然这些转译任务彼此之间没有任何依赖关系，却必须串行地执行。HappyPack 恰恰以此为切入点，它的核心特性是可以开启多个线程，并行地对不同模块进行转译，这样就可以充分利用本地的计算资源来提升打包速度。

#### 8.2 缩小打包作用域

从宏观角度来看，提升性能的方法无非两种：**增加资源**或者**缩小范围**。增加资源就是指**使用更多 CPU 和内存，用更多的计算能力来缩短执行任务的时间**；缩小范围则是针对任务本身，比如**去掉冗余的流程，尽量不做重复性的工作等**。前面我们说的 HappyPack 属于增加资源，那么接下来我们再谈谈如何缩小范围。

#### 8.3 动态链接库与 DllPlugin

动态链接库是早期 Windows 系统由于受限于当时计算机内存空间较小的问题而出现的一种内存优化方法。当一段相同的子程序被多个程序调用时，为了减少内存消耗，可以将这段子程序存储为一个可执行文件，当被多个程序调用时只在内存中生成和使用同一个实例。

DllPlugin 借鉴了动态链接库的这种思路，对于第三方模块或者一些不常变化的模块，可以将它们预先编译和打包，然后在项目实际构建过程中直接取用即可。

DllPlugin 和 Code Splitting 有点类似，都可以用来提取公共模块，但本质上有一些区别。Code splitting 的思路是设置一些特定的规则并在打包的过程中根据这些规则提取模块；DllPlugin 则是将 vendor 完全拆出来，**有自己的一整套 Webpack 配置并独立打包，在实际工程构建时就不用再对它进行任何处理，直接取用即可**。

#### 8.4 tree shaking

在第 2 章我们介绍过，ES6 Module 依赖关系的构建是代码编译时而非运行时。基于这项特性 Webpack 提供了 tree shaking 功能，它可以在打包过程中帮助我们**检测工程中没有被引用过的模块**，这部分代码将永远无法被执行到，因此也被称为“死代码”。

##### 8.4.1 ES6 Module

tree shaking 只能对 ES6 Module 生效。

##### 8.4.2 使用 Webpack 进行依赖关系构建

如果我们在工程中使用了 babel-loader，那么一定要**通过配置来禁用它的模块依赖解析**。因为如果由 babel-loader 来做依赖解析，Webpack 接收到的就都是转化过的 CommonJS 形式的模块，无法进行 tree-shaking。

### 第九章 开发环境调优

#### 9.2 模块热替换

##### 9.2.2 HMR 原理

在本地开发环境下，浏览器是客户端，webpack-dev-server (WDS) 相当于是我们的服务端。HMR 的核心就是客户端从服务端拉取更新后的资源（准确地说，HMR 拉取的部署整个资源文件，而是 chunk diff，即 chunk 需要更新的部分。）

第 1 步就是浏览器什么时候去拉取这些更新。这需要 WDS 对本地源文件进行监听。实际上 WDS 与浏览器之间维护了一个 websocket，当本地资源发生变化时 WDS 会向浏览器推送更新时间，并带上这次构建的 hash，让客户端与上一次资源进行比对。通过 hash 的比对可以防止冗余更新的出现。因为很多时候源文件的更改并不一定代表构建结果的更改（如添加了一个文件末尾空行等）。websocket 发送的事件列表如图 9-4 所示。

![image-20210530153500967](https://raw.githubusercontent.com/silence/blog/assets/assets/20210530153501.png)

有了恰当的拉取资源的时机，下一步就是要知道拉取什么。如果构建结果不一样，现在客户端已经知道新的构建结果和当前的有了差别，就会向 WDS 发起一个请求来获取更新文件的列表，即哪些模块有了改动。通常这个请求的名字为 [hash].hot-update.json。图 9-5、图 9-6 分别展示了该接口的请求地址和返回值。

![image-20210530153808595](https://raw.githubusercontent.com/silence/blog/assets/assets/20210530153808.png)

![image-20210530153820959](https://raw.githubusercontent.com/silence/blog/assets/assets/20210530153821.png)

该返回结果告诉客户端，需要更新的 chunk 为 main，版本为（构建 hash）e388ea0f0e...。这样客户端就可以再借助这些信息继续向 WDS 获取该 chunk 的增量更新。图 9-7、图 9-8 展示了一个获取增量更新接口的例子。

![image-20210530154116852](https://raw.githubusercontent.com/silence/blog/assets/assets/20210530154116.png)

![image-20210530154140711](https://raw.githubusercontent.com/silence/blog/assets/assets/20210530154140.png)

现在客户端已经获取到了 chunk 的更新，到这里又遇到了一个非常重要的问题，即客户端获取到这些增量更新之后如何处理？哪些状态需要保留，哪些又需要更新？这个就不属于 Webpack 的工作了，但是它提供了相关的 API（如前面我们提到的 module.hot.accept），开发者可以使用这些 API 针对自身场景进行处理。像 react-hot-loader 和 vue-loader 也都是借助这些 API 来实现的 HMR。

### 第十章

#### 10.1 Rollup

如果用 Webpack 与 Rollup 进行比较的话，那么 Webpack 的优势在于它更全面，基于“一切皆模块”的思想而衍生出丰富的 loader 和 plugin 可以满足各种使用场景；而 Rollup 则更像一把手术刀，它**更专注于 JavaScript 的打包**。如果当前的项目需求仅仅是打包 JavaScript，比如一个 JavaScript 库，那么 Rollup 很多时候会是我们的第一选择。