[TOC]

### 前言

本文的目的在于如何使用300~400行代码实现一个基本的 Virtual DOM 算法，并且尝试尽量把 Virtual DOM 的算法思路阐述清楚。

### 对前端应用状态管理的思考

状态改变需要操作相应的 DOM 元素，那我们可以做一个东西让视图和状态进行绑定，状态变更了视图自动变更，就不用手动更新页面了。这就是后来人们想出来的 MVVM 模式，只要在模板中声明视图组件是和什么状态进行绑定的，双向绑定引擎就会在状态更新的时候自动更新视图。

MVVM 可以很好的降低我们维护状态 → 视图的复杂程度（大大减少代码中的视图更新逻辑）。但是这不是一个唯一的办法，还有一个非常直观的方法，可以大大降低视图更新的操作：一旦状态发生了变化，就用模板引擎重新渲染**整个视图**，然后用新的视图更换掉旧的视图。

这样做最大的问题就是会很慢，因为即使一个小小的状态变更都要重新构造整棵 DOM，性价比太低；而且这样做的话，`input` 和 `textarea` 会失去原有的焦点。

**但是这里要明白和记住这种做法**，因为后面你会发现，**其实 Virtual DOM 就是这么做的，只是加了一些特别的步骤来避免了整棵 DOM 树变更**。

### Virtual DOM 算法

DOM 元素非常庞大，因为标准就是这么设计的。而且操作它们的时候要小心翼翼，轻微的触碰可能就会导致页面重排，这是杀死性能的罪魁祸首。

相对于 DOM 对象，原生的 Javascript 对象处理起来更快，而且更简单。DOM 树上的结构、属性信息我们都可以很容易地用 Javascript 对象表示出来：

```javascript
var element = {
  tagName: 'ul', // 节点标签名
  props: { // DOM的属性，用一个对象存储键值对
    id: 'list'
  },
  children: [ // 该节点的子节点
    {tagName: 'li', props: {class: 'item'}, children: ["Item 1"]},
    {tagName: 'li', props: {class: 'item'}, children: ["Item 2"]},
    {tagName: 'li', props: {class: 'item'}, children: ["Item 3"]},
  ]
}
```

上面对应的 HTML 写法是：

```html
<ul id='list'>
  <li class='item'>Item 1</li>
  <li class='item'>Item 2</li>
  <li class='item'>Item 3</li>
</ul>
```

既然原来的 DOM 树的信息都可以用 Javascript 对象来表示，反过来，你就可以根据这个用 Javascript 对象表示的树结构来构建一棵真正的 DOM 树。

所谓的 Virtual DOM 算法，包括这几个步骤：

1. 用 Javascript 对象结构表示 DOM 树的结构；然后用这个树构建一个真正的 DOM 树，插到文档当中。
2. 当状态变更的时候，重新构造一棵新的对象树。然后用新的树和旧的树进行比较，记录两棵树差异。
3. 把 2 所记录的差异应用到步骤 1 所构建的真正的 DOM 树上，视图就更新了。

### 算法实现

#### 1. 步骤一：用 JS 对象模拟 DOM 树

用 Javascript 来表示一个 DOM 节点是很简单的事情，你只需要记录它的节点类型、属性，还有子节点：

Element.js

```javascript
export function Element (tagName, props, children) {
  this.tagName = tagName
  this.props = props
  this.children = children
}
```

上面的 DOM 结构就可以简单的表示为：

```javascript
import { Element } from './Element'

const ul = new Element('ul', {id: 'list'}, [
  new Element('li', {class: 'item'}, ['Item 1']),
  new Element('li', {class: 'item'}, ['Item 2']),
  new Element('li', {class: 'item'}, ['Item 3'])
])
```

现在 `ul` 只是一个 JavaScript 对象表示的 DOM 结构，页面上并没有这个结构。我们可以根据这个 `ul` 构建真正的 `<ul>` 。

```javascript
Element.prototype.render = function () {
  var el = document.createElement(this.tagName) // 根据tagName构建
  var props = this.props
  
  for (var propName in props) {
    // 遍历设置节点的 DOM 属性
    var propValue = props[propName]
    el.setAttribute(propName, propValue)
  }
  
  var children = this.children || []
  
  children.forEach(function (child) {
    var childEl = (child instanceof Element)
      // 如果子节点也是虚拟 DOM , 递归构建 DOM 节点
      ? child.render() 
      // 如果字符串，只构建文本节点
      : document.createTextNode(child) 
    el.appendChild(childEl)
  })
  
  return el
}
```

`render` 方法会根据 `tagName` 构建一个真正的 DOM 节点，然后设置这个节点的属性，最后递归地把自己的子节点也构建起来。所以只需要：

```javascript
var ulRoot = ul.render()
document.body.appendChild(ulRoot)
```

上面的 `ulRoot` 是真正的 DOM 节点，把它塞入文档中，这样 `body` 里面就有了真正的 `<ul>` 的 DOM 结构：

