# JS前端基础理论-----⽣成器

生成器是一种返回迭代器的函数，**通过function关键字后的星号(*)来表示，函数中会用到新的关键字yield。星号可以紧挨着function关键字，也可以在中间添加一个空格**

看似同步的异步流程控制表达⻛格。使这种⻛格成为可能的“魔法”就是 ES6 ⽣成器 （generator）  

yield/next(..) 这⼀对不只是⼀种控制机制，实际上也是⼀种双向消息传递机制。yield .. 表达式本质上是暂停下来等待某个值，接下来的 next(..) 调⽤会向被暂停的 yield 表达式传回⼀个值（或者是隐式的 undefined ）。

在异步控制流程⽅⾯，⽣成器的关键优点是：⽣成器内部的代码是以⾃然的同步 / 顺序⽅式表达任务的⼀系列步骤。其技巧在于，我们把可能的异步隐藏在了关键字 yield 的后⾯，把异步移动到控制⽣成器的迭代器的代码部分。

换句话说，⽣成器为异步代码保持了顺序、同步、阻塞的代码模式，这使得⼤脑可以更⾃然地追踪代码，解决了基于回调的异步的两个关键缺陷之⼀。  

## 打破完整运⾏  

下⾯是实现这样的合作式并发的 ES6 代码：

```javascript
var x = 1;
function *foo() {
    x++;
    yield; // 暂停！
    console.log( "x:", x );
}
function bar() {
	x++;
}  
```

> 很可能你看到的其他多数 JavaScript ⽂档和代码中的⽣成器声明格式都是 function* foo() { .. } ，⽽不是我这⾥使⽤的 function *foo() { .. } ：唯⼀区别是 * 位置的⻛格不同。这两种形式在功能和语法上都是等同的，还有⼀种是function*foo(){ .. } （没有空格）也⼀样。两种⻛格，各有优缺，但总体上我⽐较喜欢 function *foo.. 的形式，因为这样在使⽤ *foo() 来引⽤⽣成器的时候就会⽐较⼀致。如果只⽤ foo() 的形式，你就不会清楚知道我指的是⽣成器还是常规函数。这完全是⼀个⻛格偏好问题。  

现在，我们要如何运⾏前⾯的代码⽚段，使得 bar() 在 *foo() 内部的 yield 处执⾏呢？

```javascript
// 构造⼀个迭代器it来控制这个⽣成器
var it = foo();
// 这⾥启动foo()！
it.next();
x; // 2
bar();
x; // 3
it.next(); // x: 3  
```

在解释 ES6 ⽣成器的不同机制和语法之前，我们先来看看运⾏过程。

1. it = foo() 运算并没有执⾏⽣成器 *foo() ，⽽只是构造了⼀个迭代器 （iterator），这个迭代器会控制它的执⾏。后⾯会介绍迭代器。
2. 第⼀个 it.next() 启动了⽣成器 *foo() ，并运⾏了 *foo()第⼀⾏的 x++ 。
3. *foo() 在 yield 语句处暂停，在这⼀点上第⼀个 it.next()调⽤结束。此时 *foo() 仍在运⾏并且是活跃的，但处于暂停状态。
4. 我们查看 x 的值，此时为 2 。
5. 我们调⽤ bar() ，它通过 x++ 再次递增 x 。
6. 我们再次查看 x 的值，此时为 3 。
7. 最后的 it.next() 调⽤从暂停处恢复了⽣成器 *foo() 的执⾏，并运⾏ console.log(..) 语句，这条语句使⽤当前 x 的值 3 。显然，foo() 启动了，但是没有完整运⾏，它在 yield 处暂停了。后⾯恢复了 foo() 并让它运⾏到结束，但这不是必需的。

因此，⽣成器就是⼀类特殊的函数，可以⼀次或多次启动和停⽌，并不⼀定⾮得要完成。

### 输⼊和输出  

1. 迭代消息传递  

   ⽣成器甚⾄提供了更强⼤更引⼈注⽬的内建消息输⼊输出能⼒，通过 yield 和 next(..) 实现。 

    考虑：

   ```javascript
   function *foo(x) {
       var y = x * (yield);
       return y;
   }
   
   var it = foo( 6 );
   
   // 启动foo(..)
   it.next();
   
   var res = it.next( 7 );
   res.value; // 42
   ```

   ⾸先，传⼊ 6 作为参数 x 。然后调⽤ it.next() ，这会启动*foo(..) 。

   在 *foo(..) 内部，开始执⾏语句 var y = x .. ，但随后就遇到了⼀个 yield 表达式。它就会在这⼀点上暂停 *foo(..) （在赋值语句中间！），并在本质上要求调⽤代码为 yield 表达式提供⼀个结果值。接下来，调⽤ it.next( 7 ) ，这⼀句把值 7 传回作为被暂停的 yield 表达式的结果。

   所以，这时赋值语句实际上就是 var y = 6 * 7 。现在，return y返回值 42 作为调⽤ it.next( 7 ) 的结果。 

    

   - **迷惑性：**

   根据你的视⾓不同，yield 和 next(..) 调⽤有⼀个不匹配。⼀般来说，需要的 next(..) 调⽤要⽐ yield 语句多⼀个，前⾯的代码⽚段有⼀个 yield 和两个 next(..) 调⽤。为什么会有这个不匹配？

   因为第⼀个 next(..) 总是启动⼀个⽣成器，并运⾏到第⼀个 yield处。不过，是第⼆个 next(..) 调⽤完成第⼀个被暂停的 yield 表达式，第三个 next(..) 调⽤完成第⼆个 yield ，以此类推。  

   

