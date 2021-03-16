# JS前端基础理论-----函数调用栈和this的关系

## [JavaScript的工作原理：引擎、运行时和调用堆栈](https://my.oschina.net/u/4581713/blog/4565092)

随着JavaScript变得越来越流行，越来越多的团队正在利用他们为技术栈中做多个级别的支持：前端、后端、混合应用、嵌入式设备等等。

本文旨在深入挖掘JavaScript及其实际的工作方式：我们认为通过了解JavaScript的构建块以及它们如何发挥作用，你将能够编写更好的代码和应用。 我们还将分享自己在构建SessionStack[https://www.sessionstack.com/]时使用的一些经验和规范，这是一个轻量级JavaScript应用，必须具有强大功能和高性能才能保持竞争力。

正如GitHut stats[http://githut.info/]所示，JavaScript在GitHub中的Active Repositories和Total Pushes方面处于领先地位。 它也不会落后于其他语言。

![img](https://oscimg.oschina.net/oscnet/ae2bc5d9-4e52-4511-a319-bf92ee659e26.png)

（查看最新的GitHub语言统计信息https://madnight.github.io/githut/ ）

如果项目越来越依赖于JavaScript，这意味着开发人员必须利用语言和其生态系统提供的所有内容，更深入的了解其内部，以便构建出色的软件。

事实证明，有很多开发人员每天都在使用JavaScript，却不了解背后究竟发生了些什么。



### 概述

几乎每个人都已经听说过V8引擎这个概念，大多数人都知道JavaScript是单线程的，或者它使用的是回调队列。

在本文中，我们将详细介绍这些概念，并解释JavaScript实际运行的方式。 通过了解这些详细信息，你将能够正确地利用其所提供的API编写更好的、非阻塞的应用，这些应用正确地利用了所提供的API。

如果你对JavaScript比较陌生，那么本文将帮助你理解为什么JavaScript与其他语言相比是如此的“奇怪”。

如果你是一位经验丰富的JavaScript开发者，尽管你每天使用它，但仍然希望它能够为你提供一些关于JavaScript运行时工作方式方面的新见解。



### JavaScript引擎

一个很流行的JavaScript引擎是Google的V8引擎。 V8引擎被用于Chrome和Node.js。 这是一个非常简化的示意图：

![img](https://oscimg.oschina.net/oscnet/6a45c556-a3b3-4198-9f3f-a8229a806c84.png)

引擎包含两个主要组件：

- 内存堆 - 这是进行内存分配的地方
- 调用栈 - 这是你的代码执行时堆栈帧的位置



### 运行时

这是几乎所有JavaScript开发人员在浏览器中都使用过的API（例如“setTimeout”）。 但是引擎并不提供这些API。

那么，他们究竟来自哪里？

实际上这有点复杂。

![img](https://oscimg.oschina.net/oscnet/98ba763d-e2a4-4489-9ddd-2d06e766d5b4.png)

所以尽管有了引擎，但是还需要很多东西。有一些叫做Web API的东西，它们是由浏览器提供的，比如DOM，AJAX，setTimeout等等。

此外还有非常受欢迎的事件循环和回调队列。



### 调用栈

JavaScript是一种单线程编程语言，这意味着它只有一个调用栈。 所以它一次只能做一件事。

调用栈是一种数据结构，它记录了当前程序中执行到的基本位置。 如果我们进入一个函数，会它放在栈的顶部。 如果我们从函数返回，就会将它从堆栈的顶部弹出。 这就是所有栈结构都可以做到的。

下面我们来看一个例子吧：

![img](https://oscimg.oschina.net/oscnet/65c3f4f9-5140-4205-8f9f-555ed261c5df.png)

当引擎开始执行上面的代码时，调用堆栈将为空。 接下来的步骤如下：

![img](https://oscimg.oschina.net/oscnet/d93631c1-5a1e-4163-87a8-13e817fa0e05.png)

调用栈中的每个条目被称为**栈帧**。

这是在抛出异常时堆栈跟踪的构造方式 —— 当异常发生时调用堆栈的大致状态。 接下来看下面这段代码：

![img](https://oscimg.oschina.net/oscnet/1f050dc4-a7ea-41ff-84c7-9d132116eba3.png)

如果在Chrome中执行这个操作（假设此代码位于名为foo.js的文件中），则将生成以下堆栈跟踪：

![img](https://oscimg.oschina.net/oscnet/d53d122f-8426-43ed-bfaa-ad07fff2266d.png)

当达到最大调用堆栈大小时会发生“**Blowing the stack**”这种情况。 这种情况是很容易发生的，尤其是在你使用递归而没有充分地测试你的代码时。 看一下这段代码：

![img](https://oscimg.oschina.net/oscnet/760cff50-e36b-488c-98cf-f92e82a0f54d.png)

当引擎开始执行此代码时，它首先调用函数“foo”。 但是这个函数是递归的，并且在没有任何终止条件的情况下开始调用自身。 因此在执行的每个步骤中，相同的函数一次又一次地被添加到调用堆栈中。 它看起来像是这样：

![img](https://oscimg.oschina.net/oscnet/4a693baf-9b87-49e4-9a03-36d7da82e61d.png)

在某些时候，如果调用栈中的函数调用数量超过了它的实际大小，浏览器就会抛出错误，该错误看起来像这样：

![img](https://oscimg.oschina.net/oscnet/bfefc48a-ee4d-49ad-b223-3422b1aa3dda.png)

在单个线程上运行代码非常简单，因为你不必处理多线程环境中出现的复杂场景，例如死锁。

但是跑在单个线程上也是非常受限的。 由于JavaScript只有一个调用，**当处理变慢时会发生什么？**



### 并发和事件循环

如果在调用堆栈中有需要花费大量时间才能处理的函数调用，会发生什么？ 比如假设你想在浏览器中用JavaScript进行一些复杂的图像转换。

你可能会问：这也算是一个问题？ 实际上虽然调用栈具有执行功能，但浏览器实并没有办法执行其他的操作，因为它会被阻止。 这意味着浏览器将无法进行渲染，也无法运行任何其他代码，它只是被卡住了。 如果你想在自己的应用中产生流畅的UI，在这里将会出现问题。

这并不是唯一的问题。 一旦你的浏览器开始在调用栈中处理如此之多的任务，它可能会在相当长的时间内停止响应。 大多数浏览器将会通过引发错误来解决这个问题，询问你是否要终止网页的运行。

![img](https://oscimg.oschina.net/oscnet/c805aab7-9b9d-4541-831a-5df6283e1059.png)

所以这并不是最佳的用户体验，对吗？

那么怎样才能在不阻止UI，并使浏览器在无响应的情况下执行繁重的代码呢？ 解决方案是异步回调。

这一点在“如何运行JavaScript”教程的第2部分中有更详细的解释：“在V8引擎是怎么工作的：有关如何编写优化代码的5个技巧[https://blog.sessionstack.com/how-javascript-works-inside-the-v8-engine-5-tips-on-how-to-write-optimized-code-ac089e62b12e]”。

与此同时，如果你在JavaScript应用程序中遇到难以复制和理解的问题，可以试试SessionStack[https://www.sessionstack.com/?utm_source=medium&utm_medium=blog&utm_content=Post-1-overview-outro]。 SessionStack会记录Web应用中所有的内容：所有的DOM修改、用户交互、JavaScript异常、堆栈跟踪、网络请求失败和调试消息。

通过SessionStack，你可以将网络应用中的问题重现，并查看发生的所有事情。

有一个免费的工具，不需要支付任何费用。 现在就可以试试[https://www.sessionstack.com/solutions/developers/?utm_source=medium&utm_medium=blog&utm_content=Post-1-overview-getStarted]。

![img](https://oscimg.oschina.net/oscnet/ec484b99-918a-4d74-a18f-d19d24ec773c.png)



## this 和 函数调用栈的关系

在理解 this 的绑定过程之前， 首先要理解调用位置 ，最重要的是要分析调用栈（就是为了到达当前执行位置所调用的所有函数）。   

把调用栈想象成一个函数调用链， 就像我们在前面代码段的注释中所写的一样。 但是这种方法非常麻烦并且容易出错。 另一个查看调用栈的方法是使用浏览器的调试工具。 绝大多数现代桌面浏览器都内置了开发者工具，其中包含 JavaScript 调试器。 就本例来说， 你可以在工具中给 foo() 函数的第一行代码设置一个断点， 或者直接在第一行代码之前插入一条 debugger;语句。 运行代码时， 调试器会在那个位置暂停， 同时会展示当前位置的函数调用列表， 这就是你的调用栈。 因此， 如果你想要分析 this 的绑定， 使用开发者工具得到调用栈， 然后找到栈中第二个元素， 这就是真正的调用位置。  



### 默认绑定

首先要介绍的是最常用的函数调用类型： 独立函数调用。 可以把这条规则看作是无法应用其他规则时的默认规则。
思考一下下面的代码：

```javascript
function foo() {
console.log( this.a );
}  

var a = 2;
foo(); // 2
```

你应该注意到的第一件事是， 声明在全局作用域中的变量（比如 var a = 2） 就是全局对象的一个同名属性。 它们本质上就是同一个东西， 并不是通过复制得到的， 就像一个硬币的两面一样。

接下来我们可以看到当调用 foo() 时， this.a 被解析成了全局变量 a。 为什么？ 因为在本例中， 函数调用时应用了 this 的默认绑定， 因此 this 指向全局对象。

那么我们怎么知道这里应用了默认绑定呢？ 可以通过分析调用位置来看看 foo() 是如何调用的。 在代码中， foo() 是直接使用不带任何修饰的函数引用进行调用的， 因此只能使用默认绑定， 无法应用其他规则。

**如果使用严格模式（strict mode）， 那么全局对象将无法使用默认绑定， 因此 this 会绑定到 undefined**  



### 隐式绑定

另一条需要考虑的规则是调用位置是否有上下文对象， 或者说是否被某个对象拥有或者包含， 不过这种说法可能会造成一些误导。

思考下面的代码：

```javascript
function foo() {
	console.log( this.a );
}
var obj = {
    a: 2,
    foo: foo
};
obj.foo(); // 2
```

首先需要注意的是 foo() 的声明方式， 及其之后是如何被当作引用属性添加到 obj 中的。但是无论是直接在 obj 中定义还是先定义再添加为引用属性， 这个函数严格来说都不属于obj 对象。

然而， 调用位置会使用 obj 上下文来引用函数， 因此你可以说函数被调用时 obj 对象“拥有” 或者“包含” 它。

无论你如何称呼这个模式， 当 foo() 被调用时， 它的落脚点确实指向 obj 对象。 当函数引用有上下文对象时， 隐式绑定规则会把函数调用中的 this 绑定到这个上下文对象。 因为调用 foo() 时 this 被绑定到 obj， 因此 this.a 和 obj.a 是一样的  



### 显式绑定

就像我们刚才看到的那样， 在分析隐式绑定时， 我们必须在一个对象内部包含一个指向函数的属性， 并通过这个属性间接引用函数， 从而把 this 间接（隐式） 绑定到这个对象上。那么如果我们不想在对象内部包含函数引用， 而想在某个对象上强制调用函数， 该怎么做呢？

JavaScript 中的“所有” 函数都有一些有用的特性（这和它们的 [[ 原型 ]] 有关——之后我们会详细介绍原型）， 可以用来解决这个问题。 具体点说， 可以使用函数的 call(..) 和apply(..) 方法。 严格来说， JavaScript 的宿主环境有时会提供一些非常特殊的函数， 它们并没有这两个方法。 但是这样的函数非常罕见， JavaScript 提供的绝大多数函数以及你自己创建的所有函数都可以使用 call(..) 和 apply(..) 方法。

这两个方法是如何工作的呢？ 它们的第一个参数是一个对象， 它们会把这个对象绑定到this， 接着在调用函数时指定这个 this。 因为你可以直接指定 this 的绑定对象， 因此我们称之为显式绑定。

```javascript
function foo() {
console.log( this.a );
}
var obj = {
a:2
};
foo.call( obj ); // 2
```

通过 foo.call(..)， 我们可以在调用 foo 时强制把它的 this 绑定到 obj 上。如果你传入了一个原始值（字符串类型、 布尔类型或者数字类型） 来当作 this 的绑定对象， 这个原始值会被转换成它的对象形式（也就是 new String(..)、 new Boolean(..) 或者new Number(..)）。 这通常被称为“装箱”  

可惜， **显式绑定仍然无法解决我们之前提出的丢失绑定问题。**  

- ####  硬绑定

但是显式绑定的一个变种可以解决这个问题。
思考下面的代码：

```javascript
function foo() {
console.log( this.a );
}
var obj = {
a:2
};
var bar = function() {
foo.call( obj );
};
bar(); // 2
setTimeout( bar, 100 ); // 2
// 硬绑定的 bar 不可能再修改它的 this
bar.call( window ); // 2 
```

我们来看看这个变种到底是怎样工作的。 我们创建了函数 bar()， 并在它的内部手动调用了 foo.call(obj)， 因此强制把 foo 的 this 绑定到了 obj。 无论之后如何调用函数 bar， 它总会手动在 obj 上调用 foo。 这种绑定是一种显式的强制绑定， 因此我们称之为硬绑定。硬绑定的典型应用场景就是创建一个包裹函数， 传入所有的参数并返回接收到的所有值：  

另一种使用方法是创建一个 i 可以重复使用的辅助函数：

```javascript
function foo(something) {
    console.log( this.a, something );
    return this.a + something;
} // 简单的辅助绑定函数
function bind(fn, obj) {
    return function() {
    	return fn.apply( obj, arguments );
    };
}
var obj = {
	a:2
};
var bar = bind( foo, obj );
var b = bar( 3 ); // 2 3
console.log( b ); // 5
```

由于硬绑定是一种非常常用的模式， 所以在 ES5 中提供了内置的方法 Function.prototype.bind， 它的用法如下：

```javascript
function foo(something) {
    console.log( this.a, something );
    return this.a + something;
}  

var obj = {
	a:2
};
var bar = foo.bind( obj );
var b = bar( 3 ); // 2 3
console.log( b ); // 5
```

bind(..) 会返回一个硬编码的新函数， 它会把参数设置为 this 的上下文并调用原始函数。  

- API调用的“上下文”

第三方库的许多函数， 以及 JavaScript 语言和宿主环境中许多新的内置函数， 都提供了一个可选的参数， 通常被称为“上下文”（context）， 其作用和 bind(..) 一样， 确保你的回调函数使用指定的 this。
举例来说：

```javascript
function foo(el) {
	console.log( el, this.id );
}
var obj = {
	id: "awesome"
};
// 调用 foo(..) 时把 this 绑定到 obj
[1, 2, 3].forEach( foo, obj );
// 1 awesome 2 awesome 3 awesome
```

这些函数实际上就是通过 call(..) 或者 apply(..) 实现了显式绑定， 这样你可以少些一些代码。  

### new绑定

这是第四条也是最后一条 this 的绑定规则， 在讲解它之前我们首先需要澄清一个非常常见的关于 JavaScript 中函数和对象的误解。

在传统的面向类的语言中，“构造函数” 是类中的一些特殊方法， 使用 new 初始化类时会调用类中的构造函数。 通常的形式是这样的：

```javascript
something = new MyClass(..);
```

JavaScript 也有一个 new 操作符， 使用方法看起来也和那些面向类的语言一样， 绝大多数开发者都认为 JavaScript 中 new 的机制也和那些语言一样。 然而， JavaScript 中 new 的机制实际上和面向类的语言完全不同。  



使用 new 来调用函数， 或者说发生构造函数调用时， 会自动执行下面的操作。

1.  创建（或者说构造） 一个全新的对象。
2.  这个新对象会被执行 [[ 原型 ]] 连接。
3.  这个新对象会绑定到函数调用的 this。
4.  如果函数没有返回其他对象， 那么 new 表达式中的函数调用会自动返回这个新对象。

我们现在关心的是第 1 步、 第 3 步、 第 4 步， 所以暂时跳过第 2 步， 第 5 章会详细介绍它。
思考下面的代码：

```javascript
function foo(a) {
this.a = a;
}
var bar = new foo(2);
console.log( bar.a ); // 2
```

使用 new 来调用 foo(..) 时， 我们会构造一个新对象并把它绑定到 foo(..) 调用中的 this上。 new 是最后一种可以影响函数调用时 this 绑定行为的方法， 我们称之为 new 绑定  

## 判断this

现在我们可以根据优先级来判断函数在某个调用位置应用的是哪条规则。 可以按照下面的顺序来进行判断：

1. 函数是否在 new 中调用（new 绑定） ？ 如果是的话 this 绑定的是新创建的对象。

   ```javascript
   var bar = new foo()
   ```

   

2. 函数是否通过 call、 apply（显式绑定） 或者硬绑定调用？ 如果是的话， this 绑定的是
   指定的对象。

   ```javascript
   var bar = foo.call(obj2)
   ```

   

3. 函数是否在某个上下文对象中调用（隐式绑定） ？ 如果是的话， this 绑定的是那个上
   下文对象。

   ```javascript
   var bar = obj1.foo()
   ```

   

4. 如果都不是的话， 使用默认绑定。 如果在严格模式下， 就绑定到 undefined， 否则绑定到
   全局对象。

   ```javascript
   var bar = foo()
   ```

   

就是这样。 对于正常的函数调用来说， 理解了这些知识你就可以明白 this 的绑定原理了。不过……凡事总有例外。  

## 绑定例外  

### 被忽略的this

如果你把 null 或者 undefined 作为 this 的绑定对象传入 call、 apply 或者 bind， 这些值
在调用时会被忽略， 实际应用的是默认绑定规则：

```javascript
function foo() {
	console.log( this.a );
} 
var a = 2;
foo.call( null ); // 2
```

那么什么情况下你会传入 null 呢？

一种非常常见的做法是使用 apply(..) 来“展开” 一个数组， 并当作参数传入一个函数。类似地， bind(..) 可以对参数进行柯里化（预先设置一些参数）， 这种方法有时非常有用：

```javascript
function foo(a,b) {
	console.log( "a:" + a + ", b:" + b );
} 

// 把数组“展开” 成参数
foo.apply( null, [2, 3] ); // a:2, b:3
// 使用 bind(..) 进行柯里化
var bar = foo.bind( null, 2 );
bar( 3 ); // a:2, b:3
```

这两种方法都需要传入一个参数当作 this 的绑定对象。 如果函数并不关心 this 的话， 你仍然需要传入一个占位值， 这时 null 可能是一个不错的选择， 就像代码所示的那样。  

> 尽管本书中未提到， 但在 ES6 中， 可以用 ... 操作符代替 apply(..) 来“展开” 数组， foo(...[1,2]) 和 foo(1,2) 是一样的， 这样可以避免不必要的this 绑定。 可惜， 在 ES6 中没有柯里化的相关语法， 因此还是需要使用bind(..)。  

##### 更安全的this

一种“更安全” 的做法是传入一个特殊的对象， 把 this 绑定到这个对象不会对你的程序产生任何副作用。   

无论你叫它什么， 在 JavaScript 中创建一个空对象最简单的方法都是 Object.create(null)（详 细 介 绍 请 看 第 5 章 ）。 Object.create(null) 和 {} 很 像， 但 是 并 不 会 创 建 Object.prototype 这个委托， 所以它比 {}“更空” ：

```javascript
function foo(a,b) {
	console.log( "a:" + a + ", b:" + b );
} 
// 我们的 DMZ 空对象
var ø = Object.create( null );
// 把数组展开成参数
foo.apply( ø, [2, 3] ); // a:2, b:3
// 使用 bind(..) 进行柯里化
var bar = foo.bind( ø, 2 );
bar( 3 ); // a:2, b:3
```

使用变量名 ø 不仅让函数变得更加“安全”， 而且可以提高代码的可读性， 因为 ø 表示“我希望 this 是空”， 这比 null 的含义更清楚。 不过再说一遍， 你可以用任何喜欢的名字来命名 DMZ 对象。  

### 间接引用

另一个需要注意的是， 你有可能（有意或者无意地） 创建一个函数的“间接引用”， 在这种情况下， 调用这个函数会应用默认绑定规则。

间接引用最容易在赋值时发生：

```javascript
function foo() {
console.log( this.a );
}  

var a = 2;
var o = { a: 3, foo: foo };
var p = { a: 4 };
o.foo(); // 3
(p.foo = o.foo)(); // 2  
```

赋值表达式 p.foo = o.foo 的返回值是目标函数的引用， 因此调用位置是 foo() 而不是p.foo() 或者 o.foo()。 根据我们之前说过的， 这里会应用默认绑定。

注意： 对于默认绑定来说， 决定 this 绑定对象的并不是调用位置是否处于严格模式， 而是函数体是否处于严格模式。 如果函数体处于严格模式， this 会被绑定到 undefined， 否则this 会被绑定到全局对象。  

### 软绑定

之前我们已经看到过， 硬绑定这种方式可以把 this 强制绑定到指定的对象（除了使用 new时）， 防止函数调用应用默认绑定规则。 问题在于， 硬绑定会大大降低函数的灵活性， 使用硬绑定之后就无法使用隐式绑定或者显式绑定来修改 this。

如果可以给默认绑定指定一个全局对象和 undefined 以外的值， 那就可以实现和硬绑定相同的效果， 同时保留隐式绑定或者显式绑定修改 this 的能力。

可以通过一种被称为软绑定的方法来实现我们想要的效果：

```javascript
if (!Function.prototype.softBind) {
    Function.prototype.softBind = function(obj) {
        var fn = this;
        // 捕获所有 curried 参数
        var curried = [].slice.call( arguments, 1 );
        var bound = function() {
            return fn.apply(
                (!this || this === (window || global)) ?
                obj : this
                curried.concat.apply( curried, arguments )
            );
        };
        bound.prototype = Object.create( fn.prototype );
        return bound;
    };
}
```

除了软绑定之外， softBind(..) 的其他原理和 ES5 内置的 bind(..) 类似。 它会对指定的函数进行封装， 首先检查调用时的 this， 如果 this 绑定到全局对象或者 undefined， 那就把指定的默认对象 obj 绑定到 this， 否则不会修改 this。 此外， 这段代码还支持可选的柯里化（详情请查看之前和 bind(..) 相关的介绍）。  



## this词法

我们之前介绍的四条规则已经可以包含所有正常的函数。 但是 ES6 中介绍了一种无法使用这些规则的特殊函数类型： 箭头函数。

箭头函数并不是使用 function 关键字定义的， 而是使用被称为“胖箭头” 的操作符 => 定义的。 箭头函数不使用 this 的四种标准规则， 而是根据外层（函数或者全局） 作用域来决定 this。

我们来看看箭头函数的词法作用域：

```javascript
function foo() {
// 返回一个箭头函数
    return (a) => {
        //this 继承自 foo()
        console.log( this.a );
    };
}
var obj1 = {
	a:2
};
var obj2 = {
	a:3  
};
var bar = foo.call( obj1 );
bar.call( obj2 ); // 2, 不是 3 ！
```

  

foo() 内部创建的箭头函数会捕获调用时 foo() 的 this。 由于 foo() 的 this 绑定到 obj1，
bar（引用箭头函数） 的 this 也会绑定到 obj1， 箭头函数的绑定无法被修改。（new 也不
行！ ）
箭头函数最常用于回调函数中， 例如事件处理器或者定时器：

```javascript
function foo() {
    setTimeout(() => {
        // 这里的 this 在此法上继承自 foo()
        console.log( this.a );
    },100);
}
var obj = {
	a:2
};
foo.call( obj ); // 2
```

箭头函数可以像 bind(..) 一样确保函数的 this 被绑定到指定对象， 此外， 其重要性还体
现在它用更常见的词法作用域取代了传统的 this 机制。 实际上， 在 ES6 之前我们就已经
在使用一种几乎和箭头函数完全一样的模式。

```javascript
function foo() {
    var self = this; // lexical capture of this
    setTimeout( function(){
    	console.log( self.a );
    }, 100 );
}
var obj = {
	a: 2
};
foo.call( obj ); // 2
```

虽然 self = this 和箭头函数看起来都可以取代 bind(..)， 但是从本质上来说， 它们想替
代的是 this 机制。  



如果你经常编写 this 风格的代码， 但是绝大部分时候都会使用 self = this 或者箭头函数
来**否定 this 机制**， 那你或许应当：

1. 只使用词法作用域并完全抛弃错误 this 风格的代码；
2. 完全采用 this 风格， 在必要时使用 bind(..)， 尽量避免使用 self = this 和箭头函数。  



> ### 小结
>
> 如果要判断一个运行中函数的 this 绑定， 就需要找到这个函数的直接调用位置。 找到之后
> 就可以顺序应用下面**这四条规则来判断 this 的绑定对象**。
>
> 1. 由 new 调用？ 绑定到新创建的对象。
> 2. 由 call 或者 apply（或者 bind） 调用？ 绑定到指定的对象。
> 3. 由上下文对象调用？ 绑定到那个上下文对象。
> 4. 默认： 在严格模式下绑定到 undefined， 否则绑定到全局对象。
>
> 一定要注意， 有些调用可能在无意中使用默认绑定规则。 如果想“更安全” 地忽略 this 绑定， 你可以使用一个 DMZ 对象， 比如 ø = Object.create(null)， 以保护全局对象。
>
> ES6 中的箭头函数并不会使用四条标准的绑定规则， 而是根据当前的词法作用域来决定this， 具体来说， 箭头函数会继承外层函数调用的 this 绑定（无论 this 绑定到什么）。 这其实和 ES6 之前代码中的 self = this 机制一样。  

