# JS前端基础理论-----基本语法

# 原生函数  

JavaScript 为基本数据类型值提供了封装对象，称为原⽣函数

## 常⽤的原⽣函数有：

String()
Number()
Boolean()
Array()
Object()
Function()
RegExp()
Date()
Error()
Symbol() ——ES6 中新加⼊的！
实际上，它们就是内建函数。  

## 内部属性 [[Class]]  

这个属性⽆法直接访问，⼀般通过Object.prototype.toString(..) 来查看。  

```javascript
Object.prototype.toString.call( [1,2,3] );
// "[object Array]"
Object.prototype.toString.call( /regex-literal/i );
// "[object RegExp]"  
```

那么基本类型值呢？下⾯先来看看 null 和 undefined ：

```javascript
Object.prototype.toString.call( null );
// "[object Null]"
Object.prototype.toString.call( undefined );
// "[object Undefined]"  
```

## 原⽣函数作为构造函数  

### Array(..)

```javascript
var a = new Array( 1, 2, 3 );
a; // [1, 2, 3]
var b = [1, 2, 3];
b; // [1, 2, 3]
```

构造函数 Array(..) 不要求必须带 new 关键字。不带时，它会被⾃动补上。因此 Array(1,2,3) 和 new
Array(1,2,3) 的效果是⼀样的。  

### Object(..) 、Function(..) 和 RegExp(..)  

同样，除⾮万不得已，否则尽量不要使⽤Object(..)/Function(..)/RegExp(..) ：  

### Date(..) 和 Error(..)  

创建⽇期对象必须使⽤ new Date() 。Date(..) 可以带参数，⽤来指定⽇期和时间，⽽不带参数的话则使⽤当前的⽇期和时间。  

Date(..) 主要⽤来获得当前的 Unix 时间戳（从 1970 年 1 ⽉ 1 ⽇
开始计算，以秒为单位）。该值可以通过⽇期对象中的 getTime()
来获得。

从 ES5 开始引⼊了⼀个更简单的⽅法，即静态函数 Date.now() 。对
ES5 之前的版本我们可以使⽤下⾯的 polyfill：

```javascript
if (!Date.now) {
    Date.now = function(){
    	return (new Date()).getTime();
    };
}  
```

创建错误对象（error object）主要是为了获得当前运⾏栈的上下⽂（⼤部分 JavaScript 引擎通过只读属性 .stack 来访问）。栈上下⽂信息包括函数调⽤栈信息和产⽣错误的代码⾏号，以便于调试（debug）。

错误对象通常与 throw ⼀起使⽤：

```javascript
function foo(x) {
    if (!x) {
    	throw new Error( "x wasn't provided" );
    }
    // ..
}
```

 通常错误对象⾄少包含⼀个 message 属性，有时也不乏其他属性（必须作为只读属性访问），如 type 。除了访问 stack 属性以外，最好的办法是调⽤（显式调⽤或者通过强制类型转换隐式调⽤，toString() 来获得经过格式化的便于阅读的错误信息。  

### Symbol(..)  

ES6 中新加⼊了⼀个基本数据类型 ——符号（Symbol）。符号是具有唯⼀性的特殊值（并⾮绝对）

符号可以⽤作属性名，但⽆论是在代码还是开发控制台中都⽆法查看和访问它的值，只会显⽰为诸如 Symbol(Symbol.create) 这样的值。  

我们可以使⽤ Symbol(..) 原⽣构造函数来⾃定义符号。但它⽐较特殊，不能带 new 关键字，否则会出错：

```javascript
var mysym = Symbol( "my own symbol" );
mysym; // Symbol(my own symbol)
mysym.toString(); // "Symbol(my own symbol)"
typeof mysym; // "symbol"

var a = { };
a[mysym] = "foobar";

Object.getOwnPropertySymbols( a );
// [ Symbol(my own symbol) ]  
```

### 原⽣原型  

原⽣构造函数有⾃⼰的 .prototype 对象，如 Array.prototype 、String.prototype 等。
这些对象包含其对应⼦类型所特有的⾏为特征。  

# 强制类型转换  

## 值类型转换  

- 类型转换  

  将值从⼀种类型转换为另⼀种类型通常称为显式的类型转换

  类型转换发⽣在静态类型语⾔的编译阶段，  

- 强制类型转换  

  隐式的情况称为强制类型转  

  强制类型转换则发⽣在动态类型语⾔的运⾏时（runtime）  

例如：

```javascript
var a = 42;
var b = a + ""; // 隐式强制类型转换
var c = String( a ); // 显式强制类型转换  
```

## 抽象值操作  

### ToString  

它负责处理⾮字符串到字符串的强制类型转换。  

1. 基本类型值的字符串化规则为：null 转换为 "null" ，undefined转换为 "undefined" ，true 转换为 "true" 。数字的字符串化则遵循通⽤规则

2. 对普通对象来说，除⾮⾃⾏定义，否则 toString()（Object.prototype.toString() ）返回内部 属性 [[Class]]的值，如 "[object Object]" 。  

3. 数组的默认 toString() ⽅法经过了重新定义，将所有单元字符串化以后再⽤ "," 连接起来：  

   ```javascript
   var a = [1,2,3];
   a.toString(); // "1,2,3"
   ```

#### JSON 字符串化

⼯具函数 JSON.stringify(..) 在将 JSON 对象序列化为字符串时也⽤到了 ToString 。

> 请注意，JSON 字符串化并⾮严格意义上的强制类型转换，因为其中也涉及 ToString 的相关规则，所以这⾥顺带介绍⼀下。

对⼤多数简单值来说，JSON 字符串化和 toString() 的效果基本相同，只不过序列化的结果总是字符串：  

1. JSON.stringify(..) 在对象中遇到 undefined 、function 和symbol 时会⾃动将其忽略，在数组中则会返回 null （以保证单元 位置不变）。  

   例如：

   ```javascript
   JSON.stringify( undefined ); // undefined
   JSON.stringify( function(){} ); // undefined
   
   JSON.stringify(
   	[1,undefined,function(){},4]
   ); // "[1,null,null,4]"
   JSON.stringify(
   	{ a:2, b:function(){} }
   ); // "{"a":2}"  
   ```

   

2. 对包含循环引⽤的对象执⾏ JSON.stringify(..) 会出错。如果对象中定义了 toJSON() ⽅法，JSON 字符串化时会⾸先调⽤该⽅法，然后⽤它的返回值来进⾏序列化。

3. 如果要对含有⾮法 JSON 值的对象做字符串化，或者对象中的某些值⽆法被序列化时，就需要定义 toJSON() ⽅法来返回⼀个安全的JSON 值。  

   例如：

   ```javascript
   var o = { };
   var a = {
       b: 42,
       c: o,
       d: function(){}
   };
   // 在a中创建⼀个循环引⽤
   o.e = a;
   // 循环引⽤在这⾥会产⽣错误  
   
   // JSON.stringify( a );
   // ⾃定义的JSON序列化
   a.toJSON = function() {
       // 序列化仅包含b
       return { b: this.b };
   };
   JSON.stringify( a ); // "{"b":42}"  
   ```

   

