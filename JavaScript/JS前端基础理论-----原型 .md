# JS前端基础理论-----原型  

## Prototype  

> 如果包含 Proxy 的话， 我们这里对 [[Get]] 和 [[Put]] 的讨论就不适用。  

### Object.create  

对于默认的 [[Get]] 操作来说， 如果无法在对象本身找到需要的属性， 就会继续访问对象的 [[Prototype]] 链：

```javascript
var anotherObject = {
	a:2
};
// 创建一个关联到 anotherObject 的对象
var myObject = Object.create( anotherObject );
myObject.a; // 2  
```

### Object.prototype  

所有普通的 [[Prototype]] 链最终都会指向内置的 Object.prototype。 由于所有的“普通”（内置， 不是特定主机的扩展） 对象都“源于”（或者说把 [[Prototype]] 链的顶端设置为）这个 Object.prototype 对象  

#### 属性设置和屏蔽  

给一个对象设置属性并不仅仅是添加一个新属性或者修改已有的属性值。现在我们完整地讲解一下这个过程：

```javascript
myObject.foo = "bar";
```

- 如果 myObject 对象中***包含***名为 foo 的普通数据访问属性， 这条赋值语句***只会修改***已有的属性值。
- 如果 foo ***不是直接存在***于 myObject 中， **[[Prototype]] 链就会被遍历**， 类似 [[Get]] 操作。如果原型链上***找不到*** foo， foo 就会被**直接添加**到 myObject 上。
- 然而， 如果 foo 存在于原型链上层， 赋值语句 myObject.foo = "bar" 的行为就会有些不同（而且可能很出人意料）。 稍后我们会进行介绍。
- 如果属性名 foo ***既出现在 myObject 中也出现在 myObject 的 [[Prototype]] 链上层***， 那么就会发生屏蔽。 myObject 中包含的 foo 属性会**屏蔽原型链**上层的所有 foo 属性， 因为myObject.foo 总是会选择原型链中最底层的 foo 属性。



**屏蔽**比我们想象中更加复杂。 下面我们分析一下如果 foo 不直接存在于 myObject 中而是存  

在于原型链上层时 myObject.foo = "bar" 会出现的三种情况。

1. 如果在 [[Prototype]] 链上层存在名为 foo 的普通数据访问属性（参见第 3 章） 并且没有被标记为只读（writable:false）， 那就会直接在 myObject 中添加一个名为 foo 的新属性， 它是屏蔽属性。
2. 如果在 [[Prototype]] 链上层存在 foo， 但是它被标记为只读（writable:false）， 那么无法修改已有属性或者在 myObject 上创建屏蔽属性。 如果运行在严格模式下， 代码会抛出一个错误。 否则， 这条赋值语句会被忽略。 总之， 不会发生屏蔽。
3. 如果在 [[Prototype]] 链上层存在 foo 并且它是一个 setter（参见第 3 章）， 那就一定会调用这个 setter。 foo 不会被添加到（或者说屏蔽于） myObject， 也不会重新定义 foo 这个 setter。

大多数开发者都认为如果向 [[Prototype]] 链上层已经存在的属性（[[Put]]） 赋值， 就一定会触发屏蔽， 但是如你所见， 三种情况中只有一种（第一种） 是这样的。

如果你希望在第二种和第三种情况下也屏蔽 foo， 那就不能使用 = 操作符来赋值， 而是使用 Object.defineProperty(..)（参见第 3 章） 来向 myObject 添加 foo。

> 第二种情况可能是最令人意外的， 只读属性会阻止 [[Prototype]] 链下层隐式创建（屏蔽） 同名属性。 这样做主要是为了模拟类属性的继承。 你可以把原型链上层的 foo 看作是父类中的属性， 它会被 myObject 继承（复制）， 这样一来 myObject 中的 foo 属性也是只读， 所以无法创建。 但是一定要注意， 实际上并不会发生类似的继承复制（参见第 4 章和第 5 章）。 这看起来有点奇怪， myObject 对象竟然会因为其他对象中有一个只读 foo 就不能包含 foo 属性。 更奇怪的是， 这个限制只存在于 = 赋值中， 使用 Object.defineProperty(..) 并不会受到影响。  



### “类”函数  

