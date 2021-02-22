# JS前端基础理论-----类理论  

其他语言中的类和 JavaScript中的“类” 并不一样。  

## JavaScript中的“类”  

JavaScript 属于哪一类呢？ 在相当长的一段时间里， JavaScript 只有一些近似类的语法元素（比如 new 和 instanceof）， 不过在后来的 ES6 中新增了一些元素， 比如 class 关键字（参见附录 A）。  

### 类的机制

在许多面向类的语言中，“标准库” 会提供 Stack 类， 它是一种“栈” 数据结构（支持压入、 弹出， 等等）。 Stack 类内部会有一些变量来存储数据， 同时会提供一些公有的可访问行为（“方法”）， 从而让你的代码可以和（隐藏的） 数据进行交互（比如添加、 删除数据）。

但是在这些语言中， 你实际上并不是直接操作 Stack（除非创建一个静态类成员引用， 这超出了我们的讨论范围）。 Stack 类仅仅是一个抽象的表示， 它描述了所有“栈” 需要做的事， 但是它本身并不是一个“栈”。 你必须先实例化 Stack 类然后才能对它进行操作。  

### 构造函数

类实例是由一个特殊的类方法构造的， 这个方法名通常和类名相同， 被称为构造函数。 这个方法的任务就是初始化实例需要的所有信息（状态）。
举例来说， 思考下面这个关于类的伪代码（编造出来的语法） ：

```javascript
class CoolGuy {
    specialTrick = nothing

    CoolGuy( trick ) {
    	specialTrick = trick
    } 

	showOff() {
    	output( "Here's my trick: ", specialTrick )
    }
}
```

我们可以调用类构造函数来生成一个 CoolGuy 实例：

```javascript
Joe = new CoolGuy( "jumping rope" )
Joe.showOff() // 这是我的绝技： 跳绳
```

注意， CoolGuy 类有一个 CoolGuy() 构造函数， 执行 new CoolGuy() 时实际上调用的就是它。 构造函数会返回一个对象（也就是类的一个实例）， 之后我们可以在这个对象上调用showOff() 方法， 来输出指定 CoolGuy 的特长。  



## 类的继承

在面向类的语言中， 你可以先定义一个类， 然后定义一个继承前者的类。  

首先回顾一下本章前面部分提出的 Vehicle 和 Car 类。 思考下面关于类继承的***伪代码***：

```javascript
class Vehicle {
    engines = 1

    ignition() {
    	output( "Turning on my engine." );
    } 

    drive() {
        ignition();
        output( "Steering and moving forward!" )
    }
}
class Car inherits Vehicle {
    wheels = 4
    
    drive() {
        inherited:drive()
        output( "Rolling on all ", wheels, " wheels!" )
    }
}

class SpeedBoat inherits Vehicle {
	engines = 2
    
    ignition() {
    	output( "Turning on my ", engines, " engines." )
    } 
    
    pilot() {
        inherited:drive()
        output( "Speeding through the water with ease!" )
    }
}  
```

接下来我们定义了两类具体的交通工具： Car 和 SpeedBoat。 它们都从 Vehicle 继承了通用的特性并根据自身类别修改了某些特性。 汽车需要四个轮子， 快艇需要两个发动机， 因此它必须启动两个发动机的点火装置。  



### 多态

Car 重写了继承自父类的 drive() 方法， 但是之后 Car 调用了 inherited:drive() 方法，这表明 Car 可以引用继承来的原始 drive() 方法。 快艇的 pilot() 方法同样引用了原始drive() 方法。

这个技术被称为***多态* ** 或者 ***虚拟多态*** 。 在本例中， 更恰当的说法是**相对多态**。  

> 在传统的面向类的语言中 super 还有一个功能，就是从子类的构造函数中通过super 可以直接调用父类的构造函数。通常来说这没什么问题，因为对于真正的类来说，构造函数是属于类的。然而，在 JavaScript 中恰好相反——实际上“类” 是属于构造函数的（类似 Foo.prototype... 这样的类型引用）。由于JavaScript 中父类和子类的关系只存在于两者构造函数对应的 .prototype 对象中，因此它们的构造函数之间并不存在直接联系，从而无法简单地实现两者的相对引用（在 ES6 的类中可以通过 super 来“解决” 这个问题，参见附录 A）。  



### 多重继承  

有些面向类的语言允许你继承多个“父类”。 多重继承意味着所有父类的定义都会被复制到子类中。

从表面上来， 对于类来说这似乎是一个非常有用的功能， 可以把许多功能组合在一起。 然而， 这个机制同时也会带来很多复杂的问题。 如果两个父类中都定义了 drive() 方法的话，子类引用的是哪个呢？ 难道每次都需要手动指定具体父类的 drive() 方法吗？ 这样多态继承的很多优点就存在了。

除此之外， 还有一种被称为钻石问题的变种。 在钻石问题中， 子类 D 继承自两个父类（B和 C）， 这两个父类都继承自 A。 如果 A 中有 drive() 方法并且 B 和 C 都重写了这个方法（多态）， 那当 D 引用 drive() 时应当选择哪个版本呢（B:drive() 还是 C:drive()） ？  



### 混入  

在继承或者实例化时， JavaScript 的对象机制并不会自动执行复制行为。 简单来说，JavaScript 中只有对象， 并不存在可以被实例化的“类”。 一个对象并不会被复制到其他对象， 它们会被关联起来  