2. 两个问题的故事

   

   - 消息是双向传递的

   ——yield.. 作为⼀个表达式可以发出消息响应 next(..) 调⽤，next(..) 也可以向暂停的 yield 表达式发送值。考虑下⾯这段稍稍调整过的代码：

   ```javascript
   function *foo(x) {
       var y = x * (yield "Hello"); // <-- yield⼀个值！
       return y;
   }
   
   var it = foo( 6 );
   
   var res = it.next(); // 第⼀个next()，并不传⼊任何东⻄
   res.value; // "Hello"
   
   res = it.next( 7 ); // 向等待的yield传⼊7
   res.value; // 42
   ```

   yield .. 和 next(..) 这⼀对组合起来，在⽣成器的执⾏过程中构成了⼀个双向消息传递系统。

   那么只看下⾯这⼀段迭代器 代码：

   ```javascript
   var res = it.next(); // 第⼀个next()，并不传⼊任何东⻄
   res.value; // "Hello"
   res = it.next( 7 ); // 向等待的yield传⼊7
   res.value; // 42
   ```

   > 我们并没有向第⼀个 next() 调⽤发送值，这是有意为之。只有暂停的 yield 才能接受这样⼀个通过 next(..) 传递的值，⽽在⽣成器的起始处我们调⽤第⼀个 next() 时，还没有暂停的 yield 来接受这样⼀个值。规范和所有兼容浏览器都会默默丢弃传递给第⼀个 next() 的任何东⻄。传值过去仍然不是⼀个好思路，因为你创建了沉默的⽆效代码，这会让⼈迷惑。因此，启动⽣成器时⼀定要⽤不带参数的 next() 。

   第⼀个 next() 调⽤（没有参数的）基本上就是在提出⼀个问题：“⽣成器 *foo(..) 要给我的下⼀个值是什么”。谁来回答这个问题呢？第⼀个 yield "hello" 表达式。

   看⻅了吗？这⾥没有不匹配。

   根据你认为提出问题的是谁，yield 和 next(..) 调⽤之间要么有不匹配，要么没有。

   但是，稍等！与 yield 语句的数量相⽐，还是多出了⼀个额外的next() 。所以，最后⼀个 it.next(7) 调⽤再次提出了这样的问题：⽣成器将要产⽣的下⼀个值是什么。但是，再没有 yield 语句来回答这个问题了，是不是？那么谁来回答呢？

   return 语句回答这个问题！

   如果你的⽣成器中没有 return 的话——在⽣成器中和在普通函数中⼀样，return 当然不是必需的——总有⼀个假定的 / 隐式的return; （也就是 return undefined; ），它会在默认情况下回答最后的 it.next(7) 调⽤提出的问题。

   这样的提问和回答是⾮常强⼤的：通过 yield 和 next(..) 建⽴的双向消息传递。但⽬前还不清楚这些机制是如何与异步流程控制联系到⼀起的。会清楚的！  

### 多个迭代器  

同⼀个⽣成器的多个实例可以同时运⾏，它们甚⾄可以彼此交互：  

```javascript
function *foo() {
    var x = yield 2;
    z++;
    var y = yield (x * z);
    console.log( x, y, z );
}
var z = 1;

var it1 = foo();
var it2 = foo();

var val1 = it1.next().value; // 2 <-- yield 2
var val2 = it2.next().value; // 2 <-- yield 2

val1 = it1.next( val2 * 10 ).value; // 40 <-- x:20, z:2
val2 = it2.next( val1 * 5 ).value; // 600 <-- x:200, z:3

it1.next( val2 / 2 ); 	// y:300
						// 20 300 3
it2.next( val1 / 4 ); 	// y:10
						// 200 10 3
```

同⼀个⽣成器的多个实例并发运⾏的最常⽤处并不是这样
的交互，⽽是⽣成器在没有输⼊的情况下，可能从某个独⽴连接
的资源产⽣⾃⼰的值。下⼀节中我们会详细介绍值产⽣。

我们简单梳理⼀下执⾏流程。

1. *foo() 的两个实例同时启动，两个 next() 分别从 yield 2 语句得到值 2 。
2. val2 * 10 也就是 2 * 10 ，发送到第⼀个⽣成器实例 it1 ，因此 x 得到值 20 。z 从 1 增加到 2 ，然后 20 * 2 通过 yield 发出，将 val1 设置为 40 。
3. val1 * 5 也就是 40 * 5 ，发送到第⼆个⽣成器实例 it2 ，因此 x 得到值 200 。z 再次从 2 递增到 3 ，然后 200 * 3 通过 yield发出，将 val2 设置为 600 。
4. val2 / 2 也就是 600 / 2 ，发送到第⼀个⽣成器实例 it1 ，因此 y 得到值 300 ，然后打印出 x y z 的值分别是 20 300 3 。
5. val1 / 4 也就是 40 / 4 ，发送到第⼆个⽣成器实例 it2 ，因此 y 得到值 10 ，然后打印出 x y z 的值分别为 200 10 3 。

在脑海中运⾏⼀遍这个例⼦很有趣。理清楚了吗？

#### 交替执⾏  

```javascript
var a = 1;
var b = 2;
function *foo() {
    a++;
    yield;
    b = b * a;
    a = (yield b) + 3;
}
function *bar() {
    b--;
    yield;
    a = (yield 8) + b;
    b = a * (yield 2);
}  
```

构建⼀个名为 step(..) 的辅助函数，⽤于控制迭代器 ：  

```javascript
function step(gen) {
    var it = gen();
    var last;
    return function() {
        // 不管yield出来的是什么，下⼀次都把它原样传回去！
        last = it.next( last ).value;
    };
}  
```

step(..) 初始化了⼀个⽣成器来创建迭代器 it ，然后返回⼀个函
数，这个函数被调⽤的时候会将迭代器 向前迭代⼀步。另外，前⾯的
yield 发出的值会在下⼀步发送回去。于是，yield 8 就是 8 ，⽽
yield b 就是 b （yield 发出时的值）。  

```javascript
// 确保重新设置a和b
a = 1;
b = 2;
var s1 = step( foo );
var s2 = step( bar );

// ⾸次运⾏*foo()
s1();
s1();
s1();

// 现在运⾏*bar()
s2();
s2();
s2();
s2();
console.log( a, b ); // 11 22  
```

最后的结果是 11 和 22 ，和第 1 章中的版本⼀样。现在交替执⾏顺
序，看看 a 和 b 的值是如何改变的：