很多⼈误以为 toJSON() 返回的是 JSON 字符串化后的值，其实不然，除⾮我们确实想要对字符串进⾏字符串化（通常不会！）。

toJSON() 返回的应该是⼀个适当的值，可以是任何类型，然后再由JSON.stringify(..) 对其进⾏字符串化。

也就是说，**toJSON() 应该“返回⼀个能够被字符串化的安全的 JSON值”**，⽽不是“返回⼀个 JSON 字符串”。  

例如：

```javascript
var a = {
    val: [1,2,3],
    // 可能是我们想要的结果！
    toJSON: function(){
    	return this.val.slice( 1 );
    }
};

var b = {
	val: [1,2,3],
    // 可能不是我们想要的结果！
    toJSON: function(){
        return "[" +
            this.val.slice( 1 ).join() +
        "]";
    }
};  

JSON.stringify( a ); // "[2,3]"
JSON.stringify( b ); // ""[2,3]""  
```

我们可以向 JSON.stringify(..) 传递⼀个可选参数 replacer，它可以是数组或者函数，⽤来指定对象序列化过程中哪些属性应该被处理，哪些应该被排除，和 toJSON() 很像。

如果 replacer 是⼀个数组，那么它必须是⼀个字符串数组，其中包含序列化要处理的对象的属性名称，除此之外其他的属性则被忽略。

如果 replacer 是⼀个函数，它会对对象本⾝调⽤⼀次，然后对对象中的每个属性各调⽤⼀次，每次传递两个参数，键和值。如果要忽略某个键就返回 undefined ，否则返回指定的值。

```javascript
var a = {
    b: 42,
    c: "42",
    d: [1,2,3]
};

JSON.stringify( a, ["b","c"] ); // "{"b":42,"c":"42"}"

JSON.stringify( a, function(k,v){
	if (k !== "c") return v;
} );
// "{"b":42,"d":[1,2,3]}"  
```

JSON.string 还有⼀个可选参数 space，⽤来指定输出的缩进格式。space 为正整数时是指定每⼀级缩进的字符数，它还可以是字符串，此时最前⾯的⼗个字符被⽤于每⼀级的缩进：  

```javascript
var a = {
    b: 42,
    c: "42",
    d: [1,2,3]
};
JSON.stringify( a, null, 3 );
// "{
    // "b": 42,
    // "c": "42",
    // "d": [
        // 1,
        // 2,
        // 3
    // ]
// }"
JSON.stringify( a, null, "-----" );
// "{
// -----"b": 42,
// -----"c": "42",
// -----"d": [
// ----------1,
// ----------2,
// ----------3
// -----]
// }"  
```

请记住，JSON.stringify(..) 并不是强制类型转换。在这⾥介绍是因为它涉及 ToString 强制类型转换，具体表现在以下两点。

(1) 字符串、数字、布尔值和 null 的 JSON.stringify(..) 规则与 ToString 基本相同。

(2) 如果传递给 JSON.stringify(..) 的对象中定义了 toJSON()⽅法，那么该⽅法会在字符串化前调⽤，以便将对象转换为安全的JSON 值。  

### ToNumber  

有时我们需要将⾮数字值当作数字来使⽤，⽐如数学运算。为此 ES5规范在 9.3 节定义了抽象操作 ToNumber 。

1. 其中 true 转换为 1 ，false 转换为 0 。undefined 转换为 NaN ，null 转换为 0 。
2. ToNumber 对字符串的处理基本遵循数字常量的相关规则 / 语法。处理失败时返回 NaN （处理数字常量失败时会产⽣语法错误）。不同之处是 ToNumber 对以 0 开头的⼗六进制数并不按⼗六进制处理（⽽是按⼗进制）。  
3. 对象（包括数组）会⾸先被转换为相应的基本类型值，如果返回的是⾮数字的基本类型值，则再遵循以上规则将其强制转换为数字。  

### ToBoolean  

JavaScript 中有两个关键词 true 和false ，分别代表布尔类型中的真和假。我们常误以为数值 1 和 0 分别等同于 true 和 false 。在有些语⾔中可能是这样，但在JavaScript 中布尔值和数字是不⼀样的。虽然我们可以将 1 强制类型转换为 true ，将 0 强制类型转换为 false ，反之亦然，但它们并不是⼀回事。  

#### 假值（falsy value）  

JavaScript 中的值可以分为以下两类：
(1) 可以被强制类型转换为 false 的值
(2) 其他（被强制类型转换为 true 的值）
JavaScript 规范具体定义了⼀⼩撮可以被强制类型转换为 false 的值。  

ES5 规范 9.2 节中定义了抽象操作 ToBoolean ，列举了布尔强制类型转换所有可能出现的结果。  

以下这些是假值：

- undefined
- null
- false
- +0 、-0 和 NaN
- ""

假值的布尔强制类型转换结果为 false 。

从逻辑上说，假值列表以外的都应该是真值（truthy）。但JavaScript 规范对此并没有明确定义，只是给出了⼀些⽰例，例如规定所有的对象都是真值，我们可以理解为假值列表以外的值都是真值 。  

#### 假值对象（falsy object）  

前⾯讲过规范规定所有的对象都是真值，怎么还会有假值对象呢？  

例如：

```javascript
var a = new Boolean( false );
var b = new Number( 0 );
var c = new String( "" );  
var d = Boolean( a && b && c );
d; // true  
```

d 为 true ，说明 a 、b 、c 都为 true 。  

如果假值对象并⾮封装了假值的对象，那它究竟是什么？

值得注意的是，虽然 JavaScript 代码中会出现假值对象，但它实际上并不属于 JavaScript 语⾔的范畴。

浏览器在某些特定情况下，在常规 JavaScript 语法基础上⾃⼰创建了⼀些外来 （exotic）值，这些就是“假值对象”。  

#### 真值（truthy value）  

真值就是假值列表之外的值。  

例如：

```javascript
var a = "false";
var b = "0";
var c = "''";
var d = Boolean( a && b && c );
d;  
```

```javascript
var a = []; // 空数组——是真值还是假值？
var b = {}; // 空对象——是真值还是假值？
var c = function(){}; // 空函数——是真值还是假值？
var d = Boolean( a && b && c );
d;
```

还是同样的道理，[] 、{} 和 function(){} 都不在假值列表中，因此它们都是真值。

也就是说真值列表可以⽆限⻓，⽆法⼀⼀列举，所以我们只能⽤假值列表作为参考。  

### 显式强制类型转换  

#### 字符串和数字之间的显式转换  

