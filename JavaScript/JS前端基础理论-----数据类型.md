# JS前端基础理论-----数据类型

## 数据类型

### 概述

JavaScript 有七种内置类 型：null 、undefined 、boolean 、number 、string 、object 和 symbol ，可以使⽤ typeof 运算符来查看。

变量没有类型，但它们持有的值有类型。类型定义了值的⾏为特征。

很多开发⼈员将 undefined 和 undeclared 混为⼀谈，但在JavaScript 中它们是两码事。undefined 是值的⼀种。undeclared则表⽰变量还没有被声明过。

遗憾的是，JavaScript 却将它们混为⼀谈，在我们试图访问"undeclared" 变量时这样报错：ReferenceError: a is notdefined，并且 typeof 对 undefined 和 undeclared 变量都返回"undefined" 。

然⽽，通过 typeof 的安全防范机制（阻⽌报错）来检查undeclared 变量，有时是个不错的办法。  



**值类型(基本类型)**：字符串（String）、数字(Number)、布尔(Boolean)、对空（Null）、未定义（Undefined）、Symbol。

**引用数据类型**：对象(Object)、数组(Array)、函数(Function)。

### JavaScript 有七种内置类型：

空值（null ）
未定义（undefined ）
布尔值（boolean ）
数字（number ）
字符串（string ）
对象（object ）
符号（symbol ，ES6 中新增）
除对象（object ）之外，其他统称为“基本类型”。  

在对变量执⾏ typeof 操作时，得到的结果并不是该变量的类型，⽽是**该变量持有的值的类型，因为 JavaScript 中的变量没有类型**。

```javascript
var a = 42;
typeof a; // "number"

a = true;
typeof a; // "boolean"  
```

### 值和类型  

JavaScript 中的变量是没有类型的，只有值才有 。变量可以随时持有任何类型的值。

换个⾓度来理解就是，JavaScript 不做“类型强制”；也就是说，语⾔引擎不要求变量 总是持有与其初始值同类型 的值。⼀个变量可以现在被赋值为字符串类型值，随后⼜被赋值为数字类型值。  

#### undefined 和 undeclared  

已在作⽤域中声明但还没有赋值的变量，是 undefined 的。相反，还没有在作⽤域中声明过的变量，是 undeclared 的。  

例如：

```javascript
var a;
a; // undefined
b; // ReferenceError: b is not defined  
```

更让⼈抓狂的是 typeof 处理 undeclared 变量的⽅式。例如：

```javascript
var a;
typeof a; // "undefined"
typeof b; // "undefined"
```

对于 undeclared（或者 not defined）变量，typeof 照样返回"undefined" 。请注意虽然 b 是⼀个 undeclared 变量，但 typeofb 并没有报错。这是因为 typeof 有⼀个特殊的安全防范机制。

#### typeof Undeclared  

问题是如何在程序中检查全局变量 DEBUG 才不会出现ReferenceError 错误。这时 typeof 的安全防范机制就成了我们的好帮⼿：

```javascript
// 这样会抛出错误
if (DEBUG) {
	console.log( "Debugging is starting" );
}
// 这样是安全的
if (typeof DEBUG !== "undefined") {
	console.log( "Debugging is starting" );
}  
```



## 内置值类型  

### 数组  

在 JavaScript 中，数组可以容纳任何类型的值，可以是字符串、数字、对象（object ），甚⾄是其他数组（多维数组就是通过这种⽅式来实现的）：  

```javascript
var a = [ 1, "2", [3] ];
a.length; // 3
a[0] === 1; // true
a[2][0] === 3; // true
```

对数组声明后即可向其中加⼊值，不需要预先设定⼤⼩

```javascript
var a = [ ];
a.length; // 0
a[0] = 1;
a[1] = "2";
a[2] = [ 3 ];
a.length; // 3  
```

> 使⽤ delete 运算符可以将单元从数组中删除，但是请注意，单元删除后，数组的 length 属性并不会发⽣变化。

#### “稀疏”数组  

```javascript
var a = [ ];
a[0] = 1;
// 此处没有设置a[1]单元
a[2] = [ 3 ];
a[1]; // undefined
a.length; // 3
```

上⾯的代码可以正常运⾏，但其中的“空⽩单元”（empty slot）可能会导致出⼈意料的结果。a[1] 的值为 undefined ，但这与将其显式赋值为 undefined （a[1] = undefined ）还是有所区别。

#### 通过数字进⾏索引  

数组通过数字进⾏索引，但有趣的是它们也是对象，所以也可以包含字符串键值和属性（但这些并不计算在数组⻓度内）：

```javascript
var a = [ ];
a[0] = 1;

a["foobar"] = 2;
a.length; // 1  

a["foobar"]; // 2
a.foobar; // 2
```

这⾥有个问题需要特别注意，如果字符串键值能够被强制类型转换为⼗进制数字的话，它就会被当作数字索引来处理。  

### 类数组  

