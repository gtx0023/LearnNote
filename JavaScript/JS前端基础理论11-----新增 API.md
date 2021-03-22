# JS前端基础理论-----新增 API  

## Array  

### Array.of(..)

Array(..) 如果只传入一个参数，并且这个参数是数字的话，那么不会构造一个值为这个数字的单个元素的数组，而是构造一个空数组，其length 属性为这个数字。这个动作会产生不幸又诡异的“空槽”行为，这是 JavaScript 数
组广为人所诟病的一点。
Array.of(..) 取代了 Array(..) 成为数组的推荐函数形式构造器，因为 Array.of(..) 并没有这个特殊的单个数字参数的问题。考虑：  

```javascript
var a = Array( 3 );
a.length; // 3
a[0]; // undefined

var b = Array.of( 3 );
b.length; // 1
b[0]; // 3

var c = Array.of( 1, 2, 3 );
c.length; // 3
c; // [1,2,3]  
```

### Array.from(..)  

普遍的需求就是把它们转换为真正的数组  

#### before

```javascript
// 类数组对象
var arrLike = {
    length: 3,
    0: "foo",
    1: "bar"
};
var arr = Array.prototype.slice.call( arrLike );
```

另外一个常见的任务是使用 slice(..) 来复制产生一个真正的数组：

```javascript
var arr2 = arr.slice(); 
```

####  After

新的 ES6 Array.from(..) 方法都是更好理解  

var arr = Array.from( arrLike );
var arrCopy = Array.from( arr );  

- 检查第一个参数是否为 iterable（参见 3.1 节），如果是的话，就使用迭代器来产生值并“复制”进入返回的数组。因为真正的数组有一个这些值之上的迭代器，所以会自动使用这个迭代器。
- 而如果你把类数组对象作为第一个参数传给 Array.from(..)，它的行为方式和 slice()（没有参数）或者 apply(..) 是一样的，就是简单地按照数字命名的属性从 0 开始直到length 值在这些值上循环  

#### 避免空槽位  

Array.from(..) 永远不会产生空槽位。  

在 ES6 之前，如果你想要产生一个初始化为某个长度，在每个槽位上都是真正的undefined 值（不是空槽位！）的数组，不得不做额外的工作：

```javascript
var a = Array( 4 );
// 4个空槽位！
var b = Array.apply( null, { length: 4 } );
// 4个undefined值
```

而现在 Array.from(..) 使其简单了很多：

```javascript
var c = Array.from( { length: 4 } );
// 4个undefined值  
```

#### 映射  

如果提供了的话，第二个参数是一个映射回调（和一般的 Array#map(..) 所期望的几乎一样），这个函数会被调用，来把来自于源的每个值映射 / 转换到返回值。考虑：  

```javascript
var arrLike = {
    length: 4,
    2: "foo"
};
Array.from( arrLike, function mapper(val,idx){
    if (typeof val == "string") {
        return val.toUpperCase();
    }
    else {
        return idx;
    }
} );
// [ 0, 1, "FOO", 3 ]  
```

### 创建数组和子类型  

一般来说，默认的行为方式很可能就是需要的，但就像我们在第 3 章中介绍的，必要的话也可以覆盖它：

```javascript
class MyCoolArray extends Array {
// 强制species为父构造器
static get [Symbol.species]() { return Array; }
}
var x = new MyCoolArray( 1, 2, 3 );
x.slice( 1 ) instanceof MyCoolArray; // false
x.slice( 1 ) instanceof Array; // true
```

需要注意的是， @@species 设置只用于像 slice(..) 这样的原型方法。 of(..) 和 from(..)  不会使用它；它们都只使用 this 绑定（由使用的构造器来构造其引用）。

考虑：

```javascript
class MyCoolArray extends Array {
// 强制species为父构造器
static get [Symbol.species]() { return Array; }
}
var x = new MyCoolArray( 1, 2, 3 );
MyCoolArray.from( x ) instanceof MyCoolArray; // true
MyCoolArray.of( [2, 3] ) instanceof MyCoolArray; // true  
```

### copyWithin(..)  

从一个数组中复制一部分到同一个数组的另一个位置，覆盖这个位置所有原来的值。  

### fill(..)  

用指定值完全（或部分）填充已存在的数组：  

```javascript
var a = Array( 4 ).fill( undefined );
a;
// [undefined,undefined,undefined,undefined]
```

fill(..) 可选地接收参数 start 和 end，它们指定了数组要填充的子集位置，比如：

```javascript
var a = [ null, null, null, null ].fill( 42, 1, 3 );
a; // [null,42,42,null]  
```

### find(..)  

数组中搜索一个值的最常用方法一直是 indexOf(..) 方法，这个方法返回找到值的索引，如果没有找到就返回 -1：  