字符串和数字之间的转换是通过 String(..) 和 Number(..) 这两个内建函数（原⽣构造函数，参⻅第 3 章）来实现的，请注意它们前⾯没有 new 关键字，并不创建封装对象。

下⾯是两者之间的显式强制类型转换：

```javascript
var a = 42;
var b = String( a );
var c = "3.14";
var d = Number( c );
b; // "42"
d; // 3.14
```

String(..) 遵循前⾯讲过的 ToString 规则，将值转换为字符串基本类型。Number(..) 遵循前⾯讲过的 ToNumber 规则，将值转换为数字基本类型。  

除了 String(..) 和 Number(..) 以外，还有其他⽅法可以实现字符串和数字之间的显式转换：

```javascript
var a = 42;
var b = a.toString();
var c = "3.14";
var d = +c;
b; // "42"
d; // 3.14
```

a.toString() 是显式的（“toString”意为“to a string”），不过其中涉及隐式转换。**因为 toString() 对 42 这样的基本类型值不适⽤**，所以 JavaScript 引擎会⾃动为 42 创建⼀个封装对象，然后对该对象调⽤ toString() 。这⾥显式转换中含有隐式转换。  

##### ⽇期显式转换为数字  

```javascript
var d = new Date( "Mon, 18 Aug 2014 08:53:06 CDT" );
+d; // 1408369986000
```

我们常⽤下⾯的⽅法来获得当前的时间戳，例如：

```javascript
var timestamp = +new Date(); 
```

 将⽇期对象转换为时间戳并⾮只有强制类型转换这⼀种⽅法，或许使
⽤更显式的⽅法会更好⼀些：

```javascript
var timestamp = new Date().getTime();
// var timestamp = (new Date()).getTime();
// var timestamp = (new Date).getTime();
```

不过最好还是使⽤ ES5 中新加⼊的静态⽅法 Date.now() ：

```javascript
var timestamp = Date.now();  
```

为⽼版本浏览器提供 Date.now() 的 polyfill 也很简单：

```javascript
if (!Date.now) {
    Date.now = function() {
    	return +new Date();
    };
}
```

 我们不建议对⽇期类型使⽤强制类型转换，应该使⽤ 

- Date.now() 来获得当前的时间戳，
- new Date(..).getTime() 来获得指定时间的时间戳。  

##### 奇特的 ~ 运算符  

indexOf(..) 不仅能够得到⼦字符串的位置，还可以⽤来检查字符串中是否包含指定的⼦字符串，相当于⼀个条件判断。例如：

```javascript
var a = "Hello World";
if (a.indexOf( "lo" ) >= 0) { // true
// 找到匹配！
}
if (a.indexOf( "lo" ) != -1) { // true
// 找到匹配！
}
if (a.indexOf( "ol" ) < 0) { // true
// 没有找到匹配！
}
if (a.indexOf( "ol" ) == -1) { // true
// 没有找到匹配！
}
```

\>= 0 和 == -1 这样的写法不是很好，称为“抽象渗漏”，意思是在代码中暴露了底层的实现细节，这⾥是指⽤ -1 作为失败时的返回值，这些细节应该被屏蔽掉。

现在我们终于明⽩ ~ 有什么⽤处了！ ~ 和 indexOf() ⼀起可以将结果强制类型转换（实际上仅仅是转换）为真 / 假值：

```javascript
var a = "Hello World";

~a.indexOf( "lo" ); // -4 <-- 真值!

if (~a.indexOf( "lo" )) { // true
// 找到匹配！
}

~a.indexOf( "ol" ); // 0 <-- 假值!
!~a.indexOf( "ol" ); // true

if (!~a.indexOf( "ol" )) { // true
// 没有找到匹配！
} 
```

如果 indexOf(..) 返回 -1 ，~ 将其转换为假值 0 ，其他情况⼀律转换为真值。  

##### 字位截除  

⼀些开发⼈员使⽤ ~~ 来截除数字值的⼩数部分，以为这和Math.floor(..) 的效果⼀样，实际上并⾮如此。

~~ 中的第⼀个 ~ 执⾏ ToInt32 并反转字位，然后第⼆个 ~ 再进⾏⼀次字位反转，即将所有字位反转回原值，最后得到的仍然是 ToInt32的结果。  

对 ~~ 我们要多加注意。⾸先它只适⽤于 32 位数字，更重要的是它对负数的处理与 Math.floor(..) 不同。

```javascript
Math.floor( -49.6 ); // -50
~~-49.6; // -49
```

~~x 能将值截除为⼀个 32 位整数，x | 0 也可以，⽽且看起来还更简洁。  

#### 显式解析数字字符串  

解析字符串中的数字和将字符串强制类型转换为数字的返回结果都是数字。但**解析和转换**两者之间还是有明显的差别。
例如：

```javascript
var a = "42";
var b = "42px";

Number( a ); // 42
parseInt( a ); // 42

Number( b ); // NaN
parseInt( b ); // 42
```

- 解析允许 字符串中含有⾮数字字符，解析按从左到右的顺序，如果遇到⾮数字字符就停⽌。
- ⽽转换不允许 出现⾮数字字符，否则会失败并返回 NaN 。  

##### 解析⾮字符串

曾经有⼈发帖吐槽过 parseInt(..) 的⼀个坑：

```javascript
parseInt( 1/0, 19 ); // 18
```

很多⼈想当然地以为（实际上⼤错特错）“如果第⼀个参数值为Infinity ，解析结果也应该是 Infinity ”，返回 18 也太⽆厘头了。  

parseInt(1/0, 19) 实际上是 parseInt("Infinity", 19) 。第⼀个字符是 "I" ，以 19 为基数时值为 18 。第⼆个字符 "n" 不是⼀个有效的数字字符，解析到此为⽌，和 "42px" 中的 "p" ⼀样。最后的结果是 18 ，⽽⾮ Infinity 或者报错。所以理解其中的⼯作原理对于我们学习 JavaScript 是⾮常重要的。

此外还有⼀些看起来奇怪但实际上解释得通的例⼦：

```javascript
parseInt( 0.000008 ); // 0 ("0" 来⾃于 "0.000008")
parseInt( 0.0000008 ); // 8 ("8" 来⾃于 "8e-7")
parseInt( false, 16 ); // 250 ("fa" 来⾃于 "false")
parseInt( parseInt, 16 ); // 15 ("f" 来⾃于 "function..")
parseInt( "0x10" ); // 16
parseInt( "103", 2 ); // 2
```

其实 parseInt(..) 函数是⼗分靠谱的，只要使⽤得当就不会有问题。因为使⽤不当⽽导致⼀些莫名其妙的结果，并不能归咎于JavaScript 本⾝。  

#### 显式转换为布尔值  

虽然 Boolean(..) 是显式的，但并不常⽤  

