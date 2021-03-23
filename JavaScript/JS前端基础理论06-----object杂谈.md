# JS前端基础理论-----object杂谈

## 为什么 null 执行typeof null 时会返回字符串 "object"。

null 有时会被当作一种对象类型， 但是这其实只是语言本身的一个 bug， 即对 null 执行typeof null 时会返回字符串 "object"。 1 实际上， null 本身是基本类型。  

原理是这样的， 不同的对象在底层都表示为二进制， 在 JavaScript 中二进制前三位都为 0 的话会被判断为 object 类型， null 的二进制表示是全 0， 自然前三位也是 0， 所以执行 typeof 时会返回“object”。  

## 类型

对象是 JavaScript 的基础。 在 JavaScript 中一共有六种主要类型（术语是“语言类型”） ：

1. string
2. number
3. boolean
4. null
5. undefined
6. object  

## 内置对象

JavaScript 中还有一些对象子类型， 通常被称为内置对象。 有些内置对象的名字看起来和
简单基础类型一样， 不过实际上它们的关系更复杂， 我们稍后会详细介绍。

1. String
2. Number
3. Boolean
4. Object
5. Function
6. Array  
7. Date
8. RegExp
9. Error  

```javascript
var strPrimitive = "I am a string";
typeof strPrimitive; // "string"
strPrimitive instanceof String; // false
var strObject = new String( "I am a string" );
typeof strObject; // "object"
strObject instanceof String; // true
// 检查 sub-type 对象
Object.prototype.toString.call( strObject ); // [object String]
```

不过简单来说， 我们可以认为子类型在内部借用了 Object 中的 toString() 方法。 从代码中可以看到， strObject 是由 String 构造函数创建的一个对象。

原始值 "I am a string" 并不是一个对象， 它只是一个字面量， 并且是一个不可变的值。如果要在这个字面量上执行一些操作， 比如获取长度、 访问其中某个字符等， 那需要将其转换为 String 对象。

## 内容    

存储在对象容器内部的是这些属性的名称， 它们就像指针（从技术角度来说就是引用） 一样， 指向这些值真正的存储位置。
思考下面的代码：

```javascript
var myObject = {
a: 2
};
myObject.a; // 2
myObject["a"]; // 2
```

如果要访问 myObject 中 a 位置上的值， 我们需要使用 . 操作符或者 [] 操作符。

-  .a 语法通常被称为“属性访问”，
-  ["a"] 语法通常被称为“键访问”。

 实际上它们访问的是同一个位置， 并且会返回相同的值 2， 所以这两个术语是可以互换的。   



在对象中， 属性名永远都是字符串。 如果你使用 string（字面量） 以外的其他值作为属性名， 那它首先会被转换为一个字符串。 即使是数字也不例外， 虽然在数组下标中使用的的确是数字， 但是在对象属性名中数字会被转换成字符串， 所以当心不要搞混对象和数组中数字的用法：

```javascript
var myObject = { };
myObject[true] = "foo";
myObject[3] = "bar";
myObject[myObject] = "baz";

myObject["true"]; // "foo"
myObject["3"]; // "bar"
myObject["[object Object]"]; // "baz" 
```

## 可计算属性名   

ES6 增加了可计算属性名， 可以在文字形式中使用 [] 包裹一个表达式来当作属性名：

```javascript
var prefix = "foo";
var myObject = {

[prefix + "bar"]: "hello",
[prefix + "baz"]: "world"

};
myObject["foobar"]; // hello
myObject["foobaz"]; // world  
```

## Object.assign

Object.assign是ES6新添加的接口，主要的用途是用来合并多个JavaScript的对象。

Object.assign()接口可以接收多个参数，第一个参数是目标对象，后面的都是源对象，assign方法将多个原对象的属性和方法都合并到了目标对象上面，如果在这个过程中出现同名的属性（方法），后合并的属性（方法）会覆盖之前的同名属性（方法）。

assign的基本用法如下：

```javascript
var target  = {a : 1}; //目标对象
var source1 = {b : 2}; //源对象1
var source2 = {c : 3}; //源对象2
var source3 = {c : 4}; //源对象3，和source2中的对象有同名属性

Object.assign(target,source1,source2,source3);
//结果如下：
//{a:1,b:2,c:4}
```