这种奇怪的“类似类” 的行为利用了函数的一种特殊特性： 所有的函数默认都会拥有一个名为 prototype 的公有并且不可枚举（参见第 3 章） 的属性， 它会指向另一个对象：

```javascript
function Foo() {
	// ...
} 

Foo.prototype; // { }  
```

这个对象通常被称为 Foo 的原型， 因为我们通过名为 Foo.prototype 的属性引用来访问它。  

最直接的解释就是， 这个对象是在调用 new Foo()（参见第 2 章） 时创建的， 最后会被（有点武断地） 关联到这个“Foo 点 prototype” 对象上。

我们来验证一下：

```javascript
function Foo() {
// ...
}

var a = new Foo();

Object.getPrototypeOf( a ) === Foo.prototype; // true
```

调用 new Foo() 时会创建 a（具体的 4 个步骤参见第 2 章）， 其中的一步就是给 a 一个内部的 [[Prototype]] 链接， 关联到 Foo.prototype 指向的那个对象。  



### 构造函数  

```javascript
function Foo() {
	// ...
} 
Foo.prototype.constructor === Foo; // true
var a = new Foo();
a.constructor === Foo; // true
```

#### 构造函数还是调用

上一段代码很容易让人认为 Foo 是一个构造函数， 因为我们使用 new 来调用它并且看到它“构造” 了一个对象。

实际上， Foo 和你程序中的其他函数没有任何区别。 函数本身并不是构造函数， 然而， 当你在普通的函数调用前面加上 new 关键字之后， 就会把这个函数调用变成一个“构造函数调用”。 实际上， new 会劫持所有普通函数并用构造对象的形式来调用它。

举例来说：

```javascript
function NothingSpecial() {
	console.log( "Don't mind me!" );
}

var a = new NothingSpecial();
// "Don't mind me!"

a; // {}
```

NothingSpecial 只是一个普通的函数， 但是使用 new 调用时， 它就会构造一个对象并赋值给 a， 这看起来像是 new 的一个副作用（无论如何都会构造一个对象）。 这个调用是一个构造函数调用， 但是 NothingSpecial 本身并不是一个构造函数。

换句话说， 在 JavaScript 中对于“构造函数” 最准确的解释是， 所有带 new 的函数调用。

函数不是构造函数， 但是当且仅当使用 new 时， 函数调用会变成“构造函数调用”。  



```javascript
function Foo(name) {
	this.name = name;
} 
Foo.prototype.myName = function() {
	return this.name;
};
var a = new Foo( "a" );
var b = new Foo( "b" );
a.myName(); // "a"
b.myName(); // "b"
```

这段代码展示了另外两种“面向类” 的技巧：

1. this.name = name 给每个对象（也就是 a 和 b， 参见第 2 章中的 this 绑定） 都添加
   了 .name 属性， 有点像类实例封装的数据值。
2. Foo.prototype.myName = ... 可能个更有趣的技巧， 它会给 Foo.prototype 对象添加一
   个属性（函数）。 现在， a.myName() 可以正常工作， 但是你可能会觉得很惊讶， 这是什
   么原理呢？

在这段代码中， 看起来似乎创建 a 和 b 时会把 Foo.prototype 对象复制到这两个对象中，然而事实并不是这样。

在本章开头介绍默认 [[Get]] 算法时我们介绍过 [[Prototype]] 链， 以及当属性不直接存在于对象中时如何通过它来进行查找。

因此， 在创建的过程中， a 和 b 的内部 [[Prototype]] 都会关联到 Foo.prototype 上。 当 a和 b 中无法找到 myName 时， 它会（通过委托， 参见第 6 章） 在 Foo.prototype 上找到。  



### （原型） 继承  

下面这段代码使用的就是典型的“原型风格” ：

```javascript
function Foo(name) {
	this.name = name;
} 

Foo.prototype.myName = function() {
	return this.name;
};

function Bar(name,label) {
    Foo.call( this, name );
    this.label = label;
}

// 我们创建了一个新的 Bar.prototype 对象并关联到 Foo.prototype
Bar.prototype = Object.create( Foo.prototype );
// 注意！ 现在没有 Bar.prototype.constructor 了
// 如果你需要这个属性的话可能需要手动修复一下它
Bar.prototype.myLabel = function() {
	return this.label;
};

var a = new Bar( "a", "obj a" );

a.myName(); // "a"
a.myLabel(); // "obj a"  
```

