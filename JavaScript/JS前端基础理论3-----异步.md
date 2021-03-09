# JS前端基础理论-----异步

## 事件循环  

ES6 JavaScript 才真正内建有直接的异步概念。  

## 并⾏线程  

术语“异步”和“并⾏”常常被混为⼀谈，但实际上它们的意义完全不同。记住，

- 异步是关于现在 和将来 的时间间隙
- 并⾏是关于能够同时发⽣的事情。  

并⾏计算最常⻅的⼯具就是进程 和线程 。进程和线程独⽴运⾏，并可能同时运⾏：在不同的处理器，甚⾄不同的计算机上，但多个线程能够共享单个进程的内存。  

事件循环把⾃⾝的⼯作分成⼀个个任务并顺序执⾏，不允许对共享内存的并⾏访问和修改。通过分⽴线程中彼此合作的事件循环，并⾏和顺序执⾏可以共存。  

JavaScript 从不跨线程共享数据，这意味着不需要考虑这⼀层次的不确定性。  

## 完整运⾏

由于 JavaScript 的单线程特性，foo() （以及 bar() ）中的代码具有原⼦性。也就是说，⼀旦 foo() 开始运⾏，它的所有代码都会在bar() 中的任意代码运⾏之前完成，或者相反。这称为完整运⾏（run-to-completion）特性。  

```javascript
var a = 1;
var b = 2;
function foo() {
    a++;
    b = b * a;
    a = b + 3;
}
function bar() {
    b--;
    a = 8 + b;
    b = a * 2;
}  
```

## 竞态条件 （race condition）

foo() 和 bar() 相互竞争，看谁先运⾏。具体来说，因为⽆法可靠预测 a 和 b 的最终结果，所以才是竞态条件。  

## 并发  

两个或多个“进程”同时执⾏就出现了并发，不管组成它们的单个运算是否并⾏ 执⾏（在独⽴的处理器或处理器核⼼上同时运⾏）。可以把并发看作“进程”级（或者任务级）的并⾏，与运算级的并⾏（不同处理器上的线程）相对。  

### ⾮交互  

如果进程间没有相互影响的话，不确定性是完全可以接受的 。  

### 交互  

并发的“进程”需要相互交流，通过作⽤域或 DOM间接交互。正如前⾯介绍的，如果出现这样的交互，就需要对它们的交互进⾏协调以避免竞态的出现。  

#### 门闩 

另⼀种可能遇到的并发交互条件有时称为竞态 （race），但是更精确的叫法是门闩 （latch）。它的特性可以描述为“只有第⼀名取胜”。  

```javascript
var a, b;
function foo(x) {
    a = x * 2;
    if (a && b) {
    	baz();
    }
}
function bar(y) {
    b = y * 2;
    if (a && b) {
    	baz();
    }
}
function baz() {
	console.log( a + b );
}
// ajax(..)是某个库中的某个Ajax函数
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```

### 协作  

还有⼀种并发合作⽅式，称为并发协作 （cooperative concurrency）。这⾥的重点不再是通过共享作⽤域中的值进⾏交互（尽管显然这也是允许的！）。这⾥的**⽬标是取到⼀个⻓期运⾏的“进程”，并将其分割成多个步骤或多批任务**，使得其他并发“进程”有机会将⾃⼰的运算插⼊到事件循环队列中交替运⾏。  

```javascript
var res = [];
// response(..)从Ajax调⽤中取得结果数组
function response(data) {
    // 添加到已有的res数组
    res = res.concat(
    // 创建⼀个新的变换数组把所有data值加倍
        data.map( function(val){
            return val * 2;
        } )
    );
}
// ajax(..)是某个库中提供的某个Ajax函数
ajax( "http://some.url.1", response );
ajax( "http://some.url.2", response );
```

如果记录有⼏千条或更少，这不算什么。但是如果有像 1000 万条记录的话，就可能需要运⾏相当⼀段时间了（在⾼性能笔记本上需要⼏秒钟，在移动设备上需要更⻓时间，等等）  

所以，要创建⼀个协作性更强更友好且不会霸占事件循环队列的并发系统，你可以异步地批处理这些结果。  

这⾥给出⼀种⾮常简单的⽅法：