assign的设计目的是用于合并接口的，所以它接收的第一个参数（目标）应该是对象，如果不是对象的话，它会在内部转换成对象，所以如果碰到了null或者undefined这种不能转换成对象的值的话，assign就会报错。但是如果源对象的参数位置，接收到了无法转换为对象的参数的话，会忽略这个源对象参数。

这里要看一个例子：

```javascript
const v1 = 'abc';
const v2 = true;
const v3 = 10;
const obj = Object.assign({}, v1, v2, v3);
console.log(obj); // { "0": "a", "1": "b", "2": "c" }
```

为什么会出现这个结果呢？首先，第一个参数位置接收到的是对象，所以不会报错，其次，由于字符串转换成对象时，会将字符串中每个字符作为一个属性，所以，abc三个字符作为“0”，“1”，“2”三个属性被合并了进去，但是布尔值和数值在转换对象时虽然也成功了，但是属性都是不可枚举的，所以属性没有被成功合并进去。在这里需要记住 “assign不会合并不可枚举的属性”

```javascript
Object(true) // {[[PrimitiveValue]]: true}
Object(10)  //  {[[PrimitiveValue]]: 10}
Object('abc') // {0: "a", 1: "b", 2: "c", length: 3, [[PrimitiveValue]]: "abc"}
```

上面就是对于布尔值、数值和字符串转换成对象后得到的结果。

同样，Object.assign拷贝的属性是有限制的，只会拷贝对象本身的属性（不会拷贝继承属性），也不会拷贝不可枚举的属性。但是属性名为Symbol值的属性，是可以被Object.assign拷贝的。

如果assign只接收到了一个对象作为参数的话，就是说没有源对象要合并到目标对象上，那会原样把目标对象返回。

#### 知识要点：

1. Object.assign进行的拷贝是浅拷贝。也就是说，如果拷贝过来的属性的值是对象等复合属性，那么只能拷贝过来一个引用。

   ```javascript
   const obj1 = {a: {b: 1}};
   const obj2 = Object.assign({}, obj1);
   obj1.a.b = 2;
   obj2.a.b // 2
   ```

   由于是浅拷贝，所以属性a的内部有任何变化，都会在目标对象上呈现出来。

2. Object.assign进行合并的时候，一旦碰到同名属性，就会出现覆盖现象。所以使用时务必小心。

3. Object.assign是针对Object开发的API，一旦在源对象的参数未知接收到了其他类型的参数，会尝试类型转换。如果是数组类型的话，类型转换的结果是将每个数组成员的值作为属性键值，将数组成员在数组中的位置作为属性键名。多个数组组成参数一同传入的话还会造成覆盖。具体例子如下：

   ```javascript
   Object.assign([1, 2, 3], [4, 5])
   // [4, 5, 3]
   ```

   上面代码中，assign把数组视为属性名为 0、1、2 的对象，因此源数组的 0 号属性4覆盖了目标数组的 0 号属性1。

Object.assign只能将属性值进行复制，如果属性值是一个get（取值函数）的话，那么会先求值，然后再复制。

```javascript
// 源对象
const source = {
   //属性是取值函数
   get foo(){return 1}
};

//目标对象
const target = {};
Object.assign(target,source);
//{foo ; 1}  此时foo的值是get函数的求值结果
```

Object.assign方法的常见用途    

1. 为对象添加属性

   ```javascript
   class Point{
      constructor(x,y){
         Object.assign(this,{x,y});
      }
   }
   ```

   上面的方法可以为对象Point类的实例对象添加属性x和属性y。

2. 为对象添加方法

   ```javascript
   // 方法也是对象
   // 将两个方法添加到类的原型对象上
   // 类的实例会有这两个方法
   Object.assign(SomeClass.prototype,{
       someMethod(arg1,arg2){...},
       anotherMethod(){...}
   });
   ```

   将方法添加到类的原型对象上后，类的实例能继承这两个方法。

3. 克隆对象

   ```javascript
   //克隆对象的方法
   function clone(origin){
       //获取origin的原型对象
       let originProto = Obejct.getPrototypeOf(origin);
       //根据原型对象，创建新的空对象，再assign
       return Object.assign(Object.create(originProto),origin);
   }
   ```