最常⽤的⽅法是 !! ，因为第⼆个 ! 会将结果反转回原值：  

```javascript
var a = "0";
var b = [];
var c = {};
var d = "";
var e = 0;
var f = null;
var g;
!!a; // true
!!b; // true
!!c; // true
!!d; // false
!!e; // false
!!f; // false
!!g; // false  
```

在 if(..).. 这样的布尔值上下⽂中，如果没有使⽤ Boolean(..)和 !! ，就会⾃动隐式地进⾏ ToBoolean 转换。建议使⽤Boolean(..) 和 !! 来进⾏显式转换以便让代码更清晰易读。  

显式 ToBoolean 的另外⼀个⽤处，是在 JSON 序列化过程中将值强制类型转换为 true 或 false ：

```javascript
var a = [
    1,
    function(){ /*..*/ },
    2,
    function(){ /*..*/ }
];

JSON.stringify( a ); // "[1,null,2,null]"

JSON.stringify( a, function(key,val){
    if (typeof val == "function") {
        // 函数的ToBoolean强制类型转换
        return !!val;
    }
    else {
        return val;
    }
} );
// "[1,true,2,true]"  
```

### 隐式强制类型转换  

1. 隐式地简化  
2. 字符串和数字之间的隐式强制类型转换  
3. 布尔值到数字的隐式强制类型转换  
4. 隐式强制类型转换为布尔值  
   1. if (..) 语句中的条件判断表达式。
   2. for ( .. ; .. ; .. ) 语句中的条件判断表达式（第⼆个）。
   3. while (..) 和 do..while(..) 循环中的条件判断表达式。
   4. ? : 中的条件判断表达式。
   5. 逻辑运算符 || （逻辑或）和 && （逻辑与）左边的操作数（作为条件判断表达式）。  
5. || 和 &&  
6. 符号的强制类型转换  

### 宽松相等和严格相等  

- 宽松相等（loose equals）==
- 严格相等（strict equals）===  

正确的解释是：“== 允许在相等⽐较中进⾏强制类型转换，⽽ === 不允许。”  

#### 相等⽐较操作的性能  

有⼈觉得 == 会⽐ === 慢，实际上虽然强制类型转换确实要多花点时间，但仅仅是微秒级（百万分之⼀秒）的差别⽽已。  

#### 抽象相等  

有⼏个⾮常规的情况需要注意。
NaN 不等于 NaN （参⻅第 2 章）。
+0 等于 -0 （参⻅第 2 章）。  

主要涉及 == 和 === 之间强制比较的情况

- 字符串和数字之间的相等⽐较  
- 其他类型和布尔类型之间的相等⽐较  
- null 和 undefined 之间的相等⽐较  
- 对象和⾮对象之间的相等⽐较  

### 抽象关系⽐较  

如果结果出现⾮字符串，就根据ToNumber 规则将双⽅强制类型转换为数字来进⾏⽐较  

如果⽐较双⽅都是字符串，则按字⺟顺序来进⾏⽐较

# 语法  

## 语句和表达式  

开发⼈员常常将“语句”（statement）和“表达式”（expression）混为⼀谈，  

### 语句的结果值  

语句都有⼀个结果值（statement completion value，undefined 也算）  

代码块的结果值就如同⼀个隐式的返回 ，即返回最后⼀个语句的结果值。  

### 表达式的副作⽤  

最常⻅的有副作⽤（也可能没有）的表达式是函数调⽤：

```javascript
function foo() {
a = a + 1;
}
var a = 1;
foo(); // 结果值：undefined。副作⽤：a的值被改变  
```

其他⼀些表达式也有副作⽤，⽐如：

```javascript
var a = 42;
var b = a++;
```

a++ ⾸先返回变量 a 的当前值 42 （再将该值赋给 b ），然后将 a 的值加 1：

```javascript
var a = 42;
var b = a++;
a; // 43
b; // 42  
```

常有⼈误以为可以⽤括号 ( ) 将 a++ 的副作⽤封装起来，例如：

```javascript
var a = 42;
var b = (a++);
a; // 43
b; // 42  
```

事实并⾮如此。( ) 本⾝并不是⼀个封装表达式，不会在表达式 a++产⽣副作⽤之后执⾏。即便可以，a++ 会⾸先返回 42 ，除⾮有表达式在 ++ 之后再次对 a 进⾏运算，否则还是不会得到 43 ，也就不能将 43 赋值给 b 。  

但也不是没有办法，可以使⽤ , 语句系列逗号运算符（statementseries comma operator）将多个独⽴的表达式语句串联成⼀个语句：

```javascript
var a = 42, b;
b = ( a++, a );
a; // 43
b; // 43  
```

> 由于运算符优先级的关系，a++, a 需要放到 ( .. )中。本章后⾯将会介绍。

a++, a 中第⼆个表达式 a 在 a++ 之后执⾏，结果为 43 ，并被赋值给 b 。  

再如 delete 运算符。第 2 章讲过，delete ⽤来删除对象中的属性和数组中的单元。它通常以单独⼀个语句的形式出现：

```javascript
var obj = {
	a: 42
};
obj.a; // 42
delete obj.a; // true
obj.a; // undefined
```

如果操作成功，delete 返回 true ，否则返回 false 。其副作⽤是属性被从对象中删除（或者单元从 array 中删除）。  

多个赋值语句串联时（链式赋值，chained assignment），赋值表达式（和语句）的结果值就能派上⽤场，⽐如：

```javascript
var a, b, c;
a = b = c = 42;
```

这⾥ c = 42 的结果值为 42 （副作⽤是将 c 赋值 42 ），然后 b =42 的结果值为 42 （副作⽤是将 b 赋值 42 ），最后是 a = 42 （副作⽤是将 a 赋值 42 ）。  

> 链式赋值常常被误⽤，例如 var a = b = 42 ，看似和前⾯的例⼦差不多，实则不然。如果变量 b 没有在作⽤域中象var b 这样声明过，则 var a = b = 42 不会对变量 b 进⾏声明。在严格模式中这样会产⽣错误，或者会⽆意中创建⼀个全局变量  

### 上下⽂规则  

这⾥我们不⼀⼀列举，只介绍⼀些常⻅情况。

#### ⼤括号

下⾯两种情况会⽤到⼤括号 { .. } （随着 JavaScript 的演进会出现更多类似的情况）。

##### 对象常量

⽤⼤括号定义对象常量（object literal）：

```javascript
// 假定函数bar()已经定义
var a = {
	foo: bar()
};  
```



##### 标签  

如果将上例中的 var a = 去掉会发⽣什么情况呢？

```javascript
// 假定函数bar()已经定义
{
	foo: bar()
}  
```



#### 代码块

还有⼀个坑常被提到（涉及强制类型转换，参⻅第 4 章）：

```javascript
[] + {}; // "[object Object]"
{} + []; // 0  
```

