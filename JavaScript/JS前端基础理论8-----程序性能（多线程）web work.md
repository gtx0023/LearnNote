# JS前端基础理论-----程序性能（多线程）

这样你可以在每个线程上运⾏不同的程序。程序中每⼀个这样的独⽴的多线程部分被称为⼀个（Web）Worker。这种类型的并⾏化被称为任务并⾏   

## Web Worker  

这是浏览器（即宿主环境）的功能，实际上和 JavaScript 语⾔本⾝⼏乎没什么关系。也就是说，**JavaScript 当前并没有任何⽀持多线程执⾏的功能**。  

Worker 是有限量，系统能够决定可以创建多少个实际的线程/CPU/ 核⼼。  

从 JavaScript 主程序（或另⼀个 Worker）中，可以这样实例化⼀个Worker：

```javascript
var w1 = new Worker( "http://some.url.1/mycoolworker.js" );  
```

> 这种通过这样的 URL 创建的 Worker 称为专⽤Worker（Dedicated Worker）。除了提供⼀个指向外部⽂件的URL，你还可以通过提供⼀个 Blob URL（另外⼀个 HTML5 特性）创建⼀个在线 Worker（Inline Worker)，本质上就是⼀个存储在单个（⼆进制）值中的在线⽂件。不过，Blob 已经超出了我们这⾥的讨论范围。  

以下是如何侦听事件（其实就是固定的 "message" 事件）：

```javascript
w1.addEventListener( "message", function(evt){
	// evt.data
} );
```

也可以发送 "message" 事件给这个 Worker：

```javascript
w1.postMessage( "something cool to say" );
```

在这个 Worker 内部，收发消息是完全对称的：

```javascript
// "mycoolworker.js"
addEventListener( "message", function(evt){
	// evt.data  

} );
postMessage( "a really cool reply" );  
```



### Worker 环境  

Worker 内部是⽆法访问主程序的任何资源的。这是⼀个完全独⽴的线程。

1. 不能访问它的任何全局变量，
2. 不能访问⻚⾯的 DOM 或者其他资源。

可以

1. 执⾏⽹络操 作（Ajax、WebSockets）

2. 设定定时器。

3. 可以访问⼏个重要的全局变量

4. 功能的本地复本，包括 navigator 、location 、JSON 和 applicationCache 。

5. 你还可以通过 importScripts(..) 向 Worker 加载额外的JavaScript 脚本：

   ```javascript
   // 在Worker内部
   importScripts( "foo.js", "bar.js" );
   ```

   这些脚本加载是同步的。也就是说，importScripts(..) 调⽤会阻塞余下 Worker 的执⾏，直到⽂件加载和执⾏完成。 

 Web Worker 通常应⽤于哪些⽅⾯呢？

1. 处理密集型数学计算
2. ⼤数据集排序
3. 数据处理（压缩、⾳频分析、图像处理等）
4. ⾼流量⽹络通信  

### 数据传递  

需要在线程之间通过事件机制传递⼤量的信息，可能是双向的。  

在早期的 Worker 中，唯⼀的选择就是把所有数据序列化到⼀个字符串值中。

1. 双向序列化导致的速度损失
2. 数据需要被复制，这意味着两倍的内存使⽤（及其引起的垃圾收集⽅⾯的波动）。



现在已经有了⼀些更好的选择。  

#### 结构化克隆算法   