```javascript
var res = [];
// response(..)从Ajax调⽤中取得结果数组
function response(data) {
    // ⼀次处理1000个
    var chunk = data.splice( 0, 1000 );
    // 添加到已有的res组
    res = res.concat(
    // 创建⼀个新的数组把chunk中所有值加倍
        chunk.map( function(val){
        	return val * 2;
        } )
    );
    // 还有剩下的需要处理吗？
    if (data.length > 0) {
        // 异步调度下⼀次批处理
        setTimeout( function(){
        	response( data );
        }, 0 );
    }
}
// ajax(..)是某个库中提供的某个Ajax函数
ajax( "http://some.url.1", response );
ajax( "http://some.url.2", response );
```

我们把数据集合放在最多包含 1000 条项⽬的块中。这样，我们就确保了“进程”运⾏时间会很短，即使这意味着需要更多的后续“进程”，因为事件循环队列的交替运⾏会提⾼站点 /App 的响应（性能）  

严格说来，**setTimeout(..0) 并不直接把项⽬插⼊到事件循环队列。**定时器会在有机会的时候插⼊事件。举例来说，**两个连续的 setTimeout(..0) 调⽤不能保证会严格按照调⽤顺序处理**，所以各种情况都有可能出现，⽐如**定时器漂移**，在这种情况下，这些事件的顺序就不可预测。在 Node.js 中，类似的⽅法是 process.nextTick(..) 。尽管它们使⽤⽅便（通常性能也更⾼），但并没有（⾄少到⽬前为⽌）直接的⽅法可以适应所有环境来确保异步事件的顺序。下⼀⼩节我们会深⼊讨论这个话题。  

## 任务  

### 任务队列  

在 ES6 中，有⼀个新的概念建⽴在事件循环队列之上，叫作任务队列（job queue）。这个概念给⼤家带来的最⼤影响可能是 Promise 的异步特性（参⻅第 3 章）。

遗憾的是，⽬前为⽌，这是⼀个没有公开 API 的机制

因此，我认为对于任务队列最好的理解⽅式就是，它是挂在事件循环队列的每个 tick 之后的⼀个队列。在事件循环的每个 tick 中，可能出现的异步动作不会导致⼀个完整的新事件添加到事件循环队列中，⽽会在当前 tick 的任务队列末尾添加⼀个项⽬（⼀个任务）。  

这就像是在说：“哦，这⾥还有⼀件事将来 要做，但要确保在其他任何事情发⽣之前就完成它。”  

### 事件循环队列

事件循环队列类似于⼀个游乐园游戏：玩过了⼀个游戏之后，你需要重新到队尾排队才能再玩⼀次。⽽任务队列类似于玩过了游戏之后，插队接着继续玩。  

### 任务循环

理论上说，任务循环 （job loop）可能⽆限循环（⼀个任务总是添加另⼀个任务，以此类推），进⽽导致程序的饿死，⽆法转移到下⼀个事件循环 tick。从概念上看，这和代码中的⽆限循环（就像 while(true)..）的体验⼏乎是⼀样的。  

# 回调  

## 嵌套回调与链式回调  

```javascript
listen( "click", function handler(evt){
    setTimeout( function request(){
        ajax( "http://some.url.1", function response(text){
            if (text == "hello") {
                handler();
            }  
            else if (text == "world") {
                request();
            }
        } );
    }, 500) ;
} );  
```

这种代码常常被称为回调地狱   

链式回调 就是 对嵌套回调的拆解，但是依旧复杂，会造成严重的上下文关联，同时也会有严重的信任依赖。如果任何回调在第三方发生多重异步，则对结果是破坏性的



## 是可信任的 Promise   

如果向 Promise.resolve(..) 传递⼀个⾮ Promise、⾮ thenable
的⽴即值，就会得到⼀个⽤这个值填充的 promise。下⾯这种情况下，
promise p1 和 promise p2 的⾏为是完全⼀样的：

```javascript
var p1 = new Promise( function(resolve,reject){
	resolve( 42 );
} );
var p2 = Promise.resolve( 42 );  
```

⽽如果向 Promise.resolve(..) 传递⼀个真正的 Promise，就只会返回同⼀个 promise：

```javascript
var p1 = Promise.resolve( 42 );
var p2 = Promise.resolve( p1 );
p1 === p2; // true  
```

Promise.resolve(..) 可以接受任何 thenable，将其解封为它的⾮ thenable 值。从 Promise.resolve(..) 得到的是⼀个真正的Promise，是⼀个可以信任的值。如果你传⼊的已经是真正的Promise，那么你得到的就是它本⾝，所以通过Promise.resolve(..) 过滤来获得可信任性完全没有坏处。