从 ES5 以来，控制匹配逻辑的最常用变通技术是使用 some(..) 方法。  

```javascript
var a = [1,2,3,4,5];
    a.some( function matcher(v){
    return v == "2";
} ); // true  

a.some( function matcher(v){
	return v == 7;
} ); // false  
```

缺点是如果找到匹配的值的时候，只能得到匹配的 true/false 指示，而无法得到真正的匹配值本身  

ES6 的 find(..) 解决了这个问题。基本上它和 some(..) 的工作方式一样，除了一旦回调返回 true/ 真值，会返回实际的数组值：

```javascript
var a = [1,2,3,4,5];
a.find( function matcher(v){
return v == "2";
} ); // 2
a.find( function matcher(v){
return v == 7; // undefined
});
```

通过自定义 matcher(..) 函数也可以支持比较像对象这样的复杂值：

```javascript
var points = [
    { x: 10, y: 20 },
    { x: 20, y: 30 },
    { x: 30, y: 40 },
    { x: 40, y: 50 },
    { x: 50, y: 60 }
];
points.find( function matcher(point) {
    return (
        point.x % 3 == 0 &&
        point.y % 4 == 0
    );
} ); // { x: 30, y: 40 }  
```

### 原型方法  findIndex(..) ？

```javascript
var points = [
    { x: 10, y: 20 },
    { x: 20, y: 30 },
    { x: 30, y: 40 },
    { x: 40, y: 50 },
    { x: 50, y: 60 }
];

points.findIndex( function matcher(point) {
    return (
        point.x % 3 == 0 &&
        point.y % 4 == 0
    );
} ); // 2

points.findIndex( function matcher(point) {
    return (
        point.x % 6 == 0 &&
        point.y % 7 == 0
    );
} ); // -1  
```

### entries()、 values()、 keys()  

从这个意义上说，它是一个集合。考虑：

```javascript
var a = [1,2,3];
[...a.values()]; // [1,2,3]
[...a.keys()]; // [0,1,2]
[...a.entries()]; // [ [0,1], [1,2], [2,3] ]
[...a[Symbol.iterator]()]; // [1,2,3]  
```

我们将展示 Array.from(..) 如何把数组中的空槽位看作值为undefined 的槽位。这实际上是因为在底层数组迭代器是这样工作的：

```javascript
var a = [];
a.length = 3;
a[1] = 2;
[...a.values()]; // [undefined,2,undefined]
[...a.keys()]; // [0,1,2]
[...a.entries()]; // [ [0,undefined], [1,2], [2,undefined] ]  
```

## Object  

### Object.is(..)  

静态函数 Object.is(..) 执行比 === 比较更严格的值比较  

Object.is(..) 调用底层 SameValue 算法（ES6 规范， 7.2.9 节）。 SameValue 算法基本上和=== 严格相等比较算法一样（ES6 规范， 7.2.13 节），但有两个重要的区别。

考虑：

```javascript
var x = NaN, y = 0, z = -0;
x === x; // false
y === z; // true
Object.is( x, x ); // true
Object.is( y, z ); // false
```

你应该继续使用 === 进行严格相等比较；不应该把 Object.is(..) 当作这个运算符的替代。但是，如果需要严格识别 NaN 或者 -0 值，那么应该选择 Object.is(..)。  

### Object.getOwnPropertySymbols(..)  

**Symbol** 很 可 能 会 成 为 对 象 最 常 用 的 特 殊（ 元 ） 属 性。 所 以 引 入 了 工 具 Object.getOwnPropertySymbols(..)，它直接从对象上取得所有的符号属性：

```javascript
var o = {
    foo: 42,
    [ Symbol( "bar" ) ]: "hello world",
    baz: true
};
Object.getOwnPropertySymbols( o ); // [ Symbol(bar) ]  
```

### Object.setPrototypeOf(..)  

考虑：

```javascript
var o1 = {
	foo() { console.log( "foo" ); }
};
var o2 = {
	// .. o2的定义 ..
};
Object.setPrototypeOf( o2, o1 );
// 委托给o1.foo()
o2.foo(); // foo
```

也可以：

```javascript
var o1 = {
	foo() { console.log( "foo" ); }
};
var o2 = Object.setPrototypeOf( {
	// .. o2的定义 ..
}, o1 );
// 委托给o1.foo()
o2.foo(); // foo  
```

### Object.assign(..)  

- 第一个参数是 target，
- 其他传入的参数都是源，
- 它们将按照列出的顺序依次被处理。
- 对于每个源来说，通过简单 = 赋值被复制。
  - 它的可枚举
  - 自己拥有的（也就是不是“继承来的”）键值，
  - 符号  