表⾯上看 + 运算符根据第⼀个操作数（[] 或 {} ）的不同会产⽣不同的结果，实则不然。

第⼀⾏代码中，{} 出现在 + 运算符表达式中，因此它被当作⼀个值（空对象）来处理。第 4 章讲过 [] 会被强制类型转换为 "" ，⽽ {}会被强制类型转换为 "[object Object]" 。

但在第⼆⾏代码中，{} 被当作⼀个独⽴的空代码块（不执⾏任何操作）。代码块结尾不需要分号，所以这⾥不存在语法上的问题。最后\+ [] 将 [] 显式强制类型转换为 0 。  

#### 对象解构  

从 ES6 开始，{ .. } 也可⽤于“解构赋值”（destructuring assignment），特别是对象的解构。例如：  

```javascript
function getData() {
    // ..
    return {
        a: 42,
        b: "foo"
    };
}

var { a, b } = getData();

console.log( a, b ); // 42 "foo"
```

{ a , b } = .. 就是 ES6 中的解构赋值，相当于下⾯的代码：

```javascript
var res = getData();
var a = res.a;
var b = res.b;  
```

{ .. } 还可以⽤作函数命名参数（named function argument）的对象解构（object destructuring），⽅便隐式地⽤对象属性赋值：

```javascript
function foo({ a, b, c }) {
    // 不再需要这样:
    // var a = obj.a, b = obj.b, c = obj.c
    console.log( a, b, c );
}
foo( {
    c: [1,2,3],
    a: 42,
    b: "foo"  
} ); // 42 "foo" [1, 2, 3]  
```

#### else if 和可选代码块  

很多⼈误以为 JavaScript 中有 else if ，因为我们可以这样来写代码：

```javascript
if (a) {
	// ..
}
else if (b) {
	// ..
}
else {
	// ..
}  
```

我们经常⽤到的 else if 实际上是这样的：  

```javascript
if (a) {
	// ..
}
else {
    if (b) {
    	// ..
    }
    else {
    	// ..
    }
}
```

if (b) { .. } else { .. } 实际上是跟在 else 后⾯的⼀个单独的语句，所以带不带 { } 都可以。换句话说，else if 不符合前⾯介绍的编码规范，else 中是⼀个单独的 if 语句。

else if 极为常⻅，能省掉⼀层代码缩进，所以很受⻘睐。但这只是我们⾃⼰发明的⽤法，切勿想当然地认为这些都属于 JavaScript 语法的范畴。  

### 运算符优先级  

下面的表将所有运算符按照优先级的不同从高（20）到低（1）排列。

