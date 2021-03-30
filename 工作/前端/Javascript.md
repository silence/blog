[TOC]

### 内置类型

1. 基本类型：`null` `undefined` `number` `string` `symbol` `boolean`  **bigint(ES7)**

2. Js的数字类型是浮点类型的，没有整型。基于IEEE 754标准实现，使用中会遇到bug，
   - 比如`NaN`也属于`number`类型，并且`NaN`不等于自身。
   
   - IEEE 754标准的双精度版本是64位的。
   
   - 二进制数转换成十进制数
     
     - $$
       \begin{split}
       {} & (110.11)_2 = 1 \times 2^2 + 1 \times2^1 + 0 \times 2^0 + 1 \times 2 ^ {-1} + 1 \times 2 ^{-2} {}\\
        & = 4 + 2 + 0 + 0.5 + 0.25 = 6.75
     
       \end{split}
     $$
       
     
   - 十进制整数转成二进制数
   
     - ![image-20210107095420622](https://raw.githubusercontent.com/silence/blog/assets/assets/20210107231957.png)
   
   - 十进制小数转成二进制数
   
     - ![image-20210107095701789](https://raw.githubusercontent.com/silence/blog/assets/assets/20210107232023.png)
   
   - 十进制 整数和小数 转换成二进制数
   
     - ![image-20210107100258194](https://raw.githubusercontent.com/silence/blog/assets/assets/20210107232150.png)
     - 0.7 = 0.101100110...无限循环
   
   - IEEE754标准 ： 浮点数在内存中的存储方式
   
     - 单精度类型 32位 float、双精度类型 64位 double
     - float 1 符号位 8指数位 23尾数位
     - double 1 符号位 11指数位 53尾数位
     - ![image-20210107232423732](https://raw.githubusercontent.com/silence/blog/assets/assets/20210107232423.png)
     - 以单精度为例，8指数位能表示255个数，但是指数位有正有负、所以它表示[-127, 128] 并取偏差+127来存储，双精度的为11位所以bias为1023
     - 尾数位在科学计数法后省略掉首部的1，以后面的来存储
     - 32位单精度浮点数的正常数取值范围示意图，因为指数位只能取[1, 254] （全为0表示极小数，这时尾数的隐藏数也是0）（全为1表示特殊数，表示正负无穷大（**尾数位全为0**）或者NaN（**尾数位不全为0**） not a number），减去bias后就是[-126, 127]，正常情况下尾数位的取值范围是[1,2)，所以32位float的正常数值取值范围如下
     - ![image-20210107234320325](https://raw.githubusercontent.com/silence/blog/assets/assets/20210107234320.png)
     - 非正常数值即趋近于0的这块的数值即指数位全为0，尾数位的隐藏位为0，**实际指数 = 1-bias = 1-127 = -126**，注意这里是规定，和规格数的计算方法不太一样，因为非规格数的隐藏数为0，可以表示±[0,1)^(-126)即(-1 * 2 ^ -126, 1* 2 ^ -126)之间的数，刚好和规格数的取值范围互补，得到(-2 * 2 ^127^, 2* 2 ^127^)范围。
     - 把非规格数与规格数放在一起审视，ieee754浮点数表盘上的蓝点依旧是越靠近0越密集，越靠近∞越稀疏。
     - 数学中的小数是连续的，而ieee754标准中的小数是离散的。
     - 比如键入0.2于ieee754标准的计算机中，计算机实际存储的0.20000000298023223876953125,因为0.2不能用二进制表示出来，所以计算机只能近似。
     - ieee754能存储到很大的数，但精度很低，数值越大精度越低，后面的数基本处于不可用的状态，32位的浮点数只适用存储±16777216之间的整数，最多只能存储7位有效数字，整数+小数大于7位就不能精确表示了
   
   - 回到正题，为什么js里0.1 + 0.2 ！= 0.3 ，通过上面的描述已经很清楚了。键入的0.1、0.2实际在二进制中都是无限循环小数。计算机相加后得到的二进制数转换成十进制就是0.30000000000000004
   
3. Typeof 对于基本类型，除了null都可以显示正确的类型，`typeof [] // object`, `typeof {} // object` ,`typeof console.log // 'function'`，typeof 对于对象除了函数都会显示`object`，对于`null`会显示为`object`，是一个留到现在的bug，之前000开头的表示为object，null全为0，就被错误判断为了object

4. 条件判断时，除了`undefined`、`null`、`false`、`''`、`NaN` 、`±0`其它所有都会被转换成`true`，包括所有对象

5. `Symbol.toPrimitibve`该方法在转基本类型时调用优先级最高

6. == 操作符会有各种莫名其妙的类型转换

   ```javascript
   1 + '1' // '11'
   2 * '2' // 4
   [1, 2] + [2, 1] // '1,22,1'
   +'2' // 2
   + 'b' // NaN
   ```

### 原型

1. ![image-20210108133649768](https://raw.githubusercontent.com/silence/blog/assets/assets/20210108133649.png)
2. \_\_proto\__属性实际上是Object.prototype.\_\_proto__实现的，Object.prototype是一个对象也拥有\_\_proto\_\_属性。
3. 注意特殊 `Function.__proto__ === Function.prototype`

### new

1. 新生成了一个对象
2. 链接到原型
3. 绑定this
4. 返回新对象

```typescript
function create() {
  var res = {}
  var protoConstructor = [].shift.call(arguments)
  res.__proto__ = protoConstructor.prototype
  var result = protoConstructor.apply(res, arguments)
  return typeof result === 'object' ? result: res
}

// etc
var Person = function (name){
  this.name = name || "leere"
  this.age = 22
}

create(Person, "john")
```

在运算符优先级里 new ... (...) 的优先级大于 new ...

```javascript
new Person.getName() // === new (Person.getName())
new Person().getName() // === (new Person()).getName() 直观上也很好理解
```

### instanceof

判断某个对象是否是某个构造方法的实例

```javascript
function instanceof(left, right){
  var prototype = right.prototype
  var left = left.__proto__
  // 递归查找，顺着原型链往上
  while(true) {
    if(left === null) return false
    if(left === prototype) return true
    left = left.__proto__
  }
}
```

### this

```javascript
function foo() {
  console.log(this.a)
}
var a = 1
foo() // 1

var obj = {
  a: 2,
  foo: foo
}
obj.foo() // 2

/** 这两种情况 this 只依赖于调用函数前的对象，优先级是第二个情况大于第一个情况 */

// 以下情况是优先级最高的， this 只会绑定在 c 上，不会被任何方式修改 this 指向
var c = new foo()
// 结合之前的实现new来看， return protoConstructor.apply(res, arguments), this 绑定在res上
c.a = 3
console.log(c.a)

// 利用call, apply, bind 改变this,这个优先级仅次于 new
```

**箭头函数其实是没有 this 的，箭头函数中的this只取决于他外面的第一个不是箭头函数的函数的this。**

### Javascript 词法作用域和动态作用域

1. 作用域是指程序源代码中定义变量的区域

2. Javascript采用词法作用域，也就是静态作用域

3. 静态作用域即，函数的作用域在函数定义的时候就决定了

4. 动态作用域即：函数的作用域是在函数调用的时候才决定的

5. ```javascript
   var value = 1;
   
   function foo() {
     // 静态作用域所以没有找到value就会从书写的位置查找上面一层的代码，所以结果就是1
       console.log(value);
   }
   
   function bar() {
     // 若是动态作用域，则结果是2
       var value = 2;
       foo();
   }
   
   bar();
   
   // 结果是 1
   ```

### Javascript 执行上下文栈

1. Javascript 的可执行代码的类型就三种，全局代码、函数代码、eval代码

2. 当执行到一个函数的时候，就会进行准备工作，这里的“准备工作”，就是“执行上下文 `execution context`”

3. Javascript 引擎创建了执行上下文栈（`Execution context stack, ECS`）来管理执行上下文

   - 定义执行上下文栈是一个数组

     ```javascript
     let ECStack = [];
     ```

   - Javascript 要解释执行代码的时候，最先遇到的就是全局代码，所以初始化的时候首先就会向执行上下文栈压入一个全局执行上下文，用 `globalContext`表示它，并且只有应用程序结束的时候，ECStack才会被清空，所以程序结束前，ECStack最底部永远有个globalContext。

     ```javascript
     ECStack = [globalContext]
     ```

   - 假设需要执行下面这段代码

     ```javascript
     function fun3() {
         console.log('fun3')
     }
     
     function fun2() {
         fun3();
     }
     
     function fun1() {
         fun2();
     }
     
     fun1();
     ```

   - 执行一个函数，就会创建一个执行上下文，并且压入执行上下文栈，当函数执行完毕时，就会将函数的执行上下文从栈中弹出。

     ```javascript
     // fun1();
     ECStack.push(<fun1> functionContext);
     
     // fun1中竟然调用了fun2...
     ECStack.push(<fun2> functionContext);
     
     // 擦，fun2还调用了fun3
     ECStack.push(<fun3> functionContext);
     
     // fun3 执行完毕
     ECStack.pop();
     
     // fun2 执行完毕
     ECStack.pop();
     
     // fun1 执行完毕
     ECStack.pop();
     
     // ECStack底层永远有个globalContext
     ```
     
    - 思考题
      ```javascript
   var scope = "global scope";
       function checkscope(){
         var scope = "local scope";
         function f(){
             return scope;
         }
         return f();
       }
       checkscope();
      ```
      ```javascript
   var scope = "global scope";
       function checkscope(){
         var scope = "local scope";
         function f(){
             return scope;
         }
         return f;
       }
       checkscope()();
      ```
   
      两端代码执行结果一样，但是这两段代码的执行上下文栈的变化不一样
      
      第一段：
   
      ```javascript
      ECStack.push(<checkscope> functionContext);
      ECStack.push(<f> functionContext);
      ECStack.pop();
      ECStack.pop();
      ```
   
      第二段是：
   
      ```javascript
      ECStack.push(<checkscope> functionContext);
      ECStack.pop();
      ECStack.push(<f> functionContext);
      ECStack.pop();
      ```

### 变量对象

1. 对于每个执行上下文，都有三个重要属性：
   - 变量对象 （Variable object, VO）
   - 作用域链（Scope chain）
   - this
   
2. 变量对象是与执行上下文相关的数据作用域，存储了在上下文中定义的变量和函数声明。

3. 全局上下文中的变量对象就是全局对象。

4. 在函数上下文中，我们用活动对象（activation object, AO）来表示变量对象。只有到当进入一个执行上下文中，这个执行上下文的变量对象才会被激活，所以才叫 `activation object`，只有被激活的变量对象，也就是AO上的各种属性才能被访问。

5. 活动对象是在进入函数上下文时刻被创建的，它通过函数的arguments属性初始化。arguemnts属性值是Arguments对象。

6. 变量对象包括：

   - 函数的所有形参（如果是函数上下文）
   - 函数声明
   - 变量声明

   ```javascript
   function foo(a) {
     var b = 2;
     function c() {}
     var d = function() {};
   
     b = 3;
   
   }
   
   foo(1);
   ```

   进入执行上下文后，AO就是： (**只初始化形参**)

   ```javascript
   AO = {
       arguments: {
           0: 1,
           length: 1
       },
       a: 1,
       b: undefined,
       c: reference to function c(){},
       d: undefined
   }
   ```

   代码执行阶段，会根据代码修改变量对象的值

   ```javascript
   AO = {
       arguments: {
           0: 1,
           length: 1
       },
       a: 1,
       b: 3,
       c: reference to function c(){},
       d: reference to FunctionExpression "d"
   }
   ```

7. 总结如下

   - 全局上下文的变量对象初始化是全局对象
   - 函数上下文的变量对象初始化只包括Arguments对象
   - 在进入执行上下文时会给变量对象添加形参、函数声明、变量声明等初始化的属性值
   - 在代码执行阶段，会再次修改变量对象的属性值

8. ```javascript
   console.log(foo)
   function foo() {
     console.log("foo")
   }
   var foo = 1
   ```

   这段代码会输出函数而不是undefined，因为在进入执行上下文时，首先会处理函数声明，其次在处理变量声明，如果变量名称和已经声明的形式参数或函数相同，变量声明不会干扰已经存在的这类属性。

9. 未进入执行阶段之前，变量对象（VO）中的属性都不能访问！但是进入执行阶段之后，变量对象（VO）转变为了活动对象（AO），里面的属性都能被访问了。VO和AO其实都是同一个对象，只是处于执行上下文的不同生命周期。

### 作用域链

1. 当查找变量的时候，会先从当前上下文的变量对象中查找，如果没有找到，就会从父级执行上下文的变量对象中查找，一直找到全局上下文的变量对象，这样由多个执行上下文的变量对象构成的链表就是作用域链

2. 函数有一个内部属性[[scope]]，当函数创建的时候，就会保存所有父变量到其中，可以理解[[scope]]就是所有父变量对象的层级链。但是[[scope]]并不代表完整的作用域链

   ```javascript
   function foo() {
       function bar() {
           ...
       }
   }
   ```

   函数创建时，各自的[[scope]]为：

   ```javascript
   foo.[[scope]] = [
   	globalContext.VO
   ]
   
   bar.[[scope]] = [
   	fooContext.AO,
   	globalContext.VO
   ]
   ```

3. 当函数激活时，进入函数上下文，创建VO/AO后，就会将活动对象添加到作用链的前端。

   这时候执行上下文的作用域链，命名为`ScopeChain`

   ```javascript
   ScopeChain = [AO].contact(func.[[scope]])
   ```

4. 总结函数执行上下文中作用域链和变量对象的创建过程

   ```javascript
   var scope = "global scope";
   function checkscope(){
       var scope2 = 'local scope';
       return scope2;
   }
   checkscope();
   ```

   - checkscope函数被创建，保存作用域链到内部属性[[scope]]

   ```javascript
   checkscope.[[scope]] = [
     globalContext.VO
   ]
   ```

   - 执行checkscope函数，创建checkscope函数执行上下文，checkscope函数执行上下文被压入执行上下文栈

   ```javascript
   ECStack = [
     checkscopeContext
     globalContext
   ]
   ```

   - checkscope函数并不立即执行，开始作准备工作，第一步：复制函数[[scope]]属性创建作用域链

   ```javascript
   checkscopeContext = {
     ScopeChain: checkscope.[[scope]]
   }
   ```

   - 用arguments创建活动对象，随后初始化活动对象，加入形参、函数声明、变量声明

   ```javascript
   checkscopeContext = {
     AO: {
       arguments: {
         length: 0
       },
       scope2: undefined
     },
     ScopeChain: checkscope.[[scope]]
   }
   ```

   - 将活动对象压入 checkscope作用域链到顶端

   ```javascript
   checkscopeContext = {
     AO: {
       arguments: {
         length: 0
       },
       scope2: undefined
     },
     ScopeChain: [AO, checkscope.[[scope]]]
   }
   ```

   - 准备工作做完，开始执行函数，随着函数的执行，修改AO的属性值

   ```javascript
   checkscopeContext = {
       AO: {
           arguments: {
               length: 0
           },
           scope2: 'local scope'
       },
       Scope: [AO, [[Scope]]]
   }
   ```

   - 查找到scope2 的值，返回后函数执行完毕，函数上下文从执行上下文栈中弹出

   ```javascript
   ECStack = [
     globalContext
   ]
   ```

### 从ECMA规范解读this

1. [参考文章Reference](https://github.com/mqyqingfeng/Blog/issues/7)，[ECMAScript 5.1规范中文版](http://yanhaijing.com/es5/#115)

2. ECMAScript 的类型分为语言类型和规范类型。

   ECMAScript语言类型是开发者直接使用ECMAScript 可以操作的。其实就是我们常说的Undefined, Null, Boolean, String, Number和Object。 

   而规范类型相当于meta-values，是用来用算法描述ECMAScript语言结构和ECMAScript语言类型的。规范类型包括：`Reference` ,`List`, `Completion`, `Property Descriptor`, `Property Identifier`, `Lexical Environmnet`, `Environment Record`。

   在ECMA规范中还有一种只存在于规范中的类型，它们的作用是用来描述语言底层行为逻辑

3. 规范第 8.7 章 The Reference Specification Type: 

   > The Reference type is used to explain the behaviour of such operators as delete, typeof, and the assignment operators

   Reference 类型就是用来解释诸如 delete、typeof、以及赋值等操作行为的。

4. 尤雨溪的话

   > 这里的 Reference 是一个 Specification Type ， 也就是“只存在于规范里的抽象类型”。它们是为了更好地描述语言的底层行为逻辑才存在的，但并不存在于实际的js代码中。

5. Reference 由三个组成部分组成，分别是

   - base value
   - Referenced name
   - strict reference

6. 获取Reference组成部分的方法，比如 `GetBase` 和 `IsPropertyReference`.

7. `GetBase` 

   > GetBase(V). Returns the base value component of the reference V.

   返回 reference 的 base value

8. `IsPropertyReference`

   > IsPropertyReference(V). Returs true if either the base value is an object or HasPrimitiveBase(V) is true; otherwise returns false.

   如果base value是一个对象，就返回true

9. 用于从Reference 类型获取对应值的方法： `GetValue`

   ```javascript
   var foo = 1;
   var fooReference = {
     base: EnvironmentRecord,
     name: 'foo',
     strict: false
   };
   GetValue(fooReference) // 1
   ```

   `GetValue`返回对象属性真正的值，但是要注意：

   **调用GetValue, 返回的将是具体的值，而不再是一个Reference**


10. 上面介绍了这么多Reference的内容，看一下 函数调用的时候，如何确定this的值。规范第11.2.3 Function Calls 里所讲，只看第一步、第六步、第七步

    > 1. let *ref* be the result of evaluation MemberExpression.
    >
    > 6. if Type(*ref*) is Reference, then
    >
    >    If IspropertyReference(ref) is true, then
    >
    >    ​	let **thisValue** be GetBase(ref).
    >
    >    Else, the base of ref is an Environment Record
    >
    >    ​	let **thisValue** be the result of calling the ImplicitThisValue concrete method of GetBase(ref).
    >
    > 7. Else, Type(*ref*) is not Reference
    >
    >    let **thisValue** be undefined
    
11. - 计算 MemberExpression 的结果赋值给 ref

      MemberExpression 见规范11.2 Left-Hand-Side Expressions: 

      可以简单理解MemberExpression 其实就是 () 左边的部分

    - 判断 ref 是不是一个Reference类型

    ```javascript
    var value = 1;
    
    var foo = {
      value: 2,
      bar: function () {
        return this.value;
      }
    }
    
    //示例1
    console.log(foo.bar());
    //示例2
    console.log((foo.bar)());
    //示例3
    console.log((foo.bar = foo.bar)());
    //示例4
    console.log((false || foo.bar)());
    //示例5
    console.log((foo.bar, foo.bar)());
    ```

    - 示例1 MemberExpression 计算的结果是foo.bar， 规范11.2.1 Property Accessors有写

      > Return a value of type Reference whose base value is baseValue and whose referenced name is propertyNameString, and whose strict mode flag is strict.

      我们得知 foo.bar()返回了一个Reference. 按照之前所述，我们接下来判断`IspropertyReference`的值，如果 base value是一个对象就返回true.这里base value是foo， 是一个对象，所以 `IsPropertyReference(ref)`结果为true.

      > let **thisValue** be GetBase(ref).

      最后我们确定this的值，GetBase是返回reference的base value，这个例子中是foo，所以this的值是foo，示例1的结果就是2.

    - 示例2 `(foo.bar)()` 规范11.1.6 The Grouping Operator

      > Return the result of evaluation Expression. This may be of type Reference.

      > NOTE **This algorrithm does not apply GetValue to the result of evaluating Expression !!!**

      左边的括号括起来并没有对foo.bar进行计算，所以这个的结果和示例1一样

    - 示例3 `(foo.bar = foo.bar)()`

      这里有赋值操作，规范里清晰的说明这里调用了`GetValue(ref)`方法，所以返回的值不是Reference类型，按照之前所述

      > Else, Type(*ref*) is not Reference
      >
      > let **thisValue** be undefined

      在非严格模式下， this 的值为 undefined 的时候，其值 会被隐式转换为全局对象。

    - 示例4 `(false || foo.bar)()` 在规范11.11 Binary Logifal Operators这章里可以看到 

      > Let lval be GetValue(lref)

      会调用GetValue方法，即计算表达式，返回的不再是Reference类型，固this为undefined

    - 示例5 `(foo.bar, foo.bar)()`

      逗号操作符, 规范11.14 Comma Operator 

      > 2. Call GetValue(lref)

      也会调用GetValue方法去计算表达式，所以返回的不是Reference类型，this为undefined

12. 上文的例子的结果就是（非严格模式下）
    
	```javascript
    var value = 1;
    
    var foo = {
      value: 2,
      bar: function () {
        return this.value;
      }
    }
    
    //示例1
    console.log(foo.bar()); // 2
    //示例2
    console.log((foo.bar)()); // 2
    //示例3
    console.log((foo.bar = foo.bar)()); // 1
    //示例4
    console.log((false || foo.bar)()); // 1
    //示例5
    console.log((foo.bar, foo.bar)()); // 1
	```


13. 最后，最普通的情况下
    
    ```javascript
    function foo() {
        console.log(this)
    }
    
    foo(); 
    ```
    
    这里MemberExpression 是foo， 解析标识符 规范10.3.1 Identifier Resolution 会返回一个 Reference 类型的值
    
    ```javascript
    var fooReference = {
      base: EnvironmentRecord,
      name: 'foo',
      strict: false
    }
    ```
    
    上文所述
    
    > Else, the base of ref is an Environment Record
    >
    > ​	let **thisValue** be the result of calling the ImplicitThisValue concrete method of GetBase(ref).
    
    查看规范 10.2.1.1.6，ImplicitThisValue方法的介绍：该函数始终返回 undefined
    
    所以最后的this值就是undefined, **这也是为什么作为函数调用的时候 this会指向 window的原因**

### 深入执行上下文

1. 对每个执行上下文，都有三个重要的属性

   - [变量对象](#变量对象)
   - [作用域链](#作用域链)
   - [从ECMA规范解读this](#从ECMA规范解读this)

2. 结合之前所述 [词法作用域和动态作用域](#Javascript 词法作用域和动态作用域)、[执行上下文栈](#Javascript 执行上下文栈)来分析一下下面这段代码

   ```javascript
   var scope = "global scope";
   function checkscope(){
       var scope = "local scope";
       function f(){
           return scope;
       }
       return f();
   }
   checkscope();
   ```

   1. 执行全局代码，创建全局执行上下文，全局上下文被压入执行上下文栈

   ```javascript
   ECStack = [
     globalContext
   ]
   ```

   2. 全局上下文初始化

   ```javascript
   globalContext = {
     VO: [global],
     ScopeChain: [globalContext.VO],
     this: globalContext.VO
   }
   ```

   2. 初始化的同时，checkscope  函数被创建，保存作用域链到函数的内部属性[[scope]]

   ```javascript
   checkscope.[[scope]] = [globalContext.VO]
   ```

   3. 执行checkscope函数，创建checkscopre函数执行上下文，checkscopre函数执行上下文被压入执行上下文栈

   ```javascript
   ECStack = [
     checkscopeContext,
     globalContext
   ]
   ```

   4. checkscope函数执行上下文初始化：

       - 复制函数[[scope]]属性创建作用域链
       - 用 arguments 创建活动对象
       - 初始化活动对象，即加入形参、函数声明、变量声明，
       - 将活动对象压入checkscope作用域链顶端

       同时函数f 被创建，保存作用域链到f函数的内部属性[[scope]]

   ```javascript
   checkscopeContext = {
     AO: {
       arguments: {
         length: 0
       },
       scope: undefined,
       f: reference to function f(){}
     },
     ScopeChain: [AO, globalContext.VO],
     this: undefined
   }
   ```

   5. 执行 f 函数，创建 f 函数执行上下文，f 函数执行上下文被压入执行上下文栈

   ```javascript
   ECStack = [
     fContext,
     checkscopeContext,
     globalContext
   ]
   ```

   6. f 函数执行上下文初始化，一下跟第4步相同：
      - 复制函数[[scope]]属性创建作用域链
      - 用 arguments 创建活动对象
      - 初始化活动对象，即加入形参、函数声明、变量声明，
      - 将活动对象压入 f 作用域链顶端
   7. f 函数执行，沿着作用域链查找scope值，返回scope值
   8. f 函数执行完毕，f 函数上下文从执行上下文栈中弹出

   ```javascript
   ECStack = [
     checkscopeContext,
     globalContext
   ]
   ```

   9. checkscope函数执行完毕，checkscope执行上下文从执行上下文栈中弹出

   ```javascript
   ECStack = [
     globalContext
   ]
   ```
### 闭包

1. MDN对闭包的定义为

   > 闭包是指那些能够访问自由变量的函数

   > 自由变量是指在函数中使用的，但既不是函数参数也不是函数的局部变量的变量

2. 理论上说所有的JavaScript函数都是闭包，但是在实践角度的闭包指：以下函数才算是闭包：

   - 即使创建它的上下文已经销毁，它仍然存在（比如，内部函数从父函数中返回）
   - 在代码中引用了自由变量

3. 我们主要关注的就是实践中的闭包

   ```javascript
   var scope = "global scope";
   function checkscope(){
       var scope = "local scope";
       function f(){
           return scope;
       }
       return f;
   }
   
   var foo = checkscope();
   foo();
   ```

   我们可能会好奇，当`f`函数执行的时候，checkscope函数上下文已经被销毁，怎么还会读取到checkscope作用域下的scope值呢？

   这里 f 的执行上下文维护了一个作用域链

   ```javascript
   fContext = {
     Scope: [AO, checkscopeContext.AO, globalContext.VO]
   }
   ```

   因为 这个作用域链，f 函数依然可以读取到 `checkscopeContext.AO` 的值，说明当 f 函数引用了checkscopeContext.AO中的值的时候，即使checkscopeContext 被销毁了，但是Javascript依然会让checkscopeContext.AO活在内存中， f 函数依然可以通过 f 函数的作用域找到它，正是因为Javascipt做到了这点，从而实现了闭包这个概念。

4. 有个小点需要在这里记一笔`setTimeout`的第三个参数是附加参数，一旦定时器到期，会作为参数传递给 `function`

### 深拷贝

1. 这个问题通常可以通过JSON.parse(JSON.stringify(object))解决

2. 但是这个方法会忽略`undefined`、`symbol`、不能序列化函数，不能解决循环引用的对象

3. 手动实现jQuery.extend功能，它可以合并两个或更多的对象的内容到第一个对象中。

   ```javascript
   jQuery.extend( target [, object1 ] [, objectN ] )
   ```

4. 






















​     

​     