- Object.assign(..) 返回目标对象。  

考虑这个对象设定：

```javascript
var target = {},
o1 = { a: 1 }, o2 = { b: 2 },
o3 = { c: 3 }, o4 = { d: 4 };
// 设定只读属性
Object.defineProperty( o3, "e", {
    value: 5,
    enumerable: true,
    writable: false,
    configurable: false
} );
// 设定不可枚举属性
Object.defineProperty( o3, "f", {
    value: 6,
    enumerable: false
} );
o3[ Symbol( "g" ) ] = 7;
// 设定不可枚举符号
Object.defineProperty( o3, Symbol( "h" ), {
    value: 8,
    enumerable: false
} );
Object.setPrototypeOf( o3, o4 );

Object.assign( target, o1, o2, o3 );
target.a; // 1
target.b; // 2
target.c; // 3
Object.getOwnPropertyDescriptor( target, "e" );
// { value: 5, writable: true, enumerable: true,
// configurable: true }
Object.getOwnPropertySymbols( target );
// [Symbol("g")]
```

- 只有属性 a、 b、 c、 e 以及 Symbol("g") 会被复制到 target 中：
- 复制过程会忽略属性 d、 f 和 Symbol("h")；
- 不可枚举的属性和非自有的属性都被排除在赋值过程之外。
- 另外， e 作为一个普通属性赋值被复制，而不是作为只读属性复制。  

## Math  

### 三角函数：

cosh(..)
双曲余弦函数
acosh(..)
双曲反余弦函数
sinh(..)
双曲正弦函数
asinh(..)
双曲反正弦函数
tanh(..)
双曲正切函数
atanh(..)
双曲反正切函数
hypot(..)
平方和的平方根（也即：广义勾股定理）  

### 算术：

cbrt(..)
立方根
clz32(..)
计算 32 位二进制表示的前导 0 个数
expm1(..)
等价于 exp(x) - 1
log2(..)
二进制对数（以 2 为底的对数）
log10(..)
以 10 为底的对数
log1p(..)
等价于 log(x + 1)
imul(..)
两个数字的 32 位整数乘法  

### 元工具：

sign(..)
返回数字符号
trunc(..)
返回数字的整数部分
fround(..)
向最接近的 32 位（单精度）浮点值取整  

## Number  

### 静态属性  

- **Number.EPSILON**
  任意两个值之间的最小差： 2^-52（参见本系列《你不知道的 JavaScript（中卷）》第一
  部分的第 2 章，其使用了这个值作为浮点数算法的精度误差值）
- **Number.MAX_SAFE_INTEGER**
  JavaScript 可以用数字值无歧义“安全”表达的最大整数： 2^53 - 1
- **Number.MIN_SAFE_INTEGER**
  JavaScript 可以用数字值无歧义“安全”表达的最小整数： -(2^53 - 1) 或 (-2)^53 + 1  

### Number.isNaN(..)  

isNaN(..) 自出现以来就是有缺陷的  

- 它对非数字的东西都会返回 true，而不是只对真实的 NaN 值返回 true  

ES6 增加了一个修正工具 Number.isNaN(..)  

```javascript
var a = NaN, b = "NaN", c = 42;
isNaN( a ); // true
isNaN( b ); // true--oops!
isNaN( c ); // false
Number.isNaN( a ); // true
Number.isNaN( b ); // false--修正了!
Number.isNaN( c ); // false
```

### Number.isFinite(..)  

isFinite(..) 这样的函数名，我们常常认为它的意思就是“非无限的”。  

```javascript
var a = NaN, b = Infinity, c = 42;
Number.isFinite( a ); // false
Number.isFinite( b ); // false
Number.isFinite( c ); // true  
```

### 整型相关静态函数  

Number.isInteger(..)  

这个工具可能会更有效地确定这个性质：

```javascript
Number.isInteger( 4 ); // true
Number.isInteger( 4.2 ); // false  
```

Number.isSafeInteger(..)  

这个工具检查一个值以确保其为整数并且在 Number.MIN_SAFE_INTEGER-Number.MAX_SAFE_INTEGER 范围之内（全包含）：  

## 字符串  

### repeat(..)  

ES6 定义了一个字符串原型方法 repeat(..) 来完成这个任务：
"foo".repeat( 3 ); // "foofoofoo"  

### 字符串检查函数  

startsWith(..)、 endsWidth(..) 和 includes(..)  

```javascript
var palindrome = "step on no pets";
palindrome.startsWith( "step on" ); // true
palindrome.startsWith( "on", 5 ); // true
palindrome.endsWith( "no pets" ); // true
palindrome.endsWith( "no", 10 ); // true
palindrome.includes( "on" ); // true
palindrome.includes( "on", 6 ); // false  
```