<table class="fullwidth-table">
 <tbody>
  <tr>
   <th>优先级</th>
   <th>运算类型</th>
   <th>关联性</th>
   <th>运算符</th>
  </tr>
  <tr>
   <td>21</td>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/Grouping"><code>圆括号</code></a></td>
   <td>n/a（不相关）</td>
   <td><code>( … )</code></td>
  </tr>
  <tr>
   <td rowspan="5">20</td>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/Property_Accessors#%e7%82%b9%e7%ac%a6%e5%8f%b7%e8%a1%a8%e7%a4%ba%e6%b3%95"><code>成员访问</code></a></td>
   <td>从左到右</td>
   <td><code>… . …</code></td>
  </tr>
  <tr>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/Property_Accessors#%e6%8b%ac%e5%8f%b7%e8%a1%a8%e7%a4%ba%e6%b3%95"><code>需计算的成员访问</code></a></td>
   <td>从左到右</td>
   <td><code>… [ … ]</code></td>
  </tr>
  <tr>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/new"><code>new</code></a> (带参数列表)</td>
   <td>n/a</td>
   <td><code>new … ( … )</code></td>
  </tr>
  <tr>
   <td><a href="/zh-CN/docs/Web/JavaScript/Guide/Functions" title="JavaScript/Reference/Operators/Special_Operators/function_call">函数调用</a></td>
   <td>从左到右</td>
   <td><code>… (&nbsp;<var>…&nbsp;</var>)</code></td>
  </tr>
  <tr>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/Optional_chaining">可选链（Optional chaining）</a></td>
   <td>从左到右</td>
   <td><code>?.</code></td>
  </tr>
  <tr>
   <td rowspan="1">19</td>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/new" title="JavaScript/Reference/Operators/Special_Operators/new_Operator">new</a>&nbsp;(无参数列表)</td>
   <td>从右到左</td>
   <td><code>new …</code></td>
  </tr>
  <tr>
   <td rowspan="2">18</td>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/Arithmetic_Operators#Increment" title="JavaScript/Reference/Operators/Arithmetic_Operators">后置递增</a>(运算符在后)</td>
   <td colspan="1" rowspan="2">n/a<br>
    &nbsp;</td>
   <td><code>… ++</code></td>
  </tr>
  <tr>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/Arithmetic_Operators#Decrement" title="JavaScript/Reference/Operators/Arithmetic_Operators">后置递减</a>(运算符在后)</td>
   <td><code>… --</code></td>
  </tr>
  <tr>
   <td colspan="1" rowspan="10">17</td>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/Logical_Operators#Logical_NOT">逻辑非</a></td>
   <td colspan="1" rowspan="10">从右到左</td>
   <td><code>! …</code></td>
  </tr>
  <tr>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/Bitwise_Operators#Bitwise_NOT" title="JavaScript/Reference/Operators/Bitwise_Operators">按位非</a></td>
   <td><code>~ …</code></td>
  </tr>
  <tr>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/Arithmetic_Operators#Unary_plus" title="JavaScript/Reference/Operators/Arithmetic_Operators">一元加法</a></td>
   <td><code>+ …</code></td>
  </tr>
  <tr>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/Arithmetic_Operators#Unary_negation" title="JavaScript/Reference/Operators/Arithmetic_Operators">一元减法</a></td>
   <td><code>- …</code></td>
  </tr>
  <tr>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/Arithmetic_Operators#Increment" title="JavaScript/Reference/Operators/Arithmetic_Operators">前置递增</a></td>
   <td><code>++ …</code></td>
  </tr>
  <tr>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/Arithmetic_Operators#Decrement" title="JavaScript/Reference/Operators/Arithmetic_Operators">前置递减</a></td>
   <td><code>-- …</code></td>
  </tr>
  <tr>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/typeof" title="JavaScript/Reference/Operators/Special_Operators/typeof_Operator">typeof</a></td>
   <td><code>typeof …</code></td>
  </tr>
  <tr>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/void" title="JavaScript/Reference/Operators/Special_Operators/void_Operator">void</a></td>
   <td><code>void …</code></td>
  </tr>
  <tr>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/delete" title="JavaScript/Reference/Operators/Special_Operators/delete_Operator">delete</a></td>
   <td><code>delete …</code></td>
  </tr>
  <tr>
   <td><a href="/en-US/docs/Web/JavaScript/Reference/Operators/await">await</a></td>
   <td><code>await …</code></td>
  </tr>
  <tr>
   <td>16</td>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/Arithmetic_Operators#Exponentiation" title="JavaScript/Reference/Operators/Arithmetic_Operators">幂</a></td>
   <td>从右到左</td>
   <td><code>…&nbsp;**&nbsp;…</code></td>
  </tr>
  <tr>
   <td rowspan="3">15</td>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/Arithmetic_Operators#Multiplication" title="JavaScript/Reference/Operators/Arithmetic_Operators">乘法</a></td>
   <td colspan="1" rowspan="3">从左到右<br>
    &nbsp;</td>
   <td><code>… *&nbsp;…</code></td>
  </tr>
  <tr>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/Arithmetic_Operators#Division" title="JavaScript/Reference/Operators/Arithmetic_Operators">除法</a></td>
   <td><code>… /&nbsp;…</code></td>
  </tr>
  <tr>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/Arithmetic_Operators#Remainder" title="JavaScript/Reference/Operators/Arithmetic_Operators">取模</a></td>
   <td><code>… %&nbsp;…</code></td>
  </tr>
  <tr>
   <td rowspan="2">14</td>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/Arithmetic_Operators#Addition" title="JavaScript/Reference/Operators/Arithmetic_Operators">加法</a></td>
   <td colspan="1" rowspan="2">从左到右<br>
    &nbsp;</td>
   <td><code>… +&nbsp;…</code></td>
  </tr>
  <tr>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/Arithmetic_Operators#Subtraction" title="JavaScript/Reference/Operators/Arithmetic_Operators">减法</a></td>
   <td><code>… -&nbsp;…</code></td>
  </tr>
  <tr>
   <td rowspan="3">13</td>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/Bitwise_Operators" title="JavaScript/Reference/Operators/Bitwise_Operators">按位左移</a></td>
   <td colspan="1" rowspan="3">从左到右</td>
   <td><code>… &lt;&lt;&nbsp;…</code></td>
  </tr>
  <tr>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/Bitwise_Operators" title="JavaScript/Reference/Operators/Bitwise_Operators">按位右移</a></td>
   <td><code>… &gt;&gt;&nbsp;…</code></td>
  </tr>
  <tr>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/Bitwise_Operators" title="JavaScript/Reference/Operators/Bitwise_Operators">无符号右移</a></td>
   <td><code>… &gt;&gt;&gt;&nbsp;…</code></td>
  </tr>
  <tr>
   <td rowspan="6">12</td>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/Comparison_Operators#Less_than_operator" title="JavaScript/Reference/Operators/Comparison_Operators">小于</a></td>
   <td colspan="1" rowspan="6">从左到右</td>
   <td><code>… &lt;&nbsp;…</code></td>
  </tr>
  <tr>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/Comparison_Operators#Less_than__or_equal_operator" title="JavaScript/Reference/Operators/Comparison_Operators">小于等于</a></td>
   <td><code>… &lt;=&nbsp;…</code></td>
  </tr>
  <tr>
   <td><a href="/en-US/docs/Web/JavaScript/Reference/Operators/Comparison_Operators#Greater_than_operator" title="JavaScript/Reference/Operators/Comparison_Operators">大于</a></td>
   <td><code>… &gt;&nbsp;…</code></td>
  </tr>
  <tr>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/Comparison_Operators#Greater_than_or_equal_operator" title="JavaScript/Reference/Operators/Comparison_Operators">大于等于</a></td>
   <td><code>… &gt;=&nbsp;…</code></td>
  </tr>
  <tr>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/in" title="JavaScript/Reference/Operators/Special_Operators/in_Operator">in</a></td>
   <td><code>… in&nbsp;…</code></td>
  </tr>
  <tr>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/instanceof" title="JavaScript/Reference/Operators/Special_Operators/instanceof_Operator">instanceof</a></td>
   <td><code>… instanceof&nbsp;…</code></td>
  </tr>
  <tr>
   <td rowspan="4">11</td>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/Comparison_Operators#Equality" title="JavaScript/Reference/Operators/Comparison_Operators">等号</a></td>
   <td colspan="1" rowspan="4">从左到右<br>
    &nbsp;</td>
   <td><code>… ==&nbsp;…</code></td>
  </tr>
  <tr>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/Comparison_Operators#Inequality" title="JavaScript/Reference/Operators/Comparison_Operators">非等号</a></td>
   <td><code>… !=&nbsp;…</code></td>
  </tr>
  <tr>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/Comparison_Operators#Identity" title="JavaScript/Reference/Operators/Comparison_Operators">全等号</a></td>
   <td><code>… ===&nbsp;…</code></td>
  </tr>
  <tr>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/Comparison_Operators#Nonidentity" title="JavaScript/Reference/Operators/Comparison_Operators">非全等号</a></td>
   <td><code>… !==&nbsp;…</code></td>
  </tr>
  <tr>
   <td>10</td>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/Bitwise_Operators#Bitwise_AND" title="JavaScript/Reference/Operators/Bitwise_Operators">按位与</a></td>
   <td>从左到右</td>
   <td><code>… &amp;&nbsp;…</code></td>
  </tr>
  <tr>
   <td>9</td>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/Bitwise_Operators#Bitwise_XOR" title="JavaScript/Reference/Operators/Bitwise_Operators">按位异或</a></td>
   <td>从左到右</td>
   <td><code>… ^&nbsp;…</code></td>
  </tr>
  <tr>
   <td>8</td>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/Bitwise_Operators#Bitwise_OR" title="JavaScript/Reference/Operators/Bitwise_Operators">按位或</a></td>
   <td>从左到右</td>
   <td><code>… |&nbsp;…</code></td>
  </tr>
  <tr>
   <td>7</td>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/Logical_Operators#Logical_AND" title="JavaScript/Reference/Operators/Logical_Operators">逻辑与</a></td>
   <td>从左到右</td>
   <td><code>… &amp;&amp;&nbsp;…</code></td>
  </tr>
  <tr>
   <td>6</td>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/Logical_Operators#Logical_OR" title="JavaScript/Reference/Operators/Logical_Operators">逻辑或</a></td>
   <td>从左到右</td>
   <td><code>… ||&nbsp;…</code></td>
  </tr>
  <tr>
   <td>5</td>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing_operator" title="JavaScript/Reference/Operators/Nullish_coalescing_operator">空值合并</a></td>
   <td>从左到右</td>
   <td><code>… ?? …</code></td>
  </tr>
  <tr>
   <td>4</td>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/Conditional_Operator" title="JavaScript/Reference/Operators/Special_Operators/Conditional_Operator">条件运算符</a></td>
   <td>从右到左</td>
   <td><code>… ? … : …</code></td>
  </tr>
  <tr>
   <td rowspan="16">3</td>
   <td rowspan="16"><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/Assignment_Operators" title="JavaScript/Reference/Operators/Assignment_Operators">赋值</a></td>
   <td rowspan="16">从右到左</td>
   <td><code>… =&nbsp;…</code></td>
  </tr>
  <tr>
   <td><code>… +=&nbsp;…</code></td>
  </tr>
  <tr>
   <td><code>… -=&nbsp;…</code></td>
  </tr>
  <tr>
   <td><code>… **=&nbsp;…</code></td>
  </tr>
  <tr>
   <td><code>… *=&nbsp;…</code></td>
  </tr>
  <tr>
   <td><code>… /=&nbsp;…</code></td>
  </tr>
  <tr>
   <td><code>… %=&nbsp;…</code></td>
  </tr>
  <tr>
   <td><code>… &lt;&lt;=&nbsp;…</code></td>
  </tr>
  <tr>
   <td><code>… &gt;&gt;=&nbsp;…</code></td>
  </tr>
  <tr>
   <td><code>… &gt;&gt;&gt;=&nbsp;…</code></td>
  </tr>
  <tr>
   <td><code>… &amp;=&nbsp;…</code></td>
  </tr>
  <tr>
   <td><code>… ^=&nbsp;…</code></td>
  </tr>
  <tr>
   <td><code>… |=&nbsp;…</code></td>
  </tr>
  <tr>
   <td><code>… &amp;&amp;=&nbsp;…</code></td>
  </tr>
  <tr>
   <td><code>… ||=&nbsp;…</code></td>
  </tr>
  <tr>
   <td><code>… ??=&nbsp;…</code></td>
  </tr>
  <tr>
   <td colspan="1" rowspan="2">2</td>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/yield" title="JavaScript/Reference/Operators/yield">yield</a></td>
   <td colspan="1" rowspan="2">从右到左</td>
   <td><code>yield&nbsp;…</code></td>
  </tr>
  <tr>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/yield*" title="JavaScript/Reference/Operators/yield">yield*</a></td>
   <td><code>yield*&nbsp;…</code></td>
  </tr>
  <tr>
   <td>1</td>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/Spread_operator" title="JavaScript/Reference/Operators/Spread_operator">展开运算符</a></td>
   <td>n/a</td>
   <td><code>...</code>&nbsp;…</td>
  </tr>
  <tr>
   <td>0</td>
   <td><a href="/zh-CN/docs/Web/JavaScript/Reference/Operators/Comma_Operator" title="JavaScript/Reference/Operators/Comma_Operator">逗号</a></td>
   <td>从左到右</td>
   <td><code>… ,&nbsp;…</code></td>
  </tr>
 </tbody>