假设我们要调⽤⼀个⼯具 foo(..) ，且并不确定得到的返回值是否是⼀个可信任的⾏为良好的 Promise，但我们可以知道它⾄少是⼀个thenable。Promise.resolve(..) 提供了可信任的 Promise 封装⼯具，可以链接使⽤：

```javascript
// 不要只是这么做：
foo( 42 )
.then( function(v){
	console.log( v );
} );

// ⽽要这么做：
Promise.resolve( foo( 42 ) )
.then( function(v){
	console.log( v );
} );  
```

对于⽤ Promise.resolve(..) 为所有函数的返回值（不管是不是 thenable）都封装⼀层。另⼀个好处是，这样做很容易把函数调⽤规范为定义良好的异步任务。如果 foo(42) 有时会返回⼀个⽴即值，有时会返回 Promise，那么Promise.resolve( foo(42) ) 就**能够保证总会返回⼀个Promise 结果**。

## 链式流  

两个 Promise 固有⾏为特性：  

- 调⽤ Promise 的 then(..) 会⾃动创建⼀个新的 Promise 从调⽤返回。
- 在完成或拒绝处理函数内部，如果返回⼀个值或抛出⼀个异常，新返回的（可链接的）Promise 就相应地决议。
- 如果完成或拒绝处理函数返回⼀个 Promise，它将会被展开，这样⼀来，不管它的决议值是什么，都会成为当前 then(..) 返回的链接 Promise 的决议值。  

我们⾸先定义⼀个⼯具 request(..) ，⽤来构造⼀个表⽰ajax(..) 调⽤完成的 promise：

```javascript
request( "http://some.url.1/" )
.then( function(response1){
	return request( "http://some.url.2/?v=" + response1 );
} )
.then( function(response2){
	console.log( response2 );
} );
```



> 开发者常会遇到这样的情况：他们想要通过本⾝并不⽀持Promise 的⼯具（就像这⾥的 ajax(..) ，它接收的是⼀个回调）实现⽀持 Promise 的异步流程控制。虽然原⽣ ES6Promise 机制并不会⾃动为我们提供这个模式，但所有实际的Promise 库都会提供。通常它们把这个过程称为“提升”“promise化”或者其他类似的名称。我们稍后会再介绍这种技术。

利⽤返回 Promise 的 request(..) ，我们通过使⽤第⼀个 URL 调⽤它来创建链接中的第⼀步，并且把返回的 promise 与第⼀个then(..) 链接起来。

response1 ⼀返回，我们就使⽤这个值构造第⼆个 URL，并发出第⼆个 request(..) 调⽤。第⼆个 request(..) 的 promise 返回，以便异步流控制中的第三步等待这个 Ajax 调⽤完成。最后，response2 ⼀返回，我们就⽴即打出结果。  

## Promise  

- Promise ⾮常好，请使⽤。它们解决了我们因只⽤回调的代码⽽备受困扰的控制反转 问题。
- 它们并没有摈弃回调，只是把回调的安排转交给了⼀个位于我们和其他⼯具之间的可信任的中介机制。
- Promise 链也开始提供（尽管并不完美）以顺序的⽅式表达异步流的⼀个更好的⽅法，这有助于我们的⼤脑更好地计划和维护异步  

术语

1. 决议 （resolve）
2. 完成 （fulfill）
3. 拒绝 （reject）  

```javascript
var p = new Promise( function(X,Y){
    // X()⽤于完成
    // Y()⽤于拒绝
} );  
```

前⾯提到的 reject(..) 不会像 resolve(..) ⼀样进⾏展开。**如果向 reject(..) 传⼊⼀个 Promise/thenable值，它会把这个值原封不动地设置为拒绝理由**。后续的拒绝处理函数接收到的是你实际传给 reject(..) 的那个Promise/thenable，⽽不是其底层的⽴即值。  

### Promise.all([ .. ])  

```javascript
// request(..)是⼀个Promise-aware Ajax⼯具
// 就像我们在本章前⾯定义的⼀样

var p1 = request( "http://some.url.1/" );
var p2 = request( "http://some.url.2/" );

Promise.all( [p1,p2] )
.then( function(msgs){
    // 这⾥，p1和p2完成并把它们的消息传⼊
    return request(
        "http://some.url.3/?v=" + msgs.join(",")
    );
} )
.then( function(msg){
	console.log( msg );
} );  
```