```javascript
// 确保重新设置a和b
a = 1;
b = 2;
var s1 = step( foo );
var s2 = step( bar );
s2(); // b--;
s2(); // yield 8
s1(); // a++;
s2(); // a = 8 + b;

// yield 2
s1(); // b = b * a;

// yield b
s1(); // a = b + 3;
s2(); // b = a * 2;
```

在告诉你结果之前，你能推断出前⾯的程序运⾏后 a 和 b 的值吗？不
要作弊！

```javascript
console.log( a, b ); // 12 18
```

## ⽣成器产⽣值    

### ⽣产者与迭代器-----非生成器使用迭代

迭代器 是⼀个定义良好的接⼝，⽤于从⼀个⽣产者⼀步步得到⼀系列值  

可以为我们的数字序列⽣成器实现标准的迭代器 接⼝：

```javascript
var something = (function(){
    var nextVal;
    return {
        // for..of循环需要
        [Symbol.iterator]: function(){ return this; },
        // 标准迭代器接⼝⽅法
        next: function(){
            if (nextVal === undefined) {
            	nextVal = 1;
            }
            else {
            	nextVal = (3 * nextVal) + 6;
            }
            return { done:false, value:nextVal };
        }
    };
})();
something.next().value; // 1
something.next().value; // 9
something.next().value; // 33
something.next().value; // 105
```

> 我们将在 4.2.2 节解释为什么在这段代码中需要[Symbol.iterator]: .. 这⼀部分。
>
> - 从语法上说，这涉及了两个 ES6 特性。这在对象术语定义中是指，指定⼀个表达式并⽤这个表达式的结果作为属性的名称。
> - 另外，Symbol.iterator 是ES6 预定义的特殊 Symbol 值之⼀。  

ES6 还新增了⼀个 for..of 循环，这意味着可以通过原⽣循环语法⾃动迭代标准迭代器 ：

```javascript
for (var v of something) {
    console.log( v );
    // 不要死循环！
    if (v > 500) {
    	break;
    }
}
// 1 9 33 105 321 969  
```

因为我们的迭代器 something 总是返回 done:false ，因此这个 for..of 循环将永远运⾏下去，这也就是为什么我们要在⾥⾯放⼀个 break 条件。迭代器永不结束是完全没问题的，但是也有⼀些情况下，迭代器 会在有限的值集合上运⾏，并最终返回 done:true 。

for..of 循环在每次迭代中⾃动调⽤ next() ，它不会向 next()传⼊任何值，并且会在接收到 done:true 之后⾃动停⽌。这对于在⼀组数据上循环很⽅便。  

当然，也可以⼿⼯在迭代器上循环，调⽤ next() 并检查done:true 条件来确定何时停⽌循环：

```javascript
for (
var ret;
(ret = something.next()) && !ret.done;
) {
    console.log( ret.value );
    // 不要死循环！
    if (ret.value > 500) {
    	break;
    }
}
// 1 9 33 105 321 969
```

这种**⼿⼯ for ⽅法当然要⽐ ES6 的 for..of 循环语法丑陋，但其优点是，这样就可以在需要时向 next() 传递值**。  

### iterable  

something 对象叫作迭代器 ，因为它的接⼝中有⼀个next() ⽅法。⽽与其紧密相关的⼀个术语是 iterable（可迭代），即指⼀个包含可以在其值上迭代的迭代器的对象。  

从 ES6 开始，从⼀个 iterable 中提取迭代器的⽅法是：iterable 必须⽀持⼀个函数，其名称是专门的 ES6 符号值 Symbol.iterator 。调⽤这个函数时，它会返回⼀个迭代器。通常每次调⽤会返回⼀个全新的迭代器，虽然这⼀点并不是必须的。

前⾯代码⽚段中的 a 就是⼀个 iterable。for..of 循环⾃动调⽤它的Symbol.iterator 函数来构建⼀个迭代器。我们当然也可以⼿⼯调⽤这个函数，然后使⽤它返回的迭代器：  

```javascript
var a = [1,3,5,7,9];
var it = a[Symbol.iterator]();
it.next().value; // 1
it.next().value; // 3
it.next().value; // 5
..
```

前⾯的代码中列出了定义的 something ，你可能已经注意到了这⼀⾏：

```javascript
[Symbol.iterator]: function(){ return this; }
```

这段有点令⼈疑惑的代码是在将 something 的值（迭代器something 的接⼝）也构建成为⼀个 iterable。现在它既是iterable，也是迭代器。然后我们把 something 传给 for..of 循环：

```javascript
for (var v of something) {
	..
}
```

for..of 循环期望 something 是 iterable，于是它寻找并调⽤它的Symbol.iterator 函数。我们将这个函数定义为就是简单的return this ，也就是把⾃⾝返回，⽽ for..of 循环并不知情。  

### ⽣成器迭代器  

严格说来，⽣成器本⾝并不是 iterable，尽管⾮常类似——当你执⾏⼀个⽣成器，就得到了⼀个迭代器：  

可以通过⽣成器实现前⾯的这个 something ⽆限数字序列⽣产者，
类似这样：

```javascript
function *something() {
    var nextVal;
    
    while (true) {
        if (nextVal === undefined) {
        	nextVal = 1;
        }
        else {
        	nextVal = (3 * nextVal) + 6;
        }
        yield nextVal;
    }
}  
```

> 通常在实际的 JavaScript 程序中使⽤ while..true 循环是⾮常糟糕的主意，⾄少如果其中没有 break 或 return 的话是这样，因为它有可能会同步地⽆限循环，并阻塞和锁住浏览器UI。
>
> 但是，*如果在⽣成器中有 yield 的话，使⽤这样的循环就完全没有问题。*因为⽣成器会在每次迭代中暂停，通过 yield 返回到主程序或事件循环队列中。简单地说就是：“⽣成器把while..true 带回了 JavaScript 编程的世界！  

⽣成器会在每个 yield 处暂停，函数 *something() 的状态（作⽤域）会被保持，即意味着不需要闭包在调⽤之间保持变量状态。  

现在，可以通过 for..of 循环使⽤我们雕琢过的新的*something() ⽣成器。你可以看到，其⼯作⽅式基本是相同的：