4. 为属性指定默认值

   ```javascript
   // 默认值对象
   const DEFAULTS = {
      logLevel : 0,
      outputFormat : 'html'
   };
   
   // 利用assign同名属性会覆盖的特性，指定默认值，如果options里有新值的话，会覆盖掉默认值
   function processContent(options){
      options = Object.assign({},DEFAULTS,options);
      console.log(options);
      //...
   }
   ```

   处于assign浅拷贝的顾虑，DEFAULTS对象和options对象此时的值最好都是简单类型的值，否则函数会失效。

## 属性描述符  

但是从 ES5 开始， 所有的属性都具备了属性描述符。
思考下面的代码：

```javascript
var myObject = {
a:2
};
Object.getOwnPropertyDescriptor( myObject, "a" );
// {
// value: 2,
// writable: true,
// enumerable: true,
// confgurable: true
// }
```

如你所见， 这个普通的对象属性对应的属性描述符（也被称为“数据描述符”， 因为它只保存一个数据值） 可不仅仅只是一个 2。 它还包含另外三个特性：

1. writable（可写）、

   ```javascript
   var myObject = {};
   Object.defineProperty( myObject, "a", {
       value: 2,
       writable: false, // 不可写！
       configurable: true,
       enumerable: true
   } );
   myObject.a = 3;
   myObject.a; // 2
   ```

   如你所见， 我们对于属性值的修改静默失败（silently failed） 了。 如果在严格模式下， 这种方法会出错：

   ```javascript
   "use strict";
   var myObject = {};
   Object.defineProperty( myObject, "a", {
       value: 2,
       writable: false, // 不可写！
       configurable: true,
       enumerable: true
   } );
   myObject.a = 3; // TypeError
   ```

   TypeError 错误表示我们无法修改一个不可写的属性。  

   

   > 可以把 writable:false 看作是属性不可改变， 相当于你定义了一个空操作 setter。 严格来说， 如果要和 writable:false 一致的话， 你的 setter 被调用时应当抛出一个 TypeError错误。  

   

2. enumerable（可枚举）

   这个描述符控制的是属性是否会出现在对象的属性枚举中， 比如说for..in 循环。 如果把 enumerable 设置成 false， 这个属性就不会出现在枚举中， 虽然仍然可以正常访问它。 相对地， 设置成 true 就会让它出现在枚举中。
   用户定义的所有的普通属性默认都是 enumerable， 这通常就是你想要的。 但是如果你不希望某些特殊属性出现在枚举中， 那就把它设置成 enumerable:false。  

   

3. configurable（可配置）。  

   只要属性是可配置的， 就可以使用 defineProperty(..) 方法来修改属性描述符：

   ```javascript
   var myObject = {
   	a:2
   };
   
   myObject.a = 3;
   myObject.a; // 3
   
   Object.defineProperty( myObject, "a", {
       value: 4,
       writable: true,
       configurable: false, // 不可配置！  
       enumerable: true
   } );
   
   myObject.a; // 4
   myObject.a = 5;
   myObject.a; // 5
   
   Object.defineProperty( myObject, "a", {
       value: 6,
       writable: true,
       configurable: true,
       enumerable: true
   } ); // TypeError
   ```

   最后一个 defineProperty(..) 会产生一个 TypeError 错误， 不管是不是处于严格模式， 尝试修改一个不可配置的属性描述符都会出错。 注意： 如你所见， 把 configurable 修改成false 是单向操作， 无法撤销！  

   > 即便属性是 configurable:false， 我们还是可以把 writable 的状态由 true 改为 false， 但是无法由 false 改为 true。  



## 不变性  

很重要的一点是， 所有的方法创建的都是浅不变形， 也就是说， 它们只会影响目标对象和它的直接属性。 如果目标对象引用了其他对象（数组、 对象、 函数， 等）， 其他对象的内容不受影响， 仍然是可变的：

```javascript
myImmutableObject.foo; // [1,2,3]
myImmutableObject.foo.push( 4 );
myImmutableObject.foo; // [1,2,3,4]
```

假设代码中的 myImmutableObject 已经被创建而且是不可变的， 但是为了保护它的内容myImmutableObject.foo， 你还需要使用下面的方法让 foo 也不可变。

在 JavaScript 程序中很少需要深不可变性。 有些特殊情况可能需要这样做，但是根据通用的设计模式， 如果你发现需要密封或者冻结所有的对象， 那你或许应当退一步， 重新思考一下程序的设计， 让它能更好地应对对象值的改变。  