- Promise.all([ .. ]) 需要⼀个参数，是⼀个数组，通常由Promise 实例组成。
- 从 Promise.all([ .. ]) 调⽤返回的promise 会收到⼀个完成消息（代码⽚段中的 msg ）。这是⼀个由所有传⼊ promise 的完成消息组成的数组，与指定的顺序⼀致（与完成顺序⽆关）。  
- 从 Promise.all([ .. ]) 返回的主 promise 在且仅在所有的成员promise 都完成后才会完成。如果这些 promise 中有任何⼀个被拒绝的话，主 Promise.all([ .. ]) promise 就会⽴即被拒绝，并丢弃来⾃其他所有 promise 的全部结果。  

### Promise.race([ .. ])  

有时候你会想只响应“第⼀个跨过终点线的 Promise”，⽽抛弃其他 Promise。  称为竞态。  

超时竞赛
我们之前看到过这个例⼦，其展⽰了如何使⽤ Promise.race([ ..]) 表达 Promise 超时模式：

```javascript
// foo()是⼀个⽀持Promise的函数
// 前⾯定义的timeoutPromise(..)返回⼀个promise，
// 这个promise会在指定延时之后拒绝

// 为foo()设定超时
Promise.race( [
    foo(), // 启动foo()
    timeoutPromise( 3000 ) // 给它3秒钟
] )
.then(
    function(){
        // foo(..)按时完成！
    },
    function(err){
        // 要么foo()被拒绝，要么只是没能够按时完成，
        // 因此要查看err了解具体原因
    }
);  
```

### all([ .. ]) 和 race([ .. ]) 的变体  

- none([ .. ])
  这个模式类似于 all([ .. ]) ，不过完成和拒绝的情况互换了。所有的 Promise 都要被拒绝，即拒绝转化为完成值，反之亦然。
- any([ .. ])
  这个模式与 all([ .. ]) 类似，但是会忽略拒绝，所以只需要完成⼀个⽽不是全部。
- first([ .. ])
  这个模式类似于与 any([ .. ]) 的竞争，即只要第⼀个Promise 完成，它就会忽略后续的任何拒绝和完成。
- last([ .. ])
  这个模式类似于 first([ .. ]) ，但却是只有最后⼀个完成胜出。  

## Promise API 概述  

### new Promise(..) 构造器  

```javascript
var p = new Promise( function(resolve,reject){
// resolve(..)⽤于决议/完成这个promise
// reject(..)⽤于拒绝这个promise
} );  
```



### Promise.resolve(..) 和Promise.reject(..)  

- Promise.resolve(..) 常⽤于创建⼀个已完成的 Promise  
- Promise.resolve(..) 也会展开 thenable 值  

- Promise.reject(..) 直接返回

### then(..) 和 catch(..)  

- then(..) 和 catch(..) 也会创建并返回⼀个新的 promise，
- 这个promise 可以⽤于实现 Promise 链式流程控制。
- 如果完成或拒绝回调中抛出异常，返回的 promise 是被拒绝的。
- 如果任意⼀个回调返回⾮ Promise、⾮ thenable 的⽴即值，这个值会被⽤作返回 promise的完成值。
- 如果完成处理函数返回⼀个 promise 或 thenable，那么这个值会被展开，并作为返回 promise 的决议值。  

## Promise 局限性  

### 顺序错误处理  

Promise 链中的错误很容易被⽆意中默默忽略掉。  

没有外部⽅法可以⽤于观察可能发⽣的错误。  

构建了⼀个没有错误处理函数的 Promise 链，链中任何地⽅的任何错误都会在链中⼀直传播下去，直到被查看（通过在某个步骤注册拒绝处理函数）。  

### 单⼀值  

Promise 只能有⼀个完成值或⼀个拒绝理由。在简单的例⼦中，这不是什么问题，但是在更复杂的场景中，你可能就会发现这是⼀种局限了。  

#### 分裂值

有时候你可以把这⼀点当作提⽰你可以 / 应该把问题分解为两个或更多 Promise 的信号。
设想你有⼀个⼯具 foo(..) ，它可以异步产⽣两个值（x 和 y ）：

```javascript
function getY(x) {
    return new Promise( function(resolve,reject){
        setTimeout( function(){
        	resolve( (3 * x) - 1 );
        }, 100 );
    } );
}
function foo(bar,baz) {
    var x = bar * baz;
    
    return getY( x )
    .then( function(y){
        // 把两个值封装到容器中
        return [x,y];
    } );
}

foo( 10, 20 )
.then( function(msgs){
    var x = msgs[0];
    var y = msgs[1];
    console.log( x, y ); // 200 599
} );  
```