```javascript
for (var v of something()) {
    console.log( v );
    // 不要死循环！
    if (v > 500) {
    	break;
    }
}
// 1 9 33 105 321 969
```

但是，不要忽略了这段 for (var v of something()) .. ！我们并不是像前⾯的例⼦那样把 something 当作⼀个值来引⽤，⽽是调⽤了 *something() ⽣成器以得到它的迭代器供 for..of 循环使⽤。  

如果认真思考的话，你也许会从这段⽣成器与循环的交互中提出两个
问题。

- 为什么不能⽤ for (var v of something) .. ？因为这⾥的 something 是⽣成器，并不是 iterable。我们需要调⽤something() 来构造⼀个⽣产者供 for..of 循环迭代。
- something() 调⽤产⽣⼀个迭代器，但 for..of 循环需要的是⼀个 iterable，对吧？是的。⽣成器的迭代器也有⼀个Symbol.iterator 函数，基本上这个函数做的就是 returnthis ，和我们前⾯定义的 iterable something ⼀样。换句话说，⽣成器的迭代器也是⼀个 iterable ！  

#### 停⽌⽣成器  

在前⾯的例⼦中，看起来似乎 *something() ⽣成器的迭代器实例在循环中的 break 调⽤之后就永远留在了挂起状态。

其实有⼀个隐藏的特性会帮助你管理此事。for..of 循环的“异常结束”（也就是“提前终⽌”），通常由 break 、return 或者未捕获异常引起，会向⽣成器的迭代器发送⼀个信号使其终⽌。  

如果在⽣成器内有 try..finally 语句，它将总是运⾏，即使⽣成器已经外部结束。如果需要清理资源的话（数据库连接等），这⼀点⾮常有⽤：

```javascript
function *something() {
    try {
        var nextVal;
        while (true) {
            if (nextVal === undefined) {
            	nextVal = 1;
            }
            else {
            	nextVal = (3 * nextVal) + 6;
            }
            yield nextVal;
        }
    }
    // 清理⼦句
    finally {
    	console.log( "cleaning up!" );
    }
} 
```

之前的例⼦中，for..of 循环内的 break 会触发 finally 语句。但是，也可以在外部通过 return(..) ⼿⼯终⽌⽣成器的迭代器实例：

```javascript
var it = something();
for (var v of it) {
    console.log( v );
    // 不要死循环！
    if (v > 500) {
        console.log(
            // 完成⽣成器的迭代器
            it.return( "Hello World" ).value
        );
        // 这⾥不需要break
    }
}
// 1 9 33 105 321 969
// 清理！
// Hello World
```

调⽤ it.return(..) 之后，它会⽴即终⽌⽣成器，这当然会运⾏finally 语句。另外，它还会把返回的 value 设置为传⼊return(..) 的内 容，这也就是 "Hello World" 被传出去的过程。现在我们也不需要包含 break 语句了，因为⽣成器的迭代器已经被设置为 done:true ，所以 for..of 循环会在下⼀个迭代终⽌  

## 异步迭代⽣成器  

通过⽣成器来表达同样的任务流程控制，可以这样实现：

```javascript
function foo(x,y) {
    ajax(
        "http://some.url.1/?x=" + x + "&y=" + y,
        function(err,data){
            if (err) {
                // 向*main()抛出⼀个错误
                it.throw( err );
            }
            else {
                // ⽤收到的data恢复*main()
                it.next( data );
            }
        }
    );
}
function *main() {
    try {
        var text = yield foo( 11, 31 );
        console.log( text );
    }
    catch (err) {
    	console.error( err );
    }
}
var it = main();
// 这⾥启动！
it.next();
```

这⾥并不是在消息传递的意义上使⽤ yield ，⽽只是将其⽤于流程控制实现暂停 / 阻塞。实际上，它还是会有消息传递，但只是⽣成器恢复运⾏之后的单向消息传递。  

把异步作为实现细节抽象了出去，使得我们可以以同步顺序的形式追踪流程控制：“发出⼀个 Ajax 请求，等它完成之后打印出响应结果。”并且，当然，我们只在这个流程控制中表达了两个步骤，⽽这种表达能⼒是可以⽆限扩展的，以便我们⽆论需要多少步骤都可以表达。  

### 同步错误处理  

```javascript
try {
    var text = yield foo( 11, 31 );
    console.log( text );
}
catch (err) {
	console.error( err );
}  
```

我们已经看到 yield 是如何让赋值语句暂停来等待 foo(..) 完成，使得响应完成后可以被赋给 text 。精彩的部分在于 yield 暂停也使得⽣成器能够捕获错误。通过这段前⾯列出的代码把错误抛出到⽣成器中：

```javascript
if (err) {
    // 向*main()抛出⼀个错误
    it.throw( err );
} 
```

⽣成器 yield 暂停的特性意味着我们不仅能够从异步函数调⽤得到看似同步的返回值，还可以同步捕获来⾃这些异步函数调⽤的错误！  

可以捕获通过 throw(..) 抛⼊⽣成器的同⼀个错误，基本上也就是给⽣成器⼀个处理它的机会；如果没有处理的话，迭代器代码就必须处理：

```javascript
function *main() {
    var x = yield "Hello World";
    // 永远不会到达这⾥
    console.log( x );
}

var it = main();
it.next();

try {
    // *main()会处理这个错误吗？看看吧！
    it.throw( "Oops" );
}
catch (err) {
    // 不⾏，没有处理！
    console.error( err ); // Oops
} 
```

在异步代码中实现看似同步的错误处理（通过 try..catch ）在可读性和合理性⽅⾯都是⼀个巨⼤的进步。 

## ⽣成器 +Promise  

ES6 中最完美的世界就是⽣成器（看似同步的异步代码）和 Promise（可信任可组合）的结合。 但如何实现呢？  

让我们来试⼀下！⾸先，把⽀持 Promise 的 foo(..) 和⽣成器*main() 放在⼀起：

```javascript
function foo(x,y) {
    return request(
    	"http://some.url.1/?x=" + x + "&y=" + y
    );
}
function *main() {
    try {
        var text = yield foo( 11, 31 );
        console.log( text );
    }
    catch (err) {
    	console.error( err );
    }
}  
```

