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