1. 对象常量
   结合 writable:false 和 configurable:false 就可以创建一个真正的常量属性（不可修改、
   重定义或者删除） ：  

   ```javascript
   var myObject = {};
   Object.defineProperty( myObject, "FAVORITE_NUMBER", {
       value: 42,
       writable: false,
       configurable: false
   } );  
   ```

2. 禁止扩展
   如 果 你 想 禁 止 一 个 对 象 添 加 新 属 性 并 且 保 留 已 有 属 性， 可 以 使 用 Object.preventExtensions(..)：

   ```javascript
   var myObject = {
   	a:2
   };
   Object.preventExtensions( myObject );
   myObject.b = 3;
   myObject.b; // undefined
   ```

   在非严格模式下， 创建属性 b 会静默失败。 在严格模式下， 将会抛出 TypeError 错误。  

3. 密封

   Object.seal(..) 会创建一个“密封” 的对象， 这个方法实际上会在一个现有对象上调用Object.preventExtensions(..) 并把所有现有属性标记为 configurable:false。所以， 密封之后不仅不能添加新属性， 也不能重新配置或者删除任何现有属性（虽然可以修改属性的值）。  

4. 冻结

   Object.freeze(..) 会创建一个冻结对象， 这个方法实际上会在一个现有对象上调用Object.seal(..) 并把所有“数据访问” 属性标记为 writable:false， 这样就无法修改它们的值。

   这个方法是你可以应用在对象上的级别最高的不可变性， 它会禁止对于对象本身及其任意直接属性的修改（不过就像我们之前说过的， 这个对象引用的其他对象是不受影响的）。你可以“深度冻结” 一个对象， 具体方法为， 首先在这个对象上调用 Object.freeze(..)，然后遍历它引用的所有对象并在这些对象上调用 Object.freeze(..)。 但是一定要小心， 因为这样做有可能会在无意中冻结其他（共享） 对象。  

## Getter和Setter  

在 ES5 中可以使用 getter 和 setter 部分改写默认操作， 但是只能应用在单个属性上， 无法应用在整个对象上。 

- getter 是一个隐藏函数， 会在**获取属性**值时调用。
- setter 也是一个隐藏函数， 会在**设置属性**值时调用。

```javascript
// 定义get的两种方式，set 同理
var myObject = {
// 给 a 定义一个 getter
    get a() {
    	return 2;
    }
};
Object.defineProperty(
    myObject, // 目标对象
    "b", // 属性名
    { // 描述符
        // 给 b 设置一个 getter
        get: function(){ return this.a * 2 },
        // 确保 b 会出现在对象的属性列表中
        enumerable: true
    }
);
myObject.a; // 2
myObject.b; // 4
```

```javascript
var myObject = {
// 给 a 定义一个 getter
    get a() {
    	return 2;
    }
};
myObject.a = 3;
myObject.a; // 2
```

由于我们只定义了 a 的 getter， 所以对 a 的值进行设置时 set 操作会忽略赋值操作， 不会抛出错误。 而且即便有合法的 setter， 由于我们自定义的 getter 只会返回 2， 所以 set 操作是没有意义的。  



为了让属性更合理， 还应当定义 setter， 和你期望的一样， setter 会覆盖单个属性默认的[[Put]]（也被称为赋值） 操作。 ***通常来说 getter 和 setter 是成对出现的（只定义一个的话通常会产生意料之外的行为）*** ：

```javascript
var myObject = {
    // 给 a 定义一个 getter
    get a() {
    	return this._a_;
    },
    // 给 a 定义一个 setter
    set a(val) {
    	this._a_ = val * 2;
    }
};
myObject.a = 2;
myObject.a; // 4  
```



## 遍历  

ES5 中增加了一些数组的辅助迭代器， 包括 

- forEach(..)

  会遍历数组中的所有值并忽略回调函数的返回值。   

- every(..) 

  every(..) 会一直运行直到回调函数返回 false（或者“假” 值）  

- some(..) 

  some(..) 会一直运行直到回调函数返回 true（或者“真” 值）。  

ES6 增加了一种用来遍历数组的 