这段代码的核心部分就是语句 Bar.prototype = Object.create( Foo.prototype )。 调用Object.create(..) 会凭空创建一个“新” 对象并把新对象内部的 [[Prototype]] 关联到你指定的对象（本例中是 Foo.prototype）。

换句话说， 这条语句的意思是：“创建一个新的 Bar.prototype 对象并把它关联到 Foo.prototype”。  





注意， 下面这两种方式是常见的错误做法， 实际上它们都存在一些问题：

```javascript
// 和你想要的机制不一样！
Bar.prototype = Foo.prototype;
// 基本上满足你的需求， 但是可能会产生一些副作用 :(
Bar.prototype = new Foo();
```

Bar.prototype = Foo.prototype 并不会创建一个关联到 Bar.prototype 的新对象， 它只是让 Bar.prototype 直接引用 Foo.prototype 对象。 因此当你执行类似 Bar.prototype.myLabel = ... 的赋值语句时会直接修改 Foo.prototype 对象本身。 显然这不是你想要的结果， 否则你根本不需要 Bar 对象， 直接使用 Foo 就可以了， 这样代码也会更简单一些。

Bar.prototype = new Foo() 的确会创建一个关联到 Bar.prototype 的新对象。 但是它使用了 Foo(..) 的“构造函数调用”， 如果函数 Foo 有一些副作用（比如写日志、 修改状态、 注册到其他对象、 给 this 添加数据属性， 等等） 的话， 就会影响到 Bar() 的“后代”， 后果不堪设想。

因此， 要创建一个合适的关联对象， 我们必须使用 Object.create(..) 而不是使用具有副作用的 Foo(..)。 这样做唯一的缺点就是需要创建一个新对象然后把旧对象抛弃掉， 不能直接修改已有的默认对象。  

### 检查“类”关系

假设有对象 a， 如何寻找对象 a 委托的对象（如果存在的话） 呢？ 在传统的面向类环境中，***检查一个实例（JavaScript 中的对象） 的继承祖先（JavaScript 中的委托关联） 通常被称为内省（或者反射）***。
思考下面的代码：

```javascript
function Foo() {
// ...
} 

Foo.prototype.blah = ...;
var a = new Foo();
```

我们如何通过内省找出 a 的“祖先”（委托关联） 呢？ 第一种方法是站在“类” 的角度来判断：

```javascript
a instanceof Foo; // true
```

instanceof 操作符的左操作数是一个普通的对象， 右操作数是一个函数。 instanceof 回答的问题是： 在 a 的整条 [[Prototype]] 链中是否有指向 Foo.prototype 的对象？

可惜， 这个方法只能处理对象（a） 和函数（带 .prototype 引用的 Foo） 之间的关系。 如果你想判断两个对象（比如 a 和 b） 之间是否通过 [[Prototype]] 链关联， 只用 instanceof无法实现。  



> 如果使用内置的 .bind(..) 函数来生成一个硬绑定函数（参见第 2 章） 的话，该函数是没有 .prototype 属性的。 在这样的函数上使用 instanceof 的话，目标函数的 .prototype 会代替硬绑定函数的 .prototype。
>
> 通常我们不会在“构造函数调用” 中使用硬绑定函数， 不过如果你这么做的话， 实际上相当于直接调用目标函数。 同理， 在硬绑定函数上使用instanceof 也相当于直接在目标函数上使用 instanceof。

下面这段荒谬的代码试图站在“类” 的角度使用 instanceof 来判断两个对象的关系：

```javascript
// 用来判断 o1 是否关联到（委托） o2 的辅助函数
function isRelatedTo(o1, o2) {
    function F(){}
    F.prototype = o2;
    return o1 instanceof F;
}
var a = {};
var b = Object.create( a );
isRelatedTo( b, a ); // true  
```