</table>

#### 短路  

&& 和 || 运算符的“短路”（short circuiting）特性。下⾯我们将对此进⾏详细介绍。

对 && 和 || 来说，如果从左边的操作数能够得出结果，就可以忽略右边的操作数。我们将这种现象称为“短路”（即执⾏最短路径）。

以 a && b 为例，如果 a 是⼀个假值，⾜以决定 && 的结果，就没有必要再判断 b 的值。同样对于 a || b ，如果 a 是⼀个真值，也⾜以决定 || 的结果，也就没有必要再判断 b 的值。  

“短路”很⽅便，也很常⽤，如：

```javascript
function doSomething(opts) {
    if (opts && opts.cool) {
    	// ..
    }
}
```

opts && opts.cool 中的 opts 条件判断如同⼀道安全保护，因为如果 opts 未赋值（或者不是⼀个对象），表达式 opts.cool 会出错。通过使⽤短路特性，opts 条件判断未通过时 opts.cool 就不会执⾏，也就不会产⽣错误！

|| 运算符也⼀样：

```javascript
function doSomething(opts) {
    if (opts.cache || primeCache()) {
    	// ..
    }
} 
```

这⾥⾸先判断 opts.cache 是否存在，如果是则⽆需调⽤primeCache() 函数，这样可以避免执⾏不必要的代码。  

### ⾃动分号  

有时 JavaScript 会⾃动为代码⾏补上缺失的分号，即⾃动分号插⼊（Automatic Semicolon Insertion，ASI）。  

### 纠错机制

是否应该完全依赖 ASI 来编码，这是 JavaScript 社区中最具争议性的话题之⼀（除此之外还有 Tab 和空格之争）。  

### 错误  

JavaScript 不仅有各种类型的运⾏时错误（TypeError 、ReferenceError 、SyntaxError 等），它的语法中也定义了⼀些编译时错误。  

#### 提前使⽤变量

ES6 规范定义了⼀个新概念，叫作 TDZ（Temporal Dead Zone，暂时性死区）。

TDZ 指的是由于代码中的变量还没有初始化⽽不能被引⽤的情况。对此，最直观的例⼦是 ES6 规范中的 let 块作⽤域：

```javascript
{
    a = 2; // ReferenceError!
    let a;
}  
```

### 函数参数  

在 ES6 中，如果参数被省略或者值为 undefined ，则取该参数的默认值：

```javascript
function foo( a = 42, b = a + 1 ) {
	console.log( a, b );
}
foo(); // 42 43
foo( undefined ); // 42 43
foo( 5 ); // 5 6
foo( void 0, 7 ); // 42 7
foo( null ); // null 1  
```

### try..finally  

try..catch 对我们来说可能已经⾮常熟悉了。但你是否知道 try 可以和 catch 或者 finally 配对使⽤，并且必要时两者可同时出现？finally 中的代码总是会在 try 之后执⾏，如果有 catch 的话则在catch 之后执⾏。也可以将 finally 中的代码看作⼀个回调函数，即⽆论出现什么情况最后⼀定会被调⽤。

如果 try 中有 return 语句会出现什么情况呢？ return 会返回⼀个值，那么调⽤该函数并得到返回值的代码是在 finally 之前还是之后执⾏呢？

```javascript
function foo() {
    try {
    	return 42;
    }
    finally {
    	console.log( "Hello" );
    }
    console.log( "never runs" );
}
console.log( foo() );
// Hello
// 42
```

这⾥ return 42 先执⾏，并将 foo() 函数的返回值设置为 42 。然后 try 执⾏完毕，接着执⾏ finally 。最后 foo() 函数执⾏完毕，console.log(..) 显⽰返回值。  

try 中的 throw 也是如此：

```javascript
function foo() {
    try {
        throw 42;
    }  

    finally {
        console.log( "Hello" );
    }
    console.log( "never runs" );
}
console.log( foo() );
// Hello
// Uncaught Exception: 42
```

如果 finally 中抛出异常（⽆论是有意还是⽆意），函数就会在此处终⽌。如果此前 try 中已经有 return 设置了返回值，则该值会被丢弃：