来实现接收和连接 yield 出来的 promise，使它能够在决议之后恢复⽣成器。先从⼿⼯实现开始：

```javascript
var it = main();
var p = it.next().value;
// 等待promise p决议
p.then(
    function(text){
    	it.next( text );
    },
    function(err){
    	it.throw( err );
    }
);  
```

这段代码看起来应该和我们前⾯⼿⼯组合通过 error-first 回调控制的⽣成器⾮常类似。除了没有 if (err) { it.throw.. ，promise已经为我们分离了完成（成功）和拒绝（失败），否则的话，迭代器控制是完全⼀样的。  

优点：

- 我们利⽤了已知 *main() 中只有⼀个需要⽀持Promise 的步骤这⼀事实。

缺点：

- 不希望每个⽣成器⼿⼯编写不同的 Promise 链！如果有⼀种⽅法可以实现重复（即循环）迭代控制，每次会⽣成⼀个 Promise，等其决议后再继续
- 还有，如果在 it.next(..) 调⽤过程中⽣成器（有意或⽆意）抛出⼀个错误会怎样呢？是应该退出呢，还是应该捕获这个错误并发送回去呢？类似地，如果通过 it.throw(..) 把⼀个 Promise 拒绝抛⼊⽣成器中，但它却没有受到处理就被直接抛回了呢？  

### ⽀持 Promise 的 Generator Runner  

```javascript
// 在此感谢Benjamin Gruenbaum （@benjamingr on GitHub）的巨⼤改进！
function run(gen) {
    var args = [].slice.call( arguments, 1), it;// 在当前上下⽂中初始化⽣成器
    it = gen.apply( this, args );
    // 返回⼀个promise⽤于⽣成器完成
    return Promise.resolve()
    .then( function handleNext(value){
        // 对下⼀个yield出的值运⾏
        var next = it.next( value );
        return (function handleResult(next){
            // ⽣成器运⾏完毕了吗？
            if (next.done) {
            	return next.value;
            }
            // 否则继续运⾏
            else {
                return Promise.resolve( next.value )
                .then(
                    // 成功就恢复异步循环，把决议的值发回⽣成器
                    handleNext,
                    // 如果value是被拒绝的 promise，
                    // 就把错误传回⽣成器进⾏出错处理
                    function handleErr(err) {
                        return Promise.resolve(
                        	it.throw( err )
                        )
                        .then( handleResult );
                    }
                );
            }
        })(next);
    } );
}
```

如何在运⾏ Ajax 的例⼦中使⽤ run(..) 和 *main() 呢？

```javascript
function *main() {
	// ..
}
run( main );
```

就是这样！这种运⾏ run(..) 的⽅式，它会⾃动异步运⾏你传给它的⽣成器，直到结束。  

#### ES7：async 与 await ?  

```javascript
function foo(x,y) {
    return request(
    	"http://some.url.1/?x=" + x + "&y=" + y
    );
}
async function main() {
    try {
        var text = await foo( 11, 31 );
        console.log( text );
    }
    catch (err) {
    	console.error( err );
    }
}
main();
```

- 可以看到，这⾥没有通过 run(..) 调⽤（意味着不需要库⼯具！）来触发和驱动 main() ，它只是被当作⼀个普通函数调⽤。
- main() 也不再被声明为⽣成器函数了，它现在是⼀类新的函数：async 函数。
- 我们不再 yield 出 Promise，⽽是⽤ await等待它决议。  

### ⽣成器中的 Promise 并发  

场景：你需要从两个不同的来源获取数据，然后把响应
组合在⼀起以形成第三个请求，最终把最后⼀条响应打印出来。  

最简单的⽅法：

```javascript
function *foo() {
    // 让两个请求"并⾏"
    var p1 = request( "http://some.url.1" );
    var p2 = request( "http://some.url.2" );
    // 等待两个promise都决议
    var r1 = yield p1;
    var r2 = yield p2;
    var r3 = yield request(
    	"http://some.url.3/?v=" + r1 + "," + r2
    );
    console.log( r3 );
}
// 使⽤前⾯定义的⼯具run(..)
run( foo );  
```

p1 和 p2 都会并发执⾏，⽆论完成顺序如何，两者都
要全部完成，然后才会发出 r3 = yield request.. Ajax 请求。  

通过 Promise.all([ .. ]) ⼯具实现的 gate 模式相
同。  

```javascript
function *foo() {
    // 让两个请求"并⾏"，并等待两个promise都决议
    var results = yield Promise.all( [
        request( "http://some.url.1" ),
        request( "http://some.url.2" )
    ] );
    var r1 = results[0];
    var r2 = results[1];
    var r3 = yield request(
        "http://some.url.3/?v=" + r1 + "," + r2
    );
    console.log( r3 );
}
// 使⽤前⾯定义的⼯具run(..)
run( foo );  
```

#### 隐藏的 Promise  

可能是⼀个更简洁的⽅案：

```javascript
// 注：普通函数，不是⽣成器
function bar(url1,url2) {
    return Promise.all( [
        request( url1 ),
        request( url2 )
    ] );
}
function *foo() {
    // 隐藏bar(..)内部基于Promise的并发细节
    var results = yield bar(
        "http://some.url.1",
        "http://some.url.2"
    );
    var r1 = results[0];
    var r2 = results[1];
    var r3 = yield request(
    	"http://some.url.3/?v=" + r1 + "," + r2
    );
    console.log( r3 );
}
// 使⽤前⾯定义的⼯具run(..)
run( foo );  
```

⾮常有⽤的做法是：把你的 Promise 逻辑隐藏在⼀个只从⽣成器代码中调⽤的函数内部。⽐如：

```javascript
function bar() {
    Promise.all( [
    	baz( .. )
    	.then( .. ),
    	Promise.race( [ .. ] )
    ] )
    .then( .. )
}  
```

## ⽣成器委托  

### yield 委托  

yield 委托的具体语法是：yield * （注意多出来的
\* ）。在我们弄清它在前⾯的例⼦中的使⽤之前，先来看⼀个简单点
的场景：

```javascript
function *foo() {
    console.log( "*foo() starting" );
    yield 3;
    yield 4;
    console.log( "*foo() finished" );
}
function *bar() {
    yield 1;
    yield 2;
    yield *foo(); // yield委托！
    yield 5;
}
var it = bar();
it.next().value; // 1
it.next().value; // 2
it.next().value; // *foo()启动
// 3
it.next().value; // 4
it.next().value; // *foo()完成
// 5  
```

yield *foo() 委托是如何⼯作的呢？

⾸先，和我们以前看到的完全⼀样，调⽤ foo() 创建⼀个迭代器。然后 yield * 把迭代器实例控制（当前 *bar() ⽣成器的）委托给 /转移到了这另⼀个 *foo() 迭代器。

所以，前⾯两个 it.next() 调⽤控制的是 *bar() 。但当我们发出第三个 it.next() 调⽤时，*foo() 现在启动了，我们现在控制的是*foo() ⽽不是 *bar() 。这也是为什么这被称为委托：*bar() 把⾃⼰的迭代控制委托给了 *foo() 。

⼀旦 it 迭代器控制消耗了整个 *foo() 迭代器，it 就会⾃动转回控制 *bar() 。  

### 为什么⽤委托  

主要⽬的是代码组织，以达到与普通函数调⽤的对称。  

### 消息委托  

认真跟踪下⾯的通过 yield 委托实现的消息流出⼊：

```javascript
function *foo() {
    console.log( "inside *foo():", yield "B" );
    console.log( "inside *foo():", yield "C" );
    return "D";
}
function *bar() {
    console.log( "inside *bar():", yield "A" );
    // yield委托！
    console.log( "inside *bar():", yield *foo() );
    console.log( "inside *bar():", yield "E" );
    return "F";
}
var it = bar();
console.log( "outside:", it.next().value );
// outside: A
console.log( "outside:", it.next( 1 ).value );
// inside *bar(): 1
// outside: B
console.log( "outside:", it.next( 2 ).value );
// inside *foo(): 2
// outside: C
console.log( "outside:", it.next( 3 ).value );
// inside *foo(): 3
// inside *bar(): D
// outside: E
console.log( "outside:", it.next( 4 ).value );
// inside *bar(): 4
// outside: F
```

要特别注意 it.next(3) 调⽤之后的执⾏步骤。

1. 值 3 （通过 *bar() 内部的 yield 委托）传⼊等待的 *foo()内部的 yield "C" 表达式。
2. 然后 *foo() 调⽤ return "D" ，但是这个值并没有⼀直返回到外部的 it.next(3) 调⽤。
3. 取⽽代之的是，值 "D" 作为 *bar() 内部等待的 yield*foo()表达式的结果发出——这个 yield 委托本质上在所有的 *foo() 完成之前是暂停的。所以 "D" 成为 *bar() 内部的最后结果，并被打印出来。
4. yield "E" 在 *bar() 内部调⽤，值 "E" 作为 it.next(3) 调⽤的结果被 yield 发出。

从外层的迭代器（it ）⾓度来说，是控制最开始的⽣成器还是控制委托的那个，没有任何区别。  

yield 委托甚⾄并不要求必须转到另⼀个⽣成器，它可以转到⼀个⾮⽣成器的⼀般 iterable。⽐如：

```javascript
function *bar() {
console.log( "inside *bar():", yield "A" );
// yield委托给⾮⽣成器！
console.log( "inside *bar():", yield *[ "B", "C", "D" ] );
console.log( "inside *bar():", yield "E" );
return "F";
}
var it = bar();
console.log( "outside:", it.next().value );
// outside: A
console.log( "outside:", it.next( 1 ).value );
// inside *bar(): 1
// outside: B
console.log( "outside:", it.next( 2 ).value );
// outside: C
console.log( "outside:", it.next( 3 ).value );
// outside: D
console.log( "outside:", it.next( 4 ).value );
// inside *bar(): undefined
// outside: E
console.log( "outside:", it.next( 5 ).value );
// inside *bar(): 5
// outside: F
```

注意这个例⼦和之前那个例⼦在消息接收位置和报告位置上的区别。

最显著的是，默认的数组迭代器并不关⼼通过 next(..) 调⽤发送的任何消息，所以值 2 、3 和 4 根本就被忽略了。还有，因为迭代器没有显式的返回值（和前⾯使⽤的 *foo() 不同），所以 yield * 表达式完成后得到的是⼀个 undefined 。  

#### 异常也被委托！  

和 yield 委托透明地双向传递消息的⽅式⼀样，错误和异常也是双向传递的：

```javascript
function *foo() {
    try {
    	yield "B";
    }
    catch (err) {
    	console.log( "error caught inside *foo():", err );
    }
    yield "C";
    throw "D";
}
function *bar() {
    yield "A";
    try {
    	yield *foo();
    }
    catch (err) {
    	console.log( "error caught inside *bar():", err );
    }
    yield "E";
    yield *baz();
    // 注：不会到达这⾥！
    yield "G";
}
function *baz() {
	throw "F";
}
var it = bar();
console.log( "outside:", it.next().value );
// outside: A
console.log( "outside:", it.next( 1 ).value );
// outside: B
console.log( "outside:", it.throw( 2 ).value );
// error caught inside *foo(): 2
// outside: C
console.log( "outside:", it.next( 3 ).value );
// error caught inside *bar(): D
// outside: E
try {
	console.log( "outside:", it.next( 4 ).value );
}
catch (err) {
	console.log( "error caught outside:", err );
}
// error caught outside: F
```

这段代码中需要注意以下⼏点。

1. 调⽤ it.throw(2) 时，它会发送错误消息 2 到 *bar() ，它⼜将其委托给 *foo() ，后者捕获并处理它。然后，yield "C" 把"C" 发送回去作为 it.throw(2) 调⽤返回的 value 。
2. 接下来从 *foo() 内 throw 出来的值 "D" 传播到 *bar() ，这个函数捕获并处理它。然后 yield "E" 把 "E" 发送回去作为it.next(3) 调⽤返回的 value 。
3. 然后，从 *baz() throw 出来的异常并没有在 *bar() 内被捕获——所以 *baz() 和 *bar() 都被设置为完成状态。这段代码之后，就再也⽆法通过任何后续的 next(..) 调⽤得到值 "G" ，next(..)调⽤只会给 value 返回 undefined 。  

### 异步委托  

```javascript
function *foo() {
    var r2 = yield request( "http://some.url.2" );
    var r3 = yield request( "http://some.url.3/?v=" + r2 );
    return r3;
}
function *bar() {
    var r1 = yield request( "http://some.url.1" );
    var r3 = yield *foo();
    console.log( r3 );
}
run( bar );
```

### 递归委托  

使⽤ yield 委托实现异步的⽣成器递归 ，即⼀个yield 委托到它⾃⾝的⽣成器：

```javascript
function *foo(val) {
    if (val > 1) {
        // ⽣成器递归
        val = yield *foo( val - 1 );
    }
    return yield request( "http://some.url/?v=" + val );
}
function *bar() {
    var r1 = yield *foo( 3 );
    console.log( r1 );
}
run( bar );  
```

接下来的细节描述可能会⾮常复杂。  

1. run(bar) 启动⽣成器 *bar() 。
2. foo(3) 创建了⼀个 *foo(..) 的迭代器，并传⼊ 3 作为其参数val 。
3. 因为 3 > 1 ，所以 foo(2) 创建了另⼀个迭代器，并传⼊ 2 作为其参数 val 。
4. 因为 2 > 1 ，所以 foo(1) ⼜创建了⼀个新的迭代器，并传⼊ 1作为其参数 val 。
5. 因为 1 > 1 不成⽴，所以接下来以值 1 调⽤ request(..) ，并从这第⼀个 Ajax 调⽤得到⼀个 promise。
6. 这个 promise 通过 yield 传出，回到 *foo(2) ⽣成器实例。  
7. yield * 把这个 promise 传出回到 *foo(3) ⽣成器实例。另⼀个 yield * 把这个 promise 传出回到 *bar() ⽣成器实例。再有⼀个 yield * 把这个 promise 传出回到 run(..) ⼯具，这个⼯具会等待这个 promsie（第⼀个 Ajax 请求）的处理。
8. 这个 promise 决议后，它的完成消息会发送出来恢复 *bar() ；后者通过 yield * 转⼊ *foo(3) 实例；后者接着通过 yield * 转⼊ *foo(2) ⽣成器实例；后者再接着通过 yield * 转⼊ *foo(3)⽣成器实例内部的等待着的普通 yield 。
9. 第⼀个调⽤的 Ajax 响应现在⽴即从 *foo(3) ⽣成器实例中返回。这个实例把值作为 *foo(2) 实例中 yield * 表达式的结果返回，赋给它的局部变量 val 。
10. 在 *foo(2) 中，通过 request(..) 发送了第⼆个 Ajax 请求。它的 promise 通过 yield 发回给 *foo(1) 实例，然后通过yield * ⼀路传递到 run(..) （再次进⾏步骤 7）。这个 promise决议后，第⼆个 Ajax 响应⼀路传播回到 *foo(2) ⽣成器实例，赋给它的局部变量 val。
11. 最后，通过 request(..) 发出第三个 Ajax 请求，它的promise 传出到 run(..) ，然后它的决议值⼀路返回，然后return 返回到 *bar() 中等待的 yield * 表达式。  

## ⽣成器并发  

```javascript
// request(..)是⼀个⽀持Promise的Ajax⼯具
var res = [];
function *reqData(url) {
    var data = yield request( url );
    // 控制转移
    yield;
    res.push( data );
}
var it1 = reqData( "http://some.url.1" );
var it2 = reqData( "http://some.url.2" );
var p1 = it.next();
var p2 = it.next();
p1.then( function(data){
	it1.next( data );
} );
p2.then( function(data){
	it2.next( data );
} );
Promise.all( [p1,p2] )
.then( function(){
    it1.next();
    it2.next();
} );
```

好吧，这看起来好⼀点（尽管仍然是⼿⼯的！），因为现在*reqData(..) 的两个实例确实是并发运⾏了，⽽且（⾄少对于前⼀部分来说）是相互独⽴的。  

来设想⼀下使⽤⼀个称为runAll(..) 的⼯具：

```javascript
// request(..)是⼀个⽀持Promise的Ajax⼯具
var res = [];
runAll(
    function*(){
        var p1 = request( "http://some.url.1" );
        // 控制转移
        yield;
        res.push( yield p1 );
    },
    function*(){
        var p2 = request( "http://some.url.2" );
        // 控制转移
        yield;
        res.push( yield p2 );
    }
);  
```

以下是 runAll(..) 内部运⾏的过程。

1. 第⼀个⽣成器从第⼀个来⾃于 "http://some.url.1" 的 Ajax响应得到⼀个 promise，然后把控制 yield 回 runAll(..) ⼯具。
2. 第⼆个⽣成器运 ⾏，对于 "http://some.url.2" 实现同样的操 作，把控制 yield 回 runAll(..) ⼯具。
3. 第⼀个⽣成器恢复运⾏，通过 yield 传出其 promise p1 。在这种情况下，runAll(..) ⼯具所做的和我们之前的 run(..) ⼀样，因为它会等待这个 promise 决议，然后恢复同⼀个⽣成器（没有控制转移！）。p1 决议后，runAll(..) 使⽤这个决议值再次恢复第⼀个⽣成器，然后 res[0] 得到了⾃⼰的值。接着，在第⼀个⽣成器完成的时候，有⼀个隐式的控制转移。
4. 第⼆个⽣成器恢复运⾏，通过 yield 传出其 promise p2 ，并等待其决议。⼀旦决议，runAll(..) 就⽤这个值恢复第⼆个⽣成器，设置 res[1] 。  

```javascript
// request(..)是⼀个⽀持Promise的Ajax⼯具
runAll(
    function*(data){
        data.res = [];
        // 控制转移（以及消息传递）
        var url1 = yield "http://some.url.2";
        var p1 = request( url1 ); // "http://some.url.1"
        // 控制转移
        yield;
        data.res.push( yield p1 );
    },
    function*(data){
        // 控制转移（以及消息传递）
        var url2 = yield "http://some.url.1";
        var p2 = request( url2 ); // "http://some.url.2"
        // 控制转移
        yield;
        data.res.push( yield p2 );
    }
);
```

在这⼀⽅案中，实际上两个⽣成器不只是协调控制转移，还彼此通信，通过 data.res 和 yield 的消息来交换 url1 和 url2 的值。真是极其强⼤！  

通信顺序进程 （Communicating Sequential Processes，CSP）的更⾼级异步技术提供了⼀个概念基础。  

## 形实转换程序  

JavaScript 中的 thunk 是指⼀个⽤于调⽤另外⼀个函数的函数，没有任何参数。

换句话说，你⽤⼀个函数定义封装函数调⽤，包括需要的任何参数，来定义这个调⽤的执⾏，那么这个封装函数就是⼀个形实转换程序。之后在执⾏这个 thunk 时，最终就是调⽤了原始的函数。  

考虑：

```javascript
function thunkify(fn) {
    var args = [].slice.call( arguments, 1 );
    return function(cb) {
        args.push( cb );
        return fn.apply( null, args );
    };
}
var fooThunk = thunkify( foo, 3, 4 );
// 将来
fooThunk( function(sum) {
	console.log( sum ); // 7
} );  
```

前⾯ thunkify(..) 的实现接收 foo(..) 函数引⽤以及它需要的任意参数，并返回 thunk 本⾝（fooThunk(..) ）。但是，这并不是JavaScript 中使⽤ thunk 的典型⽅案。

典型的⽅法——如果不令⼈迷惑的话——并不是 thunkify(..) 构造thunk 本⾝，⽽是 thunkify(..) ⼯具产⽣⼀个⽣成 thunk 的函数。

考虑：

```javascript
function thunkify(fn) {
    return function() {
        var args = [].slice.call( arguments );
        return function(cb) {
            args.push( cb );
            return fn.apply( null, args );
        };
    };
} 
```

此处主要的区别在于多出来的 return function() { .. } 这⼀层。以下是⽤法上的区别：  

```javascript
var whatIsThis = thunkify( foo );
var fooThunk = whatIsThis( 3, 4 );
// 将来
fooThunk( function(sum) {
	console.log( sum ); // 7
} );  
```

## ES6 之前的⽣成器  

### ⼿⼯变换  

```javascript
function foo(url) {
    // 管理⽣成器状态
    var state;
    // ⽣成器变量范围声明
    var val;
    function process(v) {
        switch (state) {
            case 1:
                console.log( "requesting:", url );
                return request( url );
            case 2:
                val = v;
                console.log( val );
                return;
            case 3:
                var err = v;
                console.log( "Oops:", err );
                return false;
        }
    }
    // 构造并返回⼀个⽣成器
    return {
        next: function(v) {
            // 初始状态
            if (!state) {
                state = 1;
                return {
                    done: false,
                    value: process()
                };
            }
            // yield成功恢复
            else if (state == 1) {
                state = 2;
                return {
                    done: true,
                    value: process( v )
                };
            }
            // ⽣成器已经完成
            else {
                return {
                    done: true,
                    value: undefined
                };
            }
        },
        "throw": function(e) {
            // 唯⼀的显式错误处理在状态1
            if (state == 1) {
                state = 3;
                return {
                    done: true,
                    value: process( e )
                };
            }
            // 否则错误就不会处理，所以只把它抛回
            else {
                throw e;
            }
        }
    };
} 
```

这段代码是如何⼯作的呢？

1. (1) 对迭代器的 next() 的第⼀个调⽤会把⽣成器从未初始化状态转移到状态 1 ，然后调⽤ process() 来处理这个状态。
   request(..) 的返回值是对应 Ajax 响应的 promise，作为 value属性从 next() 调⽤返回。
2. (2) 如果 Ajax 请求成功，第⼆个 next(..) 调⽤应该发送 Ajax 响应值进来，这会把状态转移到状态 2 。再次调⽤ process(..) （这次包括传⼊的 Ajax 响应值），从 next(..) 返回的 value 属性将是undefined 。
3. (3) 然⽽，如果 Ajax 请求失败的话，就会使⽤错误调⽤ throw(..)，这会把状态从 1 转移到 3 （⽽⾮ 2 ）。再次调⽤ process(..)，这⼀次包含错误值。这个 case 返回 false ，被作为 throw(..)调⽤返回的 value 属性。

从外部来看（也就是说，只与迭代器交互），这个普通函数 foo(..)与⽣成器 *foo(..) 的⼯作⼏乎完全⼀样。所以我们已经成功地把ES6 ⽣成器转为了前 ES6 兼容代码  ！  

### ⾃动转换  

regenerator 就是这样的⼀个⼯具（http://facebook.github.io/regenerator/ ），出⾃ Facebook 的⼏个聪明⼈。  

如果使⽤ regenerator 来转换前⾯的⽣成器的话，以下是产⽣的代码
（本书写作之时）：

```javascript
// request(..)是⼀个⽀持Promise的Ajax⼯具
var foo = regeneratorRuntime.mark(function foo(url) {
    var val;
    return regeneratorRuntime.wrap(function foo$(context$1$0) {
        while (1) switch (context$1$0.prev = context$1$0.next) {
            case 0:
                context$1$0.prev = 0;
                console.log( "requesting:", url );
                context$1$0.next = 4;
                return request( url );
            case 4:
                val = context$1$0.sent;
                console.log( val );
                context$1$0.next = 12;
                break;
            case 8:
                context$1$0.prev = 8;
                context$1$0.t0 = context$1$0.catch(0);
                console.log("Oops:", context$1$0.t0);
                return context$1$0.abrupt("return", false);
            case 12:
            case "end":
            return context$1$0.stop();
        }
    }, foo, this, [[0, 8]]);
});  
```