⼯具函数 slice(..) 经常被⽤于这类转换：

```javascript
function foo() {
    var arr = Array.prototype.slice.call( arguments );
    arr.push( "bam" );
    console.log( arr );
}  

foo( "bar", "baz" ); // ["bar","baz","bam"]
```

如上所⽰，slice() 返回参数列表（上例中是⼀个类数组）的⼀个数组复本。

⽤ ES6 中的内置⼯具函数 Array.from(..) 也能实现同样的功能：

```javascript
...
var arr = Array.from( arguments );
...  
```

### 字符串  

字符串经常被当成字符数组。字符串的内部实现究竟有没有使⽤数组并不好说，但 JavaScript 中的字符串和字符数组并不是⼀回事，最多只是看上去相似⽽已。  

字符串不可变是指字符串的成员函数不会改变其原始值，⽽是创建并返回⼀个新的字符串。⽽数组的成员函数都是在其原始值上进⾏操作。  

许多数组函数⽤来处理字符串很⽅便。虽然字符串没有这些函数，但可以通过“借⽤”数组的⾮变更⽅法来处理字符串：

```javascript
a.join; // undefined
a.map; // undefined

var c = Array.prototype.join.call( a, "-" );
var d = Array.prototype.map.call( a, function(v){
	return v.toUpperCase() + ".";
} ).join( "" );

c; // "f-o-o"
d; // "F.O.O."
```

另⼀个不同点在于字符串反转（JavaScript ⾯试常⻅问题）。数组有⼀个字符串没有的可变更成员函数 reverse() ：

```javascript
a.reverse; // undefined
b.reverse(); // ["!","o","O","f"]
b; // ["f","O","o","!"]  
```

⼀个变通（破解）的办法是先将字符串转换为数组，待处理完后再将结果转换回字符串：

```javascript
var c = a
// 将a的值转换为字符数组
.split( "" )
// 将数组中的字符进⾏倒转
.reverse()
// 将数组中的字符拼接回字符串
.join( "" );
c; // "oof"  
```

### 数字  

JavaScript 没有真正意义上的整数，这也是它⼀直以来为⼈诟病的地⽅。这种情况在将来或许会有所改观，但⽬前只有数字类型。

JavaScript 中的“整数”就是没有⼩数的⼗进制数。所以 42.0 即等同于“整数”42 。  

由于数字值可以使⽤ Number 对象进⾏封装，因此数字值可以调⽤ Number.prototype 中的⽅法。例如，tofixed(..) ⽅法可指定⼩数部分的显⽰位数：

```javascript
var a = 42.59;
a.toFixed( 0 ); // "43"
a.toFixed( 1 ); // "42.6"
a.toFixed( 2 ); // "42.59"
a.toFixed( 3 ); // "42.590"
a.toFixed( 4 ); // "42.5900"
```

请注意，上例中的输出结果实际上是**给定数字的字符串形式**，如果指定的⼩数部分的显⽰位数多于实际位数就⽤ 0 补⻬。  

#### 较⼩的数值  

```javascript
0.1 + 0.2 === 0.3; // false
```

从数学⾓度来说，上⾯的条件判断应该为 true ，可结果为什么是false 呢？

简单来说，⼆进制浮点数中的 0.1 和 0.2 并不是⼗分精确，它们相加的结果并⾮刚好等于 0.3 ，⽽是⼀个⽐较接近的数字0.30000000000000004 ，所以条件判断结果为 false 。  

问题是，如果⼀些数字⽆法做到完全精确，是否意味着数字类型毫⽆⽤处呢？答案当然是否定的。

在处理带有⼩数的数字时需要特别注意。很多（也许是绝⼤多数）程序只需要处理整数，最⼤不超过百万或者万亿，此时使⽤ JavaScript的数字类型是绝对安全的。

那么应该怎样来判断 0.1 + 0.2 和 0.3 是否相等呢？最常⻅的⽅法是设置⼀个误差范围值，通常称为“机器精度”（machine epsilon），对 JavaScript 的数字来说，这个值通常是 2^-52 (2.220446049250313e-16) 。  

从 ES6 开始，该值定义在 Number.EPSILON 中，我们可以直接拿来⽤，也可以为 ES6 之前的版本写 polyfill：

```javascript
if (!Number.EPSILON) {
	Number.EPSILON = Math.pow(2,-52);
} 
```

可以使⽤ Number.EPSILON 来⽐较两个数字是否相等（在指定的误差范围内）：

```javascript
function numbersCloseEnoughToEqual(n1,n2) {
	return Math.abs( n1 - n2 ) < Number.EPSILON;
}

var a = 0.1 + 0.2;
var b = 0.3;

numbersCloseEnoughToEqual( a, b ); // true
numbersCloseEnoughToEqual( 0.0000001, 0.0000002 ); // false
```