```javascript
function foo() {
    try {
        return 42;
    }
    finally {
        throw "Oops!";
    }
    console.log( "never runs" );
}
console.log( foo() );
// Uncaught Exception: Oops!
```

continue 和 break 等控制语句也是如此：

```javascript
for (var i=0; i<10; i++) {
    try {
        continue;
    }
    finally {
        console.log( i );
    }  
}
// 0 1 2 3 4 5 6 7 8 9
```

continue 在每次循环之后，会在 i++ 执⾏之前执⾏
console.log(i) ，所以结果是 0..9 ⽽⾮ 1..10 。  

### switch  

现在来简单介绍⼀下 switch ，可以把它看作 if..else	if..else.. 的简化版本：

```javascript
switch (a) {
    case 2:
        // 执⾏⼀些代码
        break;
    case 42:
        // 执⾏另外⼀些代码
        break;
    default:
    	// 执⾏缺省代码
}
```

这⾥ a 与 case 表达式逐⼀进⾏⽐较。如果匹配就执⾏该 case 中的代码，直到 break 或者 switch 代码块结束。

这看似并⽆特别之处，但其中存在⼀些不太为⼈所知的陷阱。  

⾸先，a 和 case 表达式的匹配算法与 === （参⻅第 4 章）相同。通常 case 语句中的 switch 都是简单值，所以这并没有问题。

然⽽，有时可能会需要通过强制类型转换来进⾏相等⽐较（即 == ，参⻅第 4 章），这时就需要做⼀些特殊处理：

```javascript
var a = "42";
switch (true) {
    case a == 10:
    	console.log( "10 or '10'" );  
    	break;
    case a == 42;
    	console.log( "42 or '42'" );
    	break;
    default:
    	// 永远执⾏不到这⾥
}
// 42 or '42'
```

除简单值以外，case 中还可以出现各种表达式，它会将表达式的结果值和 true 进⾏⽐较。因为 a == 42 的结果为 true ，所以条件成⽴。  

尽管可以使⽤ == ，但 switch 中 true 和 true 之间仍然是严格相等⽐较。即如果 case 表达式的结果为真值，但不是严格意义上的true （参⻅第 4 章），则条件不成⽴。所以，在这⾥使⽤ || 和 &&等逻辑运算符就很容易掉进坑⾥：

```javascript
var a = "hello world";
var b = 10;
switch (true) {
    case (a || b == 10):
        // 永远执⾏不到这⾥
        break;
    default:
    	console.log( "Oops" );
}
// Oops
```

因为 (a || b == 10) 的结果是 "hello world" ⽽⾮ true ，所以严格相等⽐较不成⽴。此时可以通过强制表达式返回 true 或false ，如 case !!(a || b == 10): （参⻅第 4 章）。  

最后，default 是可选的，并⾮必不可少（虽然惯例如此）。break相关规则对 default 仍然适⽤：

```javascript
var a = 10;
switch (a) {
    case 1:
    case 2:
    	// 永远执⾏不到这⾥
    default:
    	console.log( "default" );
    case 3:
        console.log( "3" );
        break;
    case 4:
    	console.log( "4" );
}
// default
// 3  
```

上例中的代码是这样执⾏的，⾸先遍历并找到所有匹配的 case ，如果没有匹配则执⾏ default 中的代码。因为其中没有 break ，所以继续执⾏已经遍历过的 case 3 代码块，直到 break 为⽌。  

## 原⽣原型  

⼀个⼴为⼈知的 JavaScript 的最佳实践是：不要扩展原⽣原型。  

当时我正在为⼀些⽹站开发⼀个嵌⼊式构件，该构件基于 jQuery（基本上所有的框架都会犯这样的错误）。基本上它在所有的⽹站上都可以运⾏，但是在某个⽹站上却彻底⽆法运⾏。  

```javascript
// Netscape 4没有Array.push
Array.prototype.push = function(item) {
	this[this.length-1] = item;
};
```

除了注释以外（谁还会关⼼ Netscape 4 呢？），上述代码似乎没有问题，是吧？

问题在于 Array.prototype.push 随后被加⼊到了规范中，并且和这段代码不兼容。标准的 push(..) 可以⼀次加⼊多个值。⽽这段代码中的 push ⽅法则只会处理第⼀个值。

⼏乎所有 JavaScript 框架的代码都使⽤ push(..) 来处理多个值。我的问题则是 CSS 选择器引擎（CSS selector）。可想⽽知其他很多地⽅也会有这样的问题。  

其次，在扩展原⽣⽅法时需要加⼊判断条件（因为你可能⽆意中覆盖了原来的⽅法）。对于前⾯的例⼦，下⾯的处理⽅式要更好⼀些：

```javascript
if (!Array.prototype.push) {
    // Netscape 4没有Array.push
    Array.prototype.push = function(item) {
    	this[this.length-1] = item;
    };
} 
```

其中，if 语句⽤来确保当 JavaScript 运⾏环境中没有 push() ⽅法时才将扩展加⼊。这应该可以解决我的问题。但它并⾮万全之策，并且存在着⼀定的隐患。

如果⽹站代码中的 push(..) 原本就不打算处理多个值的情况，那么标准的 push(..) 出台后会导致代码运⾏出错。  

那么是否应该既检测原⽣⽅法是否存在，⼜要测试它能否执⾏我们想要的功能？如果测试没通过，是不是意味着代码要停⽌执⾏？

```javascript
// 不要信任 Array.prototype.push
(function(){
    if (Array.prototype.push) {
        var a = [];
        a.push(1,2);
        if (a[0] === 1 && a[1] === 2) {
            // 测试通过，可以放⼼使⽤！
            return;
    	}
    }
    throw Error(
    	"Array#push() is missing/broken!"
    );
})();
```

理论上说这个⽅法不错，但实际上不可能为每个原⽣函数都做这样的测试。  

## shim/polyfill  

通常来说，在⽼版本的（不符合规范的）运⾏环境中扩展原⽣⽅法是唯⼀安全的，因为环境不太可能发⽣变化——⽀持新规范的新版本浏览器会完全替代⽼版本浏览器，⽽⾮在⽼版本上做扩展。

如果能够预⻅哪些⽅法会在将来成为新的标准，如Array.prototype.foobar ，那么就可以完全放⼼地使⽤当前的扩展版本，不是吗？

```javascript
if (!Array.prototype.foobar) {
    // 幼稚
    Array.prototype.foobar = function() {
    	this.push( "foo", "bar" );
    };
} 
```

如果规范中已经定义了 Array.prototype.foobar ，并且其功能和上⾯的代码类似，那就没有什么问题。这种情况⼀般称为 polyfill（或者 shim）。

polyfill 能有效地为不符合最新规范的⽼版本浏览器填补缺失的功能，让你能够通过可靠的代码来⽀持所有你想要⽀持的运⾏环境。 