- for..of

  ```javascript
  var myArray = [ 1, 2, 3 ];
  for (var v of myArray) {
  	console.log( v );
  }
  // 1
  // 2
  // 3
  ```

  for..of 循环首先会向被访问对象请求一个迭代器对象， 然后通过调用迭代器对象的next() 方法来遍历所有返回值。
  数组有内置的 @@iterator， 因此 for..of 可以直接应用在数组上。 我们使用内置的 @@iterator 来手动遍历数组， 看看它是怎么工作的：

  ```javascript
  var myArray = [ 1, 2, 3 ];
  var it = myArray[Symbol.iterator]();
  it.next(); // { value:1, done:false }
  it.next(); // { value:2, done:false }
  it.next(); // { value:3, done:false }
  it.next(); // { done:true }  
  ```

  当然， 你可以给任何想遍历的对象定义 @@iterator， 举例来说：

  ```javascript
  var myObject = {
      a: 2,
      b: 3
  };
  Object.defineProperty( myObject, Symbol.iterator, {
      enumerable: false,
      writable: false,
      configurable: true,
      value: function() {
          var o = this;
          var idx = 0;
          var ks = Object.keys( o );
          return {
              next: function() {
                  return {
                      value: o[ks[idx++]],
                      done: (idx > ks.length)
                  };
              }
          };
      }
  } );
  // 手动遍历 myObject
  var it = myObject[Symbol.iterator]();
  it.next(); // { value:2, done:false }
  it.next(); // { value:3, done:false }
  it.next(); // { value:undefned, done:true }
  // 用 for..of 遍历 myObject
  for (var v of myObject) {
  	console.log( v );
  }
  // 2
  // 3  
  ```

  ## Object.observe(..)  

Web 前端开发 数据绑定——侦听数据对象的更新，同步这个数据的 DOM表示。多数 JavaScript 框架都为这类操作提供了某种机制。  

你可以观察的改变有 6 种类型：  

1. add
2. update
3. delete
4. reconfigure
5. setPrototype
6. preventExtensions  

默认情况下，你可以得到所有这些类型的变化的通知，也可以进行过滤只侦听关注的类型。
考虑：

```javascript
var obj = { a: 1, b: 2 };
Object.observe(
    obj,
    function(changes){
        for (var change of changes) {
            console.log( change );
        }
    },
    [ "add", "update", "delete" ]
);
obj.c = 3;
// { name: "c", object: obj, type: "add" }
obj.a = 42;
// { name: "a", object: obj, type: "update", oldValue: 1 }
delete obj.b;
// { name: "b", object: obj, type: "delete", oldValue: 2 }
```

除了主要的 "add"、 "update" 和 "delete" 变化类型：  

- 如果一个对象通过 Object.defineProperty(..) 重新配置这个对象的属性，比如修改它的 writable 属性，就会发出 "reconfigure" 改变事件。
- 如果一个对象通过 Object.preventExtensions(..) 变为不可扩展，就会发出 "prevent Extensions" 改变事件。  

### 自定义改变事件  

除了前面 6 类内置改变事件，你也可以侦听和发出自定义改变事件。  

考虑：

```javascript
function observer(changes){
    for (var change of changes) {
        if (change.type == "recalc") {
            change.object.c =
            change.object.oldValue +
            change.object.a +
            change.object.b;
        }
    }
}
function changeObj(a,b) {
    var notifier = Object.getNotifier( obj );
    obj.a = a * 2;
    obj.b = b * 3;
    // 把改变事件排到一个集合中
    notifier.notify( {
        type: "recalc",
        name: "c",
        oldValue: obj.c
    } );
}
var obj = { a: 1, b: 2, c: 3 };
Object.observe(
    obj,  
    observer,
    ["recalc"]
);
changeObj( 3, 11 );
obj.a; // 12
obj.b; // 30
obj.c; // 3
```

改变集合（"recalc" 自定义事件） 已经排入队列准备发送给观测者，但是还没有发送，因此 obj.c 的值仍然是 3。  

考虑：

```javascript
notifier.performChange( "recalc", function(){
    return {
        name: "c",
        // this就是在观察之中的对象
        oldValue: this.c
    };
} );  
```

### 结束观测  

可以通过Object.unobserve(..) 来实现。

举例来说：  

```javascript
var obj = { a: 1, b: 2 };
Object.observe( obj, function observer(changes) {
    for (var change of changes) {
        if (change.type == "setPrototype") {
            Object.unobserve(
                change.object, observer
            );  
            break;
        }
    }
} );  
```

## 