由于在其他语言中类表现出来的都是复制行为， 因此 JavaScript 开发者也想出了一个方法来模拟类的复制行为， 这个方法就是混入。 接下来我们会看到两种类型的混入： 显式和隐式。  

#### 显式混入  

首先我们来回顾一下之前提到的 Vehicle 和 Car。 由于 JavaScript 不会自动实现 Vehicle
到 Car 的复制行为， 所以我们需要手动实现复制功能。 这个功能在许多库和框架中被称为
extend(..)， 但是为了方便理解我们称之为 mixin(..)。

```javascript
// 非常简单的 mixin(..) 例子 :
function mixin( sourceObj, targetObj ) {
    for (var key in sourceObj) {
        // 只会在不存在的情况下复制
        if (!(key in targetObj)) {
        	targetObj[key] = sourceObj[key];
        }
    }
    return targetObj;
}
var Vehicle = {
    engines: 1,
    ignition: function() {
    	console.log( "Turning on my engine." );
    },
    drive: function() {
    	this.ignition();
    	console.log( "Steering and moving forward!" );
    }
};
var Car = mixin( Vehicle, {
    wheels: 4,
    drive: function() {
        Vehicle.drive.call( this );
        console.log(
        	"Rolling on all " + this.wheels + " wheels!"
        );
    }
} );  
```

现在 Car 中就有了一份 Vehicle 属性和函数的副本了。 从技术角度来说， 函数实际上没有被复制， 复制的是函数引用。 所以， Car 中的属性 ignition 只是从 Vehicle 中复制过来的对于 ignition()   **函数的引用**  。 相反， 属性 engines 就是直接从 Vehicle 中复制了值 1。  

#### 再说多态

- 我们来分析一下这条语句： Vehicle.drive.call( this )。 这就是我所说的显式多态。
- 在之前的伪代码中对应的语句是 inherited:drive()， 我们称之为相对多态。  

**JavaScript（在 ES6 之前； 参见附录 A） 并没有相对多态的机制。** 所以， 由于 Car 和
Vehicle 中都有 drive() 函数， 为了指明调用对象， 我们必须使用绝对（而不是相对） 引
用。 我们通过名称显式指定 Vehicle 对象并调用它的 drive() 函数。

但是如果直接执行 Vehicle.drive()， 函数调用中的 this 会被绑定到 Vehicle 对象而不是
Car 对象（参见第 2 章）， 这并不是我们想要的。 因此， 我们会使用 .call(this)（参见第 2
章） 来确保 drive() 在 Car 对象的上下文中执行。  

#### 混合复制

回顾一下之前提到的 mixin(..) 函数：

```javascript
// 非常简单的 mixin(..) 例子 :
function mixin( sourceObj, targetObj ) {  

    for (var key in sourceObj) {
    // 只会在不存在的情况下复制
        if (!(key in targetObj)) {
        targetObj[key] = sourceObj[key];
        }
    } 
    return targetObj;
}  
```

#### 寄生继承

显式混入模式的一种变体被称为“寄生继承”， 它既是显式的又是隐式的， 主要推广者是
Douglas Crockford。
下面是它的工作原理：

```javascript
//“传统的 JavaScript 类” Vehicle
function Vehicle() {
	this.engines = 1;
} 
Vehicle.prototype.ignition = function() {
	console.log( "Turning on my engine." );
};
Vehicle.prototype.drive = function() {
    this.ignition();
    console.log( "Steering and moving forward!" );
};  

//“寄生类” Car
function Car() {
    // 首先， car 是一个 Vehicle
    var car = new Vehicle();
    // 接着我们对 car 进行定制
    car.wheels = 4;
    // 保存到 Vehicle::drive() 的特殊引用
    var vehDrive = car.drive;
    // 重写 Vehicle::drive()
    car.drive = function() {
        vehDrive.call( this );
        console.log(
        "Rolling on all " + this.wheels + " wheels!"
        );
    }
    return car;
}
var myCar = new Car();
myCar.drive();
// 发动引擎。
// 手握方向盘！
// 全速前进！
```

如你所见， 首先我们复制一份 Vehicle 父类（对象） 的定义， 然后混入子类（对象） 的定
义（如果需要的话保留到父类的特殊引用）， 然后用这个复合对象构建实例。  

> 调用 new Car() 时会创建一个新对象并绑定到 Car 的 this 上（参见第 2章）。但是因为我们没有使用这个对象而是返回了我们自己的 car 对象，所以最初被创建的这个对象会被丢弃，因此可以不使用 new 关键字调用 Car()。这样做得到的结果是一样的，但是可以避免创建并丢弃多余的对象。



4.4.2 隐式混入
隐式混入和之前提到的显式伪多态很像， 因此也具备同样的问题。
思考下面的代码：

```javascript
var Something = {
    cool: function() {
        this.greeting = "Hello World";
        this.count = this.count ? this.count + 1 : 1;
        }
    };  

    Something.cool();
    Something.greeting; // "Hello World"
    Something.count; // 1
    var Another = {
    cool: function() {
        // 隐式把 Something 混入 Another
        Something.cool.call( this );
    }
};
Another.cool();
Another.greeting; // "Hello World"
Another.count; // 1（count 不是共享状态）
```

通过在构造函数调用或者方法调用中使用 Something.cool.call( this )， 我们实际上“借用” 了函数 Something.cool() 并在 Another 的上下文中调用了它（通过 this 绑定； 参加第 2 章）。 最终的结果是 Something.cool() 中的赋值操作都会应用在 Another 对象上而不是Something 对象上。  