```javascript
<ul id='list'>
  <li class='item'>Item 1</li>
  <li class='item'>Item 2</li>
  <li class='item'>Item 3</li>
</ul>
```

#### 2 步骤二：比较两棵虚拟 DOM 树的差异

正如你所预料的，比较两棵 DOM 树的差异是 Virtual DOM 算法最核心的部分，这也是所谓的 Virtual DOM 的 diff 算法。两个树的完全的 diff 算法是一个时间复杂度为 O(n^3^) 的问题。但是在前端当中，你很少会跨越层级地移动 DOM 元素。所以 Virtual DOM 只会对同一个层级的元素进行对比：

![image-20210207192308480](https://raw.githubusercontent.com/silence/blog/assets/assets/20210207192308.png)

上面的 `div` 只会和同一层级的 `div` 对比，第二层级的只会跟第二层级对比。这样算法复杂度就可以达到 O(n)。

##### 2.1 深度优先遍历，记录差异

在实际的代码中，会对新旧两棵树进行一个深度优先的遍历，这样每个节点都会有一个唯一的标记：

![image-20210207192604486](https://raw.githubusercontent.com/silence/blog/assets/assets/20210207192604.png)

在深度优先遍历的时候，每遍历到一个节点就把该节点和新的树进行对比。如果有差异的话就记录到一个对象里面。

```javascript
// diff 函数，对比两棵树
function diff(oldTree, newTree) {
  // 当前节点的标志
  var index = 0
  // 用来记录每个节点差异的对象
  var patches = {}
  dfsWalk(oldTree, newTree, index, patches)
  return patches
}

// 对两棵树进行深度优先遍历
function dfsWalk (oldNode, newNode, index, patches) {
  // 对比 oldNode 和 newNode 的不同，记录下来
  patches[index] = [...]
                    
  diffChildren(oldNode.children, newNode.children, index, patches)
}

// 遍历子节点
function diffChildren (oldChildren, newChildren, index, patches) {
  var leftNode = null
  var currentNodeIndex = index
  oldChildren.forEach(function (child, i) {
    var newChild = newChildren[i]
    // 这里的count是指一个节点下有多少个 Element 的实例，即排除掉文本节点（文本节点不再递归），因为 foreach 是平级调用的，dfs 的递归需要知道当前节点到哪个index了，这里用闭包来包住上一个平级节点，并通过其中的属性count来计算节点标识数
    currentNodeIndex = (leftNode && left.count) // 计算节点标识数
    	? currentNodeIndex + leftNode.count + 1
      : currentNodeIndex + 1
    dfsWalk(child, newChild, currentNodeIndex, patches)
    leftNode = child
  })
}
```

例如，上面的 `div` 和新的 `div` 有差异，当前的标记是 0，那么：

```javascript
patches[0] = [{difference}, {difference}, ...] // 用数组存储新旧节点的不同
```

同理 `p` 是 `patches[1]`，`ul` 是 `patches[3]`，类推。

##### 2.2 差异类型

上面所述的节点的差异指的是什么呢？对DOM 操作可能会：

1. 替换掉原来的节点，例如把上面的 `div` 换成了 `section`
2. 移动、删除、新增子节点，例如上面 `div` 的子节点，把 `p` 和 `ul` 顺序互换
3. 修改了节点的属性
4. 对于文本节点，文本内容可能会改变。例如修改上面的文本节点 2 内容为 `Virtual DOM 2`。

所以我们定义了几种差异类型：

```javascript
var REPLACE = 0
var REORDER = 1
var PROPS = 2
var TEXT = 3
```

对于节点替换，很简单。判断新旧节点的 `tagName` 是不是一样的，如果不一样的说明需要替换掉。如 `div` 换成 `section` ，就记录下：

```javascript
patches[0] = [{
  type: REPALCE,
  node: newNode // new Element('section', props, children)
}]
```

如果给 `div` 新增了属性 `id` 为 `container`，就记录下：

```javascript
patches[0] = [{
  type: PROPS,
  props: {
    id: 'container'
  }
}]
```

如果是文本节点，如上面的文本节点2，就记录下：

```javascript
patches[2] = [{
  type: TEXT,
  context: 'virtual DOM2'
}]
```

那如果我们 `div` 的子节点重新排序呢？例如 `p, ul, div` 的顺序换成了 `div, p, ul`。这个该怎么对比？如果按照同层级进行顺序对比的话，它们都会被替换掉。如 `p` 和 `div` 的 `tagName` 不同，`p` 会被`div` 所替代。最终，三个节点都会被替换，这样 DOM 开销就非常大。而实际上是不需要替换节点，而只需要经过节点移动就可以达到，我们只需知道怎么进行移动。

这牵涉到两个列表对比算法，需要另外起一个小节来讨论。

##### 2.3 列表对比算法

 假设现在可以英文字母唯一地标识每一个子节点：

旧的节点顺序：

```
a b c d e f g h i
```

现在对节点进行了删除、插入、移动的操作。新增 `j` 节点，删除 `e` 节点，移动 `h` 节点：

新的节点顺序：

```
a b c h d f g i j
```

现在知道了新旧的顺序，求最小的插入、删除操作（移动可以看成是删除和插入操作的结合）。这个问题抽象出来其实是字符串的最小编辑距离问题（[Edition Distance](https://en.wikipedia.org/wiki/Edit_distance)），最常见的解决算法是 [Levenshtein Distance](https://en.wikipedia.org/wiki/Levenshtein_distance) ，通过动态规划求解，时间复杂度为 O (M * N)。但是我们并不需要真的达到最小的操作，我们只需要优化一些比较常见的移动情况，牺牲一定 DOM 操作，让算法时间复杂度达到线性的 （O(max(M, N))）。具体算法细节比较多，这里不累述，有兴趣可以参考[代码](https://github.com/livoras/list-diff/blob/master/lib/diff.js)。

我们能够获取到某个父节点的子节点的操作，就可以记录下来：

```javascript
patches[0] = [{
  type: REORDER,
  moves: [{remove or insert}, {remove or insert}, ...]
}]
```

但是要注意的是，因为 `tagName` 是可重复的，不能用这个来进行对比。所以需要给子节点加上唯一标识 `key` ，列表对比的时候，使用 `key` 进行对比，这样才能复用老的 DOM 树上的节点。

这样，我们就可以通过深度优先遍历两棵树，每层的节点进行对比，记录下每个节点的差异了。完整 `diff` 算法代码可见 [diff.js](https://github.com/livoras/simple-virtual-dom/blob/master/lib/diff.js)。

#### 3 步骤三：把差异应用到真正的 DOM 树上

因为步骤一所构建的 Javascript 对象树和 `render` 出来的真正的 DOM 树的信息、结构是一样的。所以我们可以对那颗 DOM 树也进行深度优先的遍历，遍历的时候从步骤二生成的 `patches` 对象中找出当前遍历的节点差异，然后进行 DOM 操作。

```javascript
function patch (node, patches) {
  var walker = { index: 0 }
  dfsWalk(node, walker, patches)
}

function dfsWalk (node, walker, patches) {
  // 从patches 拿出当前节点的差异
  var currentPatches = patches[walker.index] 
  
  var len = node.childNodes ? node.childNodes.length : 0
  // 深度遍历子节点
  for (var i = 0; i < len; i++) {
    var child = node.childNodes[i]
    walker.index ++
    dfsWalk(child, walker, patches)
  }
  
  if (currentPatches) {
    applyPatches(node, currentPatches)
  }
}
```

applyPatches，根据不同类型的差异对当前节点进行 DOM 操作：

```javascript
function applyPatches (node, currentPatches) {
  currentPatches.forEach(function (currentPatch) {
    switch (currentPatch.type) {
      case REPLACE:
        // 直接渲染一个新的节点就好
        node.parentNode.replaceChild(currentPatch.node.render(), node)
        break
      case REORDER:
        reorderChildren(node, currentPatch.moves)
        break
      case PROPS:
        setProps(node, currentPatch.props)
        break
      case TEXT:
        node.textContent = currentPatch.content
        break
      default:
        throw new Error('Unknown patch type ' + currentPatch.type)
    }
  })
}
```

完整代码可见 [patch.js](https://github.com/livoras/simple-virtual-dom/blob/master/lib/patch.js)。

### 结语

Virtual DOM 算法主要是实现上面步骤的三个函数：[element](https://github.com/livoras/simple-virtual-dom/blob/master/lib/element.js)，[diff](https://github.com/livoras/simple-virtual-dom/blob/master/lib/diff.js)，[patch](https://github.com/livoras/simple-virtual-dom/blob/master/lib/patch.js)。然后就可以实际的进行使用：

```javascript
// 1. 构建虚拟 DOM 
var tree = new Element('div', {'id': 'container'}, [
    new Element('h1', {style: 'color: blue'}, ['simple virtal dom']),
    new Element('p', ['Hello, virtual-dom']),
    new Element('ul', [el('li')])
])

// 2. 通过虚拟 DOM 构建真正的 DOM 
var root = tree.render()
document.body.appendChild(root)

// 3. 生成新的虚拟 DOM
var newTree = new Element('div', {'id': 'container'}, [
    new Element('h1', {style: 'color: red'}, ['simple virtal dom']),
    new Element('p', ['Hello, virtual-dom']),
    new Element('ul', [el('li'), el('li')])
])

// 4. 比较两棵虚拟 DOM 树的不同
var patches = diff(tree, newTree)

// 5. 在真正的 DOM 元素上应用变更
patch(root, patches)
```

当然这是非常粗糙的实践，实际中还需要处理事件监听等；生成虚拟 DOM 的时候也可以加入 JSX 语法。这些事情都做了的话，就可以构造一个简单的 ReactJS 了。

本文所实现的完整代码存放在 [Github](https://github.com/livoras/simple-virtual-dom) ，仅供学习。

### References

https://github.com/livoras/blog/issues/13

https://github.com/Matt-Esch/virtual-dom/blob/master/vtree/diff.js