我们重新组织⼀下 foo(..) 返回的内容，这样就不再需要把 x和 y 封装到⼀个数组值中以通过 promise 传输。  

```javascript
function foo(bar,baz) {
    var x = bar * baz;
    
    // 返回两个promise
    return [
        Promise.resolve( x ),
        getY( x )
    ];
}

Promise.all(
	foo( 10, 20 )
)
.then( function(msgs){
    var x = msgs[0];
    var y = msgs[1];
    console.log( x, y );
} );  
```

这种细节放在 foo(..) 内部抽象，这样更整洁也更灵活。这⾥使⽤了 Promise.all([ .. ]) ，当然，这并不是唯⼀的选择。  

#### 展开 / 传递参数  

var x = .. 和 var y = .. 赋值操作仍然是⿇烦的开销。我们可以在辅助⼯具中采⽤某种函数技巧（感谢 Reginald Braithwaite，推特：@raganwald）：

```javascript
function spread(fn) {
	return Function.apply.bind( fn, null );
}
Promise.all(
	foo( 10, 20 )
)
.then(
    spread( function(x,y){
    	console.log( x, y ); // 200 599
    } )
) 
```

这样会好⼀点！当然，你可以把这个函数戏法在线化，以避免额外的辅助⼯具：

```javascript
Promise.all(
	foo( 10, 20 )
)
.then( Function.apply.bind(
    function(x,y){
        console.log( x, y ); // 200 599
    },
    null
) );
```

这些技巧可能很灵巧，但 ES6 给出了⼀个更好的答案：解构。数组解构赋值形式看起来是这样的：

```javascript
Promise.all(
	foo( 10, 20 )
)  
.then( function(msgs){
    var [x,y] = msgs;
    console.log( x, y ); // 200 599
} );
```

不过最好的是，ES6 提供了数组参数解构形式：

```javascript
Promise.all(
	foo( 10, 20 )
)
.then( function([x,y]){
	console.log( x, y ); // 200 599
} );
```

现在，我们符合了“每个 Promise ⼀个值”的理念，并且⼜将重复样板代码量保持在了最⼩！  

#### 单决议  

Promise 只能被决议⼀次（完成或拒绝）。在许多异步情况中，你只会获取⼀个值⼀次，所以这可以⼯作良好。  

场景：你可能要启动⼀系列异步步骤以响应某种可能多次发⽣的激励（就像是事件），⽐如按钮点击。  

这样可能不会按照你的期望⼯作：

```javascript
// click(..)把"click"事件绑定到⼀个DOM元素
// request(..)是前⾯定义的⽀持Promise的Ajax
var p = new Promise( function(resolve,reject){
	click( "#mybtn", resolve );
} );
p.then( function(evt){
    var btnID = evt.currentTarget.id;
    return request( "http://some.url.1/?id=" + btnID );
} )
.then( function(text){
    console.log( text );
} );  
```

因此，你可能需要转化这个范例，为每个事件的发⽣创建⼀整个新的Promise 链：

```javascript
click( "#mybtn", function(evt){
    var btnID = evt.currentTarget.id;
    
    request( "http://some.url.1/?id=" + btnID )
    .then( function(text){
        console.log( text );
    } );
} );  
```



- 需要在事件处理函数中定义整个 Promise 链，这很丑陋  
- 这个设计在某种程度上破坏了关注点与功能分离 （SoC）的思想。  
- 很可能想要把事件处理函数的定义和对事件的响应（那个Promise 链）的定义放在代码中的不同位置。如果没有辅助机制的话，在这种模式下很难这样实现。  

#### 惯性  

考虑如下的类似基于回调的场景：

```javascript
function foo(x,y,cb) {
    ajax(
        "http://some.url.1/?x=" + x + "&y=" + y,
        cb
    );
}
foo( 11, 31, function(err,text) {
    if (err) {
    	console.error( err );
    }
    else {
    	console.log( text );
    }
} );
```

能够很快明显看出要把这段基于回调的代码转化为基于 Promise 的代
码应该从哪些步骤开始吗？

绝对需要⼀个⽀持 Promise ⽽不是基于回调的 Ajax
⼯具，可以称之为 request(..) 。  

```javascript
// polyfill安全的guard检查
if (!Promise.wrap) {
    Promise.wrap = function(fn) {
        return function() {  
            var args = [].slice.call( arguments );
            return new Promise( function(resolve,reject){
                fn.apply(
                    null,
                    args.concat( function(err,v){
                        if (err) {
                        	reject( err );
                        }
                        else {
                        	resolve( v );
                        }
                    } )
                );
            } );
        };
    };
}  
```