能够呈现的最⼤浮点数⼤约是 1.798e+308 （这是⼀个相当⼤的数字），它定义在 Number.MAX_VALUE 中。最⼩浮点数定义在Number.MIN_VALUE 中，⼤约是 5e-324 ，它不是负数，但⽆限接近于 0 ！  

#### 整数检测  

要检测⼀个值是否是整数，可以使⽤ ES6 中的Number.isInteger(..) ⽅法：

```javascript
Number.isInteger( 42 ); // true
Number.isInteger( 42.000 ); // true
Number.isInteger( 42.3 ); // false
```

也可以为 ES6 之前的版本 polyfill Number.isInteger(..) ⽅法：

```javascript
if (!Number.isInteger) {
    Number.isInteger = function(num) {
    	return typeof num == "number" && num % 1 == 0;
    };
}
```

 要检测⼀个值是否是安全的整数 ，可以使⽤ ES6 中的Number.isSafeInteger(..) ⽅法：  

```javascript
Number.isSafeInteger( Number.MAX_SAFE_INTEGER ); // true
Number.isSafeInteger( Math.pow( 2, 53 ) ); // false
Number.isSafeInteger( Math.pow( 2, 53 ) - 1 ); // true
```

可以为 ES6 之前的版本 polyfill Number.isSafeInteger(..) ⽅法：

```javascript
if (!Number.isSafeInteger) {
    Number.isSafeInteger = function(num) {
        return Number.isInteger( num ) &&
        Math.abs( num ) <= Number.MAX_SAFE_INTEGER;
    };
}  
```

### 特殊数值  

#### 不是值的值  

null 指空值（empty value）
undefined 指没有值（missing value）
或者：
undefined 指从未赋值
null 指曾赋过值，但是⽬前没有值  

##### **永远不要重新定义 undefined 。**  

##### void 运算符  

#### 特殊的数字  

##### 不是数字的数字  

###### NaN

NaN 意指“不是⼀个数字”（not a number），这个名字容易引起误会，后⾯将会提到。将它理解为“⽆效数值”“失败数值”或者“坏数值”可能更准确些。  

**NaN 是 JavaScript 中唯⼀ ⼀个不等于⾃⾝的值。**  

###### isNaN(..) 有⼀个严重的缺陷  

```javascript
var a = 2 / "foo";
var b = "foo";
a; // NaN
b; "foo"
window.isNaN( a ); // true
window.isNaN( b ); // true——晕！
```

###### Number.isNaN(..)  

```javascript
if (!Number.isNaN) {
    Number.isNaN = function(n) {
        return (
            typeof n === "number" &&
            window.isNaN( n )
        );
    };
}
var a = 2 / "foo";
var b = "foo";
Number.isNaN( a ); // true
Number.isNaN( b ); // false——好！
```

##### ⽆穷数  

计算结果⼀旦溢出为⽆穷数 （infinity）就⽆法再得到有穷数。  

```javascript
var a = 1 / 0;
```

然⽽在 JavaScript 中上例的结果为 Infinity （即Number.POSITIVE_INfiNITY ）。同样：

```javascript
var a = 1 / 0; // Infinity
var b = -1 / 0; // -Infinity  
```

##### 零值  

JavaScript 有⼀个常规的 0 （也叫作 +0 ）和⼀个 -0 。

-0 除了可以⽤作常量以外，也可以是某些数学运算的返回值。例如：

```javascript
var a = 0 / -3; // -0
var b = 0 * -3; // -0
```

加法和减法运算不会得到负零（negative zero）。

负零在开发调试控制台中通常显⽰为 -0 ，但在⼀些⽼版本的浏览器中仍然会显⽰为 0 。

根据规范，对负零进⾏字符串化会返回 "0" ：  

**抛开学术上的繁枝褥节不论，我们为什么需要负零呢？**

有些应⽤程序中的数据需要以级数形式来表⽰（⽐如动画帧的移动速度），数字的符号位（sign）⽤来代表其他信息（⽐如移动的⽅向）。此时如果**⼀个值为 0 的变量失去了它的符号位，它的⽅向信息就会丢失**。所以保留 0 值的符号位可以防⽌这类情况发⽣。  

#### 特殊等式  

ES6 中新加⼊了⼀个⼯具⽅法 Object.is(..) 来判断两个值是否绝
对相等，可以⽤来处理上述所有的特殊情况：

```javascript
var a = 2 / "foo";
var b = -3 * 0;

Object.is( a, NaN ); // true
Object.is( b, -0 ); // true  
Object.is( b, 0 ); // false
```

对于 ES6 之前的版本，Object.is(..) 有⼀个简单的 polyfill：

```javascript
if (!Object.is) {
    Object.is = function(v1, v2) {
        // 判断是否是-0
        if (v1 === 0 && v2 === 0) {
        	return 1 / v1 === 1 / v2;
        }
        // 判断是否是NaN
        if (v1 !== v1) {
        	return v2 !== v2;
        }
        // 其他情况
        return v1 === v2;
    };
}  
```