结构化克隆算法是[由HTML5规范定义](http://www.w3.org/html/wg/drafts/html/master/infrastructure.html#safe-passing-of-structured-data)的用于复制复杂JavaScript对象的算法。通过来自 [Workers](https://developer.mozilla.org/en-US/docs/Web/API/Worker)的 `postMessage() `或使用 [IndexedDB](https://developer.mozilla.org/en-US/docs/Glossary/IndexedDB) 存储对象时在内部使用。它通过递归输入对象来构建克隆，同时保持先前访问过的引用的映射，以避免无限遍历循环。

把这个对象复制到另⼀边。这个算法⾮常⾼级，甚⾄可以处理要复制的对象有循环引⽤的情况。这样就不⽤付出 to-string 和 from-string 的性能损失了，但是这种⽅案还是要使⽤双倍的内存。  

还有⼀个更好的选择  

#### Transferable 对象  

对象所有权的转移，数据本⾝并没有移动。⼀旦你把对象传递到⼀个 Worker 中，在原来的位置上，它就变为空的或者是不可访问的，这样就消除了多线程编程作⽤域共享带来的混乱。当然，所有权传递是可以双向进⾏的。  

下⾯是如何使⽤ postMessage(..) 发送⼀个Transferable 对象：

```javascript
// ⽐如foo是⼀个Uint8Array
postMessage( foo.buffer, [ foo.buffer ] );
```

第⼀个参数是⼀个原始缓冲区，第⼆个是⼀个要传输的内容的列表。  

### 共享 Worker  

防⽌重复专⽤ Worker   

降低系统的资源使⽤  

- 限制socket ⽹络连接  
- 限制来⾃于同⼀客户端的连接数  

SharedWorker ，可通过下⾯的⽅式创建（只有 Firefox 和Chrome ⽀持这⼀功能）  

```javascript
var w1 = new SharedWorker( "http://some.url.1/mycoolworker.js" );
```

因为共享 Worker 可以与站点的多个程序实例或多个⻚⾯连接，所以
这个 Worker 需要通过某种⽅式来得知消息来⾃于哪个程序。这个唯
⼀标识符称为端⼝ （port），可以类⽐⽹络 socket 的端⼝。因此，
调⽤程序必须使⽤ Worker 的 port 对象⽤于通信：

```javascript
w1.port.addEventListener( "message", handleMessages );
// ..
w1.port.postMessage( "something cool" );
```

还有，端⼝连接必须要初始化，形式如下：

```javascript
w1.port.start();
```

在共享 Worker 内部，必须要处理额外的⼀个事件："connect" 。这
个事件为这个特定的连接提供了端⼝对象。保持多个连接独⽴的最简
单办法就是使⽤ port 上的闭包（参⻅本系列《你不知道的
JavaScript（上卷）》的“作⽤域和闭包”部分），就像下⾯的代码⼀
样，把这个链接上的事件侦听和传递定义在 "connect" 事件的处理
函数内部：

```javascript
// 在共享Worker内部
addEventListener( "connect", function(evt){
    // 这个连接分配的端⼝
    var port = evt.ports[0];
    port.addEventListener( "message", function(evt){
        // ..
        port.postMessage( .. );
        // ..
    } );
    // 初始化端⼝连接
    port.start();
} );  
```

在共享 Worker 内部，必须要处理额外的⼀个事件："connect" 。这个事件为这个特定的连接提供了端⼝对象。  

```javascript
// 在共享Worker内部
addEventListener( "connect", function(evt){
    // 这个连接分配的端⼝
    var port = evt.ports[0];
    port.addEventListener( "message", function(evt){
        // ..
        port.postMessage( .. );
        // ..
    } );
    // 初始化端⼝连接
    port.start();
} );
```

### 模拟 Web Worker  

## SIMD  

单指令多数据（SIMD）是⼀种数据并⾏ （data parallelism）⽅式，与 Web Worker 的任务并⾏ （task parallelism）相对，  

## asm.js  

asm.js（http://asmjs.org ）这个标签是指 JavaScript 语⾔中可以⾼度优化的⼀个⼦集。通过⼩⼼避免某些难以优化的机制和模式（垃圾收集、类型强制转换，等等），asm.js ⻛格的代码可以被JavaScript 引擎识别并进⾏特别激进的底层优化。  

### 如何使⽤ asm.js 优化  

⽐如：

```javascript
var a = 42;
// ..
var b = a;
```

在这个程序中，赋值 b = a 留下了变量类型⼆义性的后门。但它也可
以换⼀种⽅式，写成这样：

```javascript
var a = 42;
// ..
var b = a | 0;
```

此处我们使⽤了与 0 的 |（⼆进制或 ）运算，除了确保这个值是 32位整型之外，对于值没有任何效果。这样的代码在⼀般的 JavaScript引擎上都可以正常⼯作。⽽对⽀持 asm.js 的 JavaScript 引擎来说，这段代码就发出这样的信号，b 应该总是被当作 32 位整型来处理，这样就可以省略强制类型转换追踪。  

### asm.js 模块  

对⼀个 asm.js 模块来说，需要明确地导⼊⼀个严格规范的命名空间——规范将之称为 stdlib ，  

还需要声明⼀个堆 （heap）并将其传⼊。这个术语⽤于表⽰内存中⼀块保留的位置，变量可以直接使⽤⽽不需要额外的内存请求或释放之前使⽤的内存。  

```javascript
var heap = new ArrayBuffer( 0x10000 ); // 64k堆  
```

可以在模块内部使⽤堆缓冲区备份⼀个 64 位浮点值数组  

```javascript
var arr = new Float64Array( heap );
```

⽤⼀个简单快捷的 asm.js ⻛格模块例⼦来展⽰这些细节是如何结合到⼀起的。我们定义了⼀个 foo(..) 。它接收⼀个起始值（x ）和终⽌值（y ）整数构成⼀个范围，并计算这个范围内的值的所有相邻数的乘积，然后算出这些值的平均数：

```javascript
function fooASM(stdlib,foreign,heap) {
    "use asm";
    var arr = new stdlib.Int32Array( heap );
    
    function foo(x,y) {
        x = x | 0;
        y = y | 0;
        var i = 0;
        var p = 0;
        var sum = 0;
        var count = ((y|0) - (x|0)) | 0;
        
        // 计算所有的内部相邻数乘积
        for (i = x | 0;
        (i | 0) < (y | 0);
        p = (p + 8) | 0, i = (i + 1) | 0
        ) {
            // 存储结果
            arr[ p >> 3 ] = (i * (i + 1)) | 0;
        }
        
        // 计算所有中间值的平均数
        for (i = 0, p = 0;
        (i | 0) < (count | 0);
        p = (p + 8) | 0, i = (i + 1) | 0
        ) {
            sum = (sum + arr[ p >> 3 ]) | 0;
        }
        
        return +(sum / count);
    }
    
    return {
        foo: foo
    };
}

var heap = new ArrayBuffer( 0x1000 );
var foo = fooASM( window, null, heap ).foo;
foo( 10, 20 ); // 233  
```

第⼀个对 fooASM(..) 的调⽤建⽴了带堆分配的 asm.js 模块。结果是⼀个 foo(..) 函数，我们可以按照需要调⽤任意多次。这些foo(..) 调⽤应该被⽀持 asm.js 的 JavaScript 引擎专门优化。很重要的⼀点是，前⾯的代码完全是标准 JavaScript，在⾮ asm.js 引擎中也能正常⼯作（没有特殊优化）。  