在 isRelatedTo(..) 内部我们声明了一个一次性函数 F， 把它的 .prototype 重新赋值并指向对象 o2， 然后判断 o1 是否是 F 的一个“实例”。 显而易见， o1 实际上并没有继承 F 也不是由 F 构造， 所以这种方法非常愚蠢并且容易造成误解。 问题的关键在于思考的角度， 强行在 JavaScript 中应用类的语义（在本例中就是使用 instanceof） 就会造成这种尴尬的局面。

**下面是第二种判断 [[Prototype]] 反射的方法**， 它更加简洁：

```javascript
Foo.prototype.isPrototypeOf( a ); // true
```

注意， 在本例中， 我们实际上并不关心（甚至不需要） Foo， 我们只需要一个可以用来判
断的对象（本例中是 Foo.prototype） 就行。 isPrototypeOf(..) 回答的问题是： 在 a 的整
条 [[Prototype]] 链中是否出现过 Foo.prototype ？

同样的问题， 同样的答案， 但是在第二种方法中并不需要间接引用函数（Foo）， 它
的 .prototype 属性会被自动访问。

我们只需要两个对象就可以判断它们之间的关系。 举例来说：

```javascript
// 非常简单： b 是否出现在 c 的 [[Prototype]] 链中？
b.isPrototypeOf( c );
```

注意， 这个方法并不需要使用函数（“类”）， 它直接使用 b 和 c 之间的对象引用来判断它
们的关系。 换句话说， 语言内置的 isPrototypeOf(..) 函数就是我们的 isRelatedTo(..) 函
数。

我们也可以**直接获取一个对象的 [[Prototype]] 链。 在 ES5 中，** 标准的方法是：

```javascript
Object.getPrototypeOf( a );
```

可以验证一下， 这个对象引用是否和我们想的一样：

```javascript
Object.getPrototypeOf( a ) === Foo.prototype; // true
```

绝大多数（不是所有！ ） 浏览器也支持一种非标准的方法来访问内部 [[Prototype]] 属性：

```javascript
a.__proto__ === Foo.prototype; // true
```

这个奇怪的 .\__proto\__（在 ES6 之前并不是标准！ ） 属性“神奇地” 引用了内部的[[Prototype]] 对象， 如果你想直接查找（甚至可以通过 .\_\_proto\_\_.\_\_ptoto\_\_... 来遍历）原型链的话， 这个方法非常有用。  

此外， .\_\_proto\_\_ 看起来很像一个属性， 但是实际上它更像一个 getter/setter（参见第 3
章）。
.\_\_proto\_\_ 的实现大致上是这样的（对象属性的定义参见第 3 章） ：

```javascript
Object.defineProperty( Object.prototype, "\_\_proto\_\_", {
    get: function() {
    	return Object.getPrototypeOf( this );
    },
    set: function(o) {
        // ES6 中的 setPrototypeOf(..)
        Object.setPrototypeOf( this, o );
        return o;
    }
} );  
```

### 对象关联  

现在我们知道了， [[Prototype]] 机制就是存在于对象中的一个内部链接， 它会引用其他对象。

通常来说， 这个链接的作用是： 如果在对象上没有找到需要的属性或者方法引用， 引擎就会继续在 [[Prototype]] 关联的对象上进行查找。   

#### 创建关联  

```javascript
var foo = {
    something: function() {
    	console.log( "Tell me something good..." );
    }
};
var bar = Object.create( foo );
bar.something(); // Tell me something good...
```

**Object.create(..) 会创建一个新对象（bar） 并把它关联到我们指定的对象（foo）**， 这样我们就可以充分发挥 [[Prototype]] 机制的威力（委托） 并且避免不必要的麻烦（比如使用 new 的构造函数调用会生成 .prototype 和 .constructor 引用）。  

#### Object.create()的polyfill代码

Object.create(..) 是在 ES5 中新增的函数， 所以在 ES5 之前的环境中（比如旧 IE） 如果要支持这个功能的话就需要使用一段简单的 polyfill 代码， 它部分实现了 Object.create(..) 的功能：

```javascript
if (!Object.create) {
    Object.create = function(o) {
        function F(){}
        F.prototype = o;
        return new F();
    };
}
```

这段 polyfill 代码使用了一个一次性函数 F， 我们通过改写它的 .prototype 属性使其指向想要关联的对象， 然后再使用 new F() 来构造一个新对象进行关联。  