它接受⼀个函数，这个函数
需要⼀个 error-first ⻛格的回调作为第⼀个参数，并返回⼀个新的函
数。返回的函数⾃动创建⼀个 Promise 并返回，并替换回调，连接到
Promise 完成或拒绝。  

与其花费太多时间解释这个 Promise.wrap(..) 辅助⼯具的⼯作原
理，还不如直接看看其使⽤⽅式：

```javascript
var request = Promise.wrap( ajax );

request( "http://some.url.1/" )
.then( .. )
..  
```

Promise.wrap(..) 并不产出 Promise。它产出的是⼀个将产⽣Promise 的函数。在某种意义上，产⽣ Promise 的函数可以看作是⼀个 Promise ⼯⼚。我提议将其命名为“promisory”（“Promise”+“factory”）。

把需要回调的函数封装为⽀持 Promise 的函数，这个动作有时被称为“提升”或“Promise ⼯⼚化”。但是，对于得到的结果函数来说，除了“被提升函数”似乎就没有什么标准术语可称呼了。所以我更喜欢“promisory”这个词，我认为它的描述更准确。  

promisory 并不是编造的。它是⼀个真实的单词，意思是包含或传输⼀个 promise。这正是这些函数所做的，所以这个术语与其意义匹配得很完美。

于是，Promise.wrap(ajax) 产⽣了⼀个 ajax(..) promisory，我们称之为 request(..) 。这个 promisory 为 Ajax 响应⽣成Promise。

如果所有函数都已经是 promisory，我们就不需要⾃⼰构造了，所以这个额外的步骤有点可惜。但⾄少这个封装模式（通常）是重复的，所以我们可以像前⾯展⽰的那样把它放⼊ Promise.wrap(..) 辅助⼯具，以帮助我们的 promise 编码。

所以，回到前⾯的例⼦，我们需要为 ajax(..) 和 foo(..) 都构造⼀个 promisory：

```javascript
// 为ajax(..)构造⼀个promisory
var request = Promise.wrap( ajax );
// 重构foo(..)，但使其外部成为基于外部回调的，
// 与⽬前代码的其他部分保持通⽤
// ——只在内部使⽤ request(..)的promise
function foo(x,y,cb) {  
    request(
    	"http://some.url.1/?x=" + x + "&y=" + y
    )
    .then(
        function fulfilled(text){
        	cb( null, text );
        },
        cb
    );
}
// 现在，为了这段代码的⽬的，为foo(..)构造⼀个 promisory
var betterFoo = Promise.wrap( foo );
// 并使⽤这个promisory
betterFoo( 11, 31 )
.then(
    function fulfilled(text){
        console.log( text );
    },
    function rejected(err){
        console.error( err );
    }
);
```


当 然，尽管我们在重构 foo(..) 以使⽤新的 request(..)promisory，但是也可以使 foo(..) 本⾝成为⼀个 promisory，⽽不是保持基于回调的形式并需要构建和使⽤后续的 betterFoo(..)promisory。这个决策就取决于 foo(..) 是否需要保持与代码库中其他部分兼容的基于回调的形式。  

#### ⽆法取消的 Promise  

⼀旦创建了⼀个 Promise 并为其注册了完成和 / 或拒绝处理函数，如果出现某种情况使得这个任务悬⽽未决的话，你也没有办法从外部停⽌它的进程。  

考虑前⾯的 Promise 超时场景：

```javascript
var p = foo( 42 );
Promise.race( [
    p,
    timeoutPromise( 3000 )
] )
.then(
    doSomething,
    handleError
);
p.then( function(){
	// 即使在超时的情况下也会发⽣ :(
} );
```

这个“超时”相对于 promise p 是外部的，所以 p 本⾝还会继续运⾏，
这⼀点可能并不是我们所期望的。  

⼀种选择是侵⼊式地定义你⾃⼰的决议回调：

```javascript
var OK = true;
var p = foo( 42 );

Promise.race( [
    p,
    timeoutPromise( 3000 )
    .catch( function(err){
        OK = false;
        throw err;
    } )
] )
.then(
    doSomething,
    handleError
);

p.then( function(){
    if (OK) {
    	// 只在没有超时情况下才会发⽣ :)
    }
} );  
```

这很丑陋。它可以⼯作，但是离理想实现还差很远。⼀般来说，应避免这样的情况。  
