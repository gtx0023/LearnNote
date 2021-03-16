# JS前端基础理论-----asynquence 库  

## [getify](https://github.com/getify)/**[asynquence](https://github.com/getify/asynquence)**

| [.github](https://github.com/getify/asynquence/tree/master/.github) | [Adding sponsorship options](https://github.com/getify/asynquence/commit/88299338f4565c5a75987e0961601992ae54ad5d) | 2 years ago   |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------- |
| [contrib](https://github.com/getify/asynquence/tree/master/contrib) | [updating jsbin links](https://github.com/getify/asynquence/commit/cc5a9df76a841478c5e180ad97b9c89fdf142f28) | 12 months ago |
| [.editorconfig](https://github.com/getify/asynquence/blob/master/.editorconfig) | [tweaking editorconfig again](https://github.com/getify/asynquence/commit/b035980ff95c7826e1625a14fc03db1b04a6c1d3) | 3 years ago   |
| ...                                                          |                                                              |               |

asynquence 提供了⼀个独特的视⾓，⽽且它还是建⽴在单个基本抽象之上的：（异步）序列。  

asynquence 不再需要引⼊分别关注异步不同⽅⾯的两个或更多异步库，⽽是把它们统⼀为序列步骤的不同变体。

通过 asynquence 使得 Promise ⻛格语义的异步流程控制编程完成起来⾮常简单。

## 序列与抽象设计  

理解 asynquence  ：

⼀个任务的⼀系列步骤，不管各⾃是同步的还是异步的，都可以整合起来看成⼀个序列（sequence）。换句话说，⼀个序列代表了⼀个任务的容器，由完成这个任务的独⽴（可能是异步）的步骤组成。  

序列中的每个步骤在形式上通过⼀个 Promise（参⻅第 3 章）控制。也就是说，添加到序列中的每个步骤隐式地创建了⼀个 Promise 连接到之前序列的尾端。由于 Promise 的语义，序列中每个单个步骤的运⾏都是异步的，即使是同步完成这个步骤也是如此。  

序列通常是从⼀个步骤到⼀个步骤线性处理的，也就是说步骤2 要在步骤 1 完成之后开始，以此类推。  

可以从现有的序列分叉 （fork）出新的序列，这意味着主序列到达流程中的这个点上就会发⽣分叉。也可以通过各种⽅法合并序列，包括在流程中的特定点上让⼀个序列包含另⼀个序列。  

### 用Promise建⽴序列抽象

- 第⼀：Promise 链接更多是⼀个⼿⼯过程。

  这些种类的抽象⼯作量可并不⼩，在现有的 Promise 链当中实现也并不优美。但是，如果把你的思路抽象为序列，并把步骤当作对 Promise 的封装，那么这样的步骤封装就可以隐藏这些细节，节省你的精⼒  

- 第⼆：以序列中的步骤这样的视⾓来考量异步流程控制，这样就可以把每个单独步骤涉及的异步类型等细节抽象出去。在此之下，总是由 Promise 来控制着这个步骤，但是表⾯上，这个步骤看起来要么类似 continuation 回调（最简单的默认情况），要么类似真正的 Promise，要么就类似完整运⾏的⽣成器  

- 第三：序列很容易被改造，以适应不同的思考模式，⽐如基于事件、基于流、基于响应的编码。

  - asynquence 提供了⼀个模式，我称之为响应序列 （reactive sequence，后⾯会介绍），是RxJS（Reactive 扩展）中 reactive observable 思想的⼀个变体。它利⽤重复的事件每次启动⼀个新的序列实例。
  - Promise 只有⼀次，所以单独使⽤ Promise 对于表达重复的异步是很笨拙的。

## asynquence API  

### 步骤  

```javascript
ASQ(
    // 步骤1
    function(done){
        setTimeout( function(){
            done( "Hello" );
        }, 100 );
    },
    // 步骤2
    function(done,greeting) {
        setTimeout( function(){
            done( greeting + " World" );
        }, 100 );
    }
)
// 步骤3
.then( function(done,msg){
    setTimeout( function(){
    	done( msg.toUpperCase() );
    }, 100 );
} )
// 步骤4
.then( function(done,msg){
	console.log( msg ); // HELLO WORLD
} );
```

> 尽管 then(..) 和原⽣ Promise API 名称相同，但是这个then(..) 是不⼀样的。你可以向 then(..) 传递任意多个函数或值，其中每⼀个都会作为⼀个独⽴步骤。其中并不涉及两个回调的完成 / 拒绝语义。  

和 Promise 不同的⼀点是：

- 在 Promise 中，如果你要把⼀个Promise 链接到下⼀个，需要创建这个 Promise 并通过 then(..)完成回调函数返回这个 Promise；
- ⽽使⽤ asynquence，你需要做的就是调⽤ continuation 回调——我⼀直称之为 done() ，可选择性将完成消息传递给它作为参数。  

通过 then(..) 定义的每个步骤都被假定为异步的。如果你有⼀个同步的步骤，那你可以直接调⽤ done(..) ，也可以使⽤更简单的步骤辅助函数 val(..) 。

```javascript
// 步骤1（同步）
ASQ( function(done){
	done( "Hello" ); // ⼿⼯同步
} )
// 步骤2（同步）
.val( function(greeting){
	return greeting + " World";
} )
// 步骤3（异步）
.then( function(done,msg){
    setTimeout( function(){
    	done( msg.toUpperCase() );
    }, 100 );
} )
// 步骤4（同步）
.val( function(msg){
	console.log( msg );
} );
```

可以看到，通过 val(..) 调⽤的步骤并不接受 continuation 回调，因为这⼀部分已经为你假定了，结果就是参数列表没那么凌乱！如果要给下⼀个步骤发送消息的话，只需要使⽤ return 。

可以把 val(..) 看作⼀个表⽰同步的“只有值”的步骤，可以⽤于同步值运算、⽇志记录及其他类似的操作  

### 错误  

与 Promise 相⽐，asynquence ⼀个重要的不同之处就是错误处理。  

通过 Promise，链中每个独⽴的 Promise（步骤）都可以有⾃⼰独⽴的错误， ⼀个序列中任何⼀个步骤出错都会把整个序列抛⼊出错模式中，剩余的普通步骤会被忽略。  

编写代码来发送⼀个序列错误信号：

```javascript
var sq = ASQ( function(done){
    setTimeout( function(){
        // 为序列发送出错信号
        done.fail( "Oops" );
    }, 100 );
} )
.then( function(done){
	// 不会到达这⾥
} )
.or( function(err){
	console.log( err ); // Oops
} )
.then( function(done){
	// 也不会到达这⾥
} );
// 之后
sq.or( function(err){
	console.log( err ); // Oops
} );
```

asynquence 的错误处理和原⽣ Promise 还有⼀个⾮常重要的区别，就是默认状态下未处理异常的⾏为。  

没有注册拒绝处理函数的被拒绝 Promise 就会默默地持有（即吞掉）这个错误。  

⽽在 asynquence 中  

- 如果⼀个序列中发⽣了错误，并且此时没有注册错误处理函数，那这个错误就会被报告到控制台。  
- ⼀旦你针对某个序列注册了错误处理函数，这个序列就不会产⽣这样的报告，从⽽避免了重复的噪⾳。  

选择关闭错误报告：

```javascript
var sq1 = ASQ( function(done){
	doesnt.Exist(); // 将会向终端抛出异常
} );
var sq2 = ASQ( function(done){
	doesnt.Exist(); // 只抛出⼀个序列错误
} )
// 显式避免错误报告
.defer();
setTimeout( function(){
    sq1.or( function(err){
    	console.log( err ); // ReferenceError
    } );
    sq2.or( function(err){
        console.log( err ); // ReferenceError
    } );
}, 100 );
// ReferenceError (from sq1)
```

这种错误处理⽅式要好于 Promise 本⾝的那种⾏为，因为它是成功的坑，⽽不是失败陷阱  

### 并⾏步骤  

和原⽣的 Promise.all([..])直接对应。  

- gate(..) 中所有的步骤都成功完成，那么所有的成功消息都会传给下⼀个序列步骤。  
- 如果它们中有任何⼀个出错的话，整个序列就会⽴即进⼊出错状态。  

考虑：

```javascript
ASQ( function(done){
	setTimeout( done, 100 );
} )
.gate(
    function(done){
        setTimeout( function(){
            done( "Hello" );
        }, 100 );
    },
    function(done){
        setTimeout( function(){
            done( "World", "!" );
        }, 100 );
    }
)
.val( function(msg1,msg2){
    console.log( msg1 ); // Hello
    console.log( msg2 ); // [ "World", "!" ]
} );
```

出于展⽰说明的⽬的，我们把这个例⼦与原⽣ Promise 对⽐：

```javascript
new Promise( function(resolve,reject){
	setTimeout( resolve, 100 );
} )
.then( function(){
    return Promise.all( [
        new Promise( function(resolve,reject){
            setTimeout( function(){
                resolve( "Hello" );
            }, 100 );
        } ),
        new Promise( function(resolve,reject){
            setTimeout( function(){
                // 注：这⾥需要⼀个[ ]数组
                resolve( [ "World", "!" ] );
            }, 100 );
        } )
    ] );
} )
.then( function(msgs){
    console.log( msgs[0] ); // Hello
    console.log( msgs[1] ); // [ "World", "!" ]
} );
```

Promise ⽤来表达同样的异步流程控制的重复样板代码的开销要多得多。  

#### 步骤的变体

contrib 插件中提供了⼏个 asynquence 的 gate(..) 步骤类型的
变体，⾮常实⽤。

- any(..) 类似于 gate(..) ，除了只需要⼀个⼦步骤最终成功就可以使得整个序列前进。
- first(..) 类似于 any(..) ，除了只要有任何步骤成功，主序列就会前进（忽略来⾃其他步骤的后续结果）。
- race(..) （对应 Promise.race([..]) ）类似于first(..) ，除了只要任何步骤完成（成功或失败），主序列就会前进。
- last(..) 类似于 any(..) ，除了只有最后⼀个成功完成的步骤会将其消息发送给主序列。
- none(..) 是 gate(..) 相反：只有所有的⼦步骤失败（所有的步骤出错消息被当作成功消息发送，反过来也是如此），主序列才前进。

让我们先定义⼀些辅助函数，以便更清楚地进⾏说明：  

map(..) 与 gate(..) ⾮常相似，除了它是从⼀个数组⽽不是从独⽴的特定函数中取得初始值，⽽且这也是因为你定义了⼀个回调函数来处理每个值：

```javascript
function double(x,done) {
    setTimeout( function(){
    	done( x * 2 );
    }, 100 );
}
ASQ().map( [1,2,3], double )
.val( output ); // [2,4,6]  
```

map(..) 的参数（数组或回调）都可以从前⼀个步骤传⼊的消息中接收：

```javascript
function plusOne(x,done) {
    setTimeout( function(){
    	done( x + 1 );
    }, 100 );
}
ASQ( [1,2,3] )
.map( double ) // 消息[1,2,3]传⼊
.map( plusOne ) // 消息[2,4,6]传⼊
.val( output ); // [3,5,7]  
```

另外⼀个变体是 waterfall(..)   

⾸先执⾏步骤 1，然后步骤 1 的成功消息发送给步骤 2，然后两个成
功消息发送给步骤 3，然后三个成功消息都到达步骤 4，以此类推。
这样，在某种程度上，这些消息集结和层叠下来就构成了“瀑
布”（waterfall）。
考虑：

```javascript
function double(done) {
    var args = [].slice.call( arguments, 1 );
    console.log( args );
    setTimeout( function(){
    	done( args[args.length - 1] * 2 );
    }, 100 );
}
ASQ( 3 )
.waterfall(
    double, // [ 3 ]
    double, // [ 6 ]
    double, // [ 6, 12 ]
    double // [ 6, 12, 24 ]
)
.val( function(){
    var args = [].slice.call( arguments );
    console.log( args ); // [ 6, 12, 24, 48 ]
} );
```

如果“瀑布”中的任何⼀点出错，整个序列就会⽴即进⼊出错状态。  

#### 容错  

try(..) 会试验执⾏⼀个步骤，如果成功的话，这个序列就和通常⼀
样继续。如果这个步骤失败的话，失败就会被转化为⼀个成功消息，
格式化为 { catch: .. } 的形式，⽤出错消息填充：  

```javascript
ASQ()
.try( success1 )
.val( output ) // 1
.try( failure3 )
.val( output ) // { catch: 3 }
.or( function(err){
	// 永远不会到达这⾥
} );  
```

#### Promise ⻛格的步骤  

插件：

```javascript
ASQ( 21 )
.pThen( function(msg){
	return msg * 2;
} )
.pThen( output ) // 42
.pThen( function(){
    // 抛出异常
    doesnt.Exist();
} )
.pCatch( function(err){
    // 捕获异常（拒绝）
    console.log( err ); // ReferenceError
} )
.val( function(){
    // 主序列以成功状态返回，
    // 因为之前的异常被 pCatch(..)捕获了
} );
```

pThen(..) 和 pCatch(..) 是设计⽤来运⾏在序列中的，但其⾏为⽅式就像是在⼀个普通的 Promise 链中。  

#### 序列分叉  

在 asynquence ⾥可使⽤ fork() 实现同样的分叉：

```javascript
var sq = ASQ(..).then(..).then(..);
var sq2 = sq.fork();
// 分叉1
sq.then(..)..;
// 分叉2
sq2.then(..)..;  
```

#### 合并序列  

如果要实现 fork() 的逆操作，可以使⽤实例⽅法 seq(..) ，通过
把⼀个序列归⼊另⼀个序列来合并这两个序列：

```javascript
var sq = ASQ( function(done){
    setTimeout( function(){
        done( "Hello World" );
    }, 200 );
} );
ASQ( function(done){
	setTimeout( done, 100 );
} )
// 将sq序列纳⼊这个序列
.seq( sq )
.val( function(msg){
	console.log( msg ); // Hello World
} )
```

正如这⾥展⽰的，seq(..) 可以接受⼀个序列本⾝，或者⼀个函数。
如果它接收⼀个函数，那么就要求这个函数被调⽤时会返回⼀个序
列。因此，前⾯的代码可以这样实现：

```
// ..
.seq( function(){
	return sq;
} )
// ..
这个步骤也可以通过 pipe(..) 来完成：
// ..
.then( function(done){
    // 把sq加⼊done continuation回调
    sq.pipe( done );
} )
// ..
```

如果⼀个序列被包含，那么它的成功消息流和出错流都会输⼊进来。  

## 可迭代序列  

使⽤ Promise 的 capability extraction 反模式看起来类似如下：

```javascript
var ready;
var domready = new Promise( function(resolve,reject){
    // 提取resolve()功能
    ready = resolve;
} );
// ..
domready.then( function(){
	// DOM就绪！
} );
// ..
document.addEventListener( "DOMContentLoaded", ready );  
```

asynquence 提供了⼀个反转的序列类型，我称之为可迭代序列 ，它把控制能⼒外部化了（对于像 domready 这样的⽤例⾮常有⽤）：

```javascript
// 注： 这⾥的domready是⼀个控制这个序列的迭代器
var domready = ASQ.iterable();

// ..

domready.val( function(){
	// DOM就绪
} );

// ..
document.addEventListener( "DOMContentLoaded", domready.next );  
```

## 运⾏⽣成器  

asynquence 也内建有这样的⼯具，叫作
runner(..) 。
为了展⽰，我们⾸先构建⼀些辅助函数：

```javascript
function doublePr(x) {
    return new Promise( function(resolve,reject){
        setTimeout( function(){
        	resolve( x * 2 );
        }, 100 );
    } );
}
function doubleSeq(x) {
    return ASQ( function(done){
        setTimeout( function(){
        	done( x * 2)
        }, 100 );
    } );
}
```

现在，可以使⽤ runner(..) 作为序列中的⼀个步骤：

```javascript
ASQ( 10, 11 )
.runner( function*(token){
    var x = token.messages[0] + token.messages[1];
    // yield⼀个真正的promise
    x = yield doublePr( x );
    // yield⼀个序列
    x = yield doubleSeq( x );
    return x;
} )
.val( function(msg){
	console.log( msg ); // 84
} );
```

### 封装的⽣成器

你也可以创建⼀个⾃封装的⽣成器，也就是说，通过 ASQ.wrap(..)包装实现⼀个运⾏指定⽣成器的普通函数，完成后返回⼀个序列：

```javascript
var foo = ASQ.wrap( function*(token){
    var x = token.messages[0] + token.messages[1];
    // yield⼀个真正的promise
    x = yield doublePr( x );
    // yield⼀个序列
    x = yield doubleSeq( x );
    return x;
}, { gen: true } );
// ..
foo( 8, 9 )
.val( function(msg){
	console.log( msg ); // 68
} );
```

runner(..) 还可以实现更多很强⼤的功能，我们将在附录 B 中深⼊介绍。  

# ⾼级异步模式  

## 可迭代序列  

回忆⼀下：

```javascript
var domready = ASQ.iterable();
// ..
domready.val( function(){
// DOM就绪
} );
// ..
document.addEventListener( "DOMContentLoaded", domready.next );
```

现在，让我们把⼀个多步骤序列定义为可迭代序列：

```javascript
var steps = ASQ.iterable();
steps
.then( function STEP1(x){
	return x * 2;
} )
.steps( function STEP2(x){
	return x + 3;
} )
.steps( function STEP3(x){
	return x * 4;
} );
steps.next( 8 ).value; // 16
steps.next( 16 ).value; // 19
steps.next( 19 ).value; // 76
steps.next().done; // true
```

可以看到，可迭代序列是⼀个符合标准的迭代器（参⻅第 4 章）。因
此，可通过 ES6 的 for..of 循环迭代，就像⽣成器（或其他任何
iterable）⼀样：

```javascript
var steps = ASQ.iterable();

steps
.then( function STEP1(){ return 2; } )
.then( function STEP2(){ return 4; } )
.then( function STEP3(){ return 6; } )
.then( function STEP4(){ return 8; } )
.then( function STEP5(){ return 10; } );

for (var v of steps) {
	console.log( v );
}
// 2 4 6 8 10  
```

## 可迭代序列扩展  

```javascript
function double(x) {
    x *= 2;
    // 应该继续扩展吗？
    if (x < 500) {
        isq.then( double );
    }
    return x;
}
// 建⽴单步迭代序列
var isq = ASQ.iterable().then( double );
for (var v = 10, ret; (ret = isq.next( v )) && !ret.done;) {
    v = ret.value;
    console.log( v );
}  
```

考虑：

```javascript
var steps = ASQ.iterable()
.then( function STEP1(token){
    var url = token.messages[0].url;
    // 提供了额外的格式化步骤了吗？
    if (token.messages[0].format) {
    	steps.then( token.messages[0].format );
    }
    return request( url );
} )
.then( function STEP2(resp){
    // 向区列中添加⼀个Ajax请求吗？
    if (/x1/.test( resp )) {
        steps.then( function STEP5(text){
            return request(
            "http://some.url.4/?v=" + text
            );
        } );
    }
    return ASQ().gate(
        request( "http://some.url.2/?v=" + resp ),
        request( "http://some.url.3/?v=" + resp )
    );
} )
.then( function STEP3(r1,r2){ return r1 + r2; } );
```

你可以看到，在两个不同的位置处，我们有条件地使⽤steps.then(..) 扩展了 steps 。要运⾏这个可迭代序列 steps ，只需要通过 ASQ#runner(..) 把它链⼊我们的带有 asynquence 序列（这⾥称为 main ）的主程序流程：

```javascript
var main = ASQ( {
    url: "http://some.url.1",
    format: function STEP4(text){
    	return text.toUpperCase();
    }
} )
.runner( steps )
.val( function(msg){
	console.log( msg );
} );
```

可迭代序列 steps 的这⼀灵活性（有条件⾏为）可以⽤⽣成器表达吗？算是可以吧，但我们不得不以⼀种有点笨拙的⽅式重新安排这个逻辑：

```javascript
function *steps(token) {
    // 步骤1
    var resp = yield request( token.messages[0].url );
    // 步骤2
    var rvals = yield ASQ().gate(
        request( "http://some.url.2/?v=" + resp ),
        request( "http://some.url.3/?v=" + resp )
    );
    // 步骤3
    var text = rvals[0] + rvals[1];
    // 步骤4
    //提供了额外的格式化步骤了吗？
    if (token.messages[0].format) {
        text = yield token.messages[0].format( text );
    }
    // 步骤5
    // 需要向序列中再添加⼀个Ajax请求吗？
    if (/foobar/.test( resp )) {
        text = yield request(
            "http://some.url.4/?v=" + text
        );
    }
    return text;
}
// 注意：*steps()可以和前⾯的steps⼀样被同⼀个ASQ序列运⾏
```

除了已经确认的⽣成器的顺序、看似同步的语法的好处（参⻅第 4章），要模拟可扩展可迭代序列 steps 的动态特性，steps 的逻辑也需要以 *steps() ⽣成器形式重新安排。

⽽如果要通过 Promise 或序列来实现这个功能会怎样呢？你可以这么做：

```
var steps = something( .. )
.then( .. )
.then( function(..){
// ..
// 扩展链是吧？
steps = steps.then( .. );
// ..
})
.then( .. );
```

其中的问题捕捉起来⽐较微妙，但是很重要。所以，考虑要把我们的steps Promise 链链⼊主程序流程。这次使⽤ Promise 来表达，⽽不是 asynquence：

```javascript
var main = Promise.resolve( {
    url: "http://some.url.1",
    format: function STEP4(text){
        return text.toUpperCase();
    }
} )
.then( function(..){
	return steps; // hint！
} )
.val( function(msg){
	console.log( msg );
} );
```

现在能看出问题所在了吗？仔细观察！

序列步骤排序有⼀个竞态条件。在你返回 steps 的时候，steps 这时可能是之前定义的 Promise 链，也可能是现在通过 steps =steps.then(..) 调⽤指向扩展后的 Promise 链。根据执⾏顺序的不同，结果可能不同。

以下是两个可能的结果。

- 如果 steps 仍然是原来的 Promise 链，⼀旦之后它通过 steps= steps.then(..) 被“扩展”，在链结尾处扩展之后的promise 就不会被 main 流程考虑，因为它已经连到了 steps链。很遗憾，这就是及早求值 的局限性。
- 如果 steps 已经是扩展后的 Promise 链，它就会按预期⼯作，因为 main 连接的是扩展后的 promise。  

## ES7 Observable  

## 通信顺序进程  

### 消息传递  

CSP 的核⼼原则是独⽴的进程之间所有的通信和交互必须要通过正式的消息传递。可能与你预期的相反，CSP 消息传递是⽤同步动作来描述的，其中发送进程和接收进程都需要准备好消息才能传递。  

### asynquence CSP 模拟  

先来构造⼀个⽀持通道的 request(..) 版本：

```javascript
function request(url) {
    var ch = ASQ.csp.channel();
    ajax( url ).then( function(content){
        // putAsync(..)的put(..)的⼀个变异版本，这个版本
        // 可以在⽣成器之外使⽤。返回⼀个运算完毕promise。
        // 这⾥我们没有使⽤这个promise，但是如果当值被
        // take(..)之后我们需要得到通知的话，可以使⽤这个promise。
        ASQ.csp.putAsync( ch, content );
    } );
    return ch;
}  
```

现在考虑使⽤ asynquence ⻛格的 CSP 实现的并发 Ajax 的例⼦：

```javascript
ASQ()
.runner(
    ASQ.csp.go( function*(ch){
        yield ASQ.csp.put( ch, "http://some.url.2" );
        var url1 = yield ASQ.csp.take( ch );
        // "http://some.url.1"
        var res1 = yield ASQ.csp.take( request( url1 ) );
        yield ASQ.csp.put( ch, res1 );
    } ),
    ASQ.csp.go( function*(ch){
        var url2 = yield ASQ.csp.take( ch );
        // "http://some.url.2"
        
        yield ASQ.csp.put( ch, "http://some.url.1" );
        var res2 = yield ASQ.csp.take( request( url2 ) );
        var res1 = yield ASQ.csp.take( ch );
        
        // 把结果传递到下⼀个序列步骤
        ch.buffer_size = 2;
        ASQ.csp.put( ch, res1 );
        ASQ.csp.put( ch, res2 );
    } )
)
.val( function(res1,res2){
    // res1来⾃"http://some.url.1"
    // res2来⾃"http://some.url.2"
} );  
```

在两个 goroutine 之间交换 URL 字符串的消息传递过程是⾮常直接的。第⼀个 goroutine 构造⼀个到第⼀个 URL 的 Ajax 请求，响应放到通道 ch 中。第⼆个 goroutine 构造⼀个到第⼆个 URL 的 Ajax 请求，然后从通道 ch 拿到第⼀个响应 res1 。这时，两个响应 res1 和res2 便都已经完成就绪了。

如果在 goroutine 运⾏结束时，通道 ch 中还有任何剩下的值，那它们就会被传递到序列的下⼀个步骤。所以，要从最后的 goroutine 传出消息，可以通过 put(..) 将其放⼊ ch 中。如上所⽰，为了避免这些最后的 put(..) 阻塞，我们通过将 ch 的 buffer_size 设置为 2（默认：0 ）⽽将 ch 切换为缓冲模式。  
