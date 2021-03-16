# JS前端基础理论-----原型-行为委托    

[[Prototype]] 机制就是指对象中的一个内部链接引用另一个对象。  

如果在**第一个对象上没有找到需要的属性或者方法引用， 引擎就会继续在 [[Prototype]]关联的对象上进行查找**。 同理， 如果在后者中也没有找到需要的引用就会继续查找它的[[Prototype]]， 以此类推。 这一系列对象的链接被称为“**原型链**”。

换句话说， JavaScript 中这个机制的本质就是**对象之间的关联关系。**  



## 行为委托。

行为委托认为对象之间是兄弟关系， 互相委托， 而不是父类和子类的关系。 JavaScript 的[[Prototype]] 机制本质上就是行为委托机制。 也就是说， 我们可以选择在 JavaScript 中努力实现类机制（参见第 4 和第 5 章）， 也可以拥抱更自然的 [[Prototype]] 委托机制。

当你只用对象来设计代码时， 不仅可以让语法更加简洁， 而且可以让代码结构更加清晰。对象关联（对象之前互相关联） 是一种编码风格， 它倡导的是直接创建和关联对象， 不把它们抽象成类。 对象关联可以用基于 [[Prototype]] 的行为委托非常自然地实现。  



## 面向委托的设计  

更直观地使用 [[Prototype]]， 我们必须认识到它代表的是一种不同于类的设计模式。  

### 类理论  

类设计模式鼓励你在继承时使用方法重写（和多态）， 比如说在 XYZ 任务中重写 Task 中定义的一些通用方法， 甚至在添加新行为时通过 super 调用这个方法的原始版本。 你会发现许多行为可以先“抽象” 到父类然后再用子类进行特殊化（重写）。  

### 委托理论  

首先你会定义一个名为 Task 的对象（和许多 JavaScript 开发者告诉你的不同， 它既不是类也不是函数）， 它会包含所有任务都可以使用（写作使用， 读作委托） 的具体行为。 接着，对于每个任务（“XYZ”、“ABC”） 你都会定义一个对象来存储对应的数据和行为。 你会把特定的任务对象都关联到 Task 功能对象上， 让它们在需要的时候可以进行委托。  

下面是推荐的代码形式， 非常简单：

```javascript
Task = {
    setID: function(ID) { this.id = ID; },
    outputID: function() { console.log( this.id ); }
};

// 让 XYZ 委托 Task
XYZ = Object.create( Task );
XYZ.prepareTask = function(ID,Label) {
    this.setID( ID );
    this.label = Label;
};

XYZ.outputTaskDetails = function() {
    this.outputID();
    console.log( this.label );
};

// ABC = Object.create( Task );
// ABC ... = ...
```

在 这 段 代 码 中， Task 和 XYZ 并 不 是 类（或 者 函 数 ）， 它 们 是 对 象。 XYZ 通 过 Object.create(..) 创建， 它的 [[Prototype]] 委托了 Task 对象（参见第 5 章）。  

相比于面向类（或者说面向对象）， 我会把这种编码风格称为“对象关联”（OLOO，objects linked to other objects）。 我们真正关心的只是 XYZ 对象（和 ABC 对象） 委托了Task 对象。

在 JavaScript 中， [[Prototype]] 机制会把对象关联到其他对象。 无论你多么努力地说服自己， **JavaScript 中就是没有类似“类” 的抽象机制。** 这有点像逆流而上： 你确实可以这么做， 但是如果你选择对抗事实， 那要达到目的就显然会更加困难。  

#### 对象关联风格的代码还有一些不同之处。

1. 在上面的代码中， id 和 label 数据成员都是直接存储在 XYZ 上（而不是 Task）。 通常来说， 在 [[Prototype]] 委托中最好把状态保存在委托者（XYZ、 ABC） 而不是委托目标（Task） 上。

2. 在类设计模式中， 我们故意让父类（Task） 和子类（XYZ） 中都有 outputTask 方法， 这样就可以利用重写（多态） 的优势。 在委托行为中则恰好相反： 我们会尽量避免在[[Prototype]] 链的不同级别中使用相同的命名， 否则就需要使用笨拙并且脆弱的语法来消除引用歧义（参见第 4 章）。

   这个设计模式要求尽量少使用容易被重写的通用方法名， 提倡使用更有描述性的方法名， 尤其是要写清相应对象行为的类型。 这样做实际上可以创建出更容易理解和维护的代码， 因为方法名（不仅在定义的位置， 而是贯穿整个代码） 更加清晰（自文档）。

3. this.setID(ID)； XYZ 中的方法首先会寻找 XYZ 自身是否有 setID(..)， 但是 XYZ 中并没有这个方法名， 因此会通过 [[Prototype]] 委托关联到 Task 继续寻找， 这时就可以找到setID(..) 方法。 此外， 由于调用位置触发了 this 的隐式绑定规则（参见第 2 章）， 因此虽然 setID(..) 方法在 Task 中， 运行时 this 仍然会绑定到 XYZ， 这正是我们想要的。在之后的代码中我们还会看到 this.outputID()， 原理相同。

换句话说， 我们和 XYZ 进行交互时可以使用 Task 中的通用方法， 因为 XYZ 委托了 Task。委托行为意味着某些对象（XYZ） 在找不到属性或者方法引用时会把这个请求委托给另一个对象（Task）。  

#### 互相委托（禁止）

你无法在两个或两个以上互相（双向） 委托的对象之间创建循环委托。 如果你把 B 关联到
A 然后试着把 A 关联到 B， 就会出错。  

#### 调试

这段传统的“类构造函数” JavaScript 代码在 Chrome 开发者工具的控制台中结果如下所示：

```javascript
function Foo() {}
var a1 = new Foo();
a1; // Foo {}
```

我们看代码的最后一行： 表达式 a1 的输出是 Foo {}。 如果你在 Firefox 中运行同样的代码会得到 Object {}。

Chrome 实际上想说的是“{} 是一个空对象， 由名为 Foo 的函数构造”。 Firefox 想说的是“{}是一个空对象， 由 Object 构造”。 之所以有这种细微的差别， 是因为 Chrome 会动态跟踪并把实际执行构造过程的函数名当作一个内置属性， 但是其他浏览器并不会跟踪这些额外的信息。  

### 比较思维模型  

我们会通过一些示例（Foo、 Bar） 代码来比较一下两种设计模式（面向对象和对象关联）具体的实现方法。 下面是典型的（“原型”） 面向对象风格：

```javascript
function Foo(who) {
	this.me = who;
} 
Foo.prototype.identify = function() {
	return "I am " + this.me;
};

function Bar(who) {
	Foo.call( this, who );
} 
Bar.prototype = Object.create( Foo.prototype );

Bar.prototype.speak = function() {
	alert( "Hello, " + this.identify() + "." );
};

var b1 = new Bar( "b1" );
var b2 = new Bar( "b2" );

b1.speak();
b2.speak();
```

子类 Bar 继承了父类 Foo， 然后生成了 b1 和 b2 两个实例。 b1 委托了 Bar.prototype， 后者委托了 Foo.prototype。 这种风格很常见， 你应该很熟悉了。  

下面我们看看如何使用对象关联风格来编写功能完全相同的代码：

```javascript
Foo = {
    init: function(who) {
        this.me = who;
    },
    identify: function() {
        return "I am " + this.me;
    }
};
Bar = Object.create( Foo );

Bar.speak = function() {
	alert( "Hello, " + this.identify() + "." );
};

var b1 = Object.create( Bar );
b1.init( "b1" );
var b2 = Object.create( Bar );  
b2.init( "b2" );

b1.speak();
b2.speak();
```

这段代码中我们同样利用 [[Prototype]] 把 b1 委托给 Bar 并把 Bar 委托给 Foo， 和上一段代码一模一样。 我们仍然实现了三个对象之间的关联。  

## 类与对象  

### 控件“类”

你可能已经习惯了面向对象设计模式， 所以很快会想到一个包含所有通用控件行为的父类（可能叫作 Widget） 和继承父类的特殊控件子类（比如 Button）。  

下面这段代码展示的是如何在不使用任何“类” 辅助库或者语法的情况下， 使用纯JavaScript 实现类风格的代码：

```javascript
// 父类
function Widget(width,height) {
    this.width = width || 50;
    this.height = height || 50;
    this.$elem = null;
} 

Widget.prototype.render = function($where){
    if (this.$elem) {
        this.$elem.css( {
            width: this.width + "px",
            height: this.height + "px"
        } ).appendTo( $where );
    }
};

// 子类
function Button(width,height,label) {
// 调用“super” 构造函数
    Widget.call( this, width, height );
    this.label = label || "Default";
    this.$elem = $( "<button>" ).text( this.label );
} 

// 让 Button“继承” Widget
Button.prototype = Object.create( Widget.prototype );

// 重写 render(..)
Button.prototype.render = function($where) {
    //“super” 调用
    Widget.prototype.render.call( this, $where );
    this.$elem.click( this.onClick.bind( this ) );
};

Button.prototype.onClick = function(evt) {
	console.log( "Button '" + this.label + "' clicked!" );  
};

$( document ).ready( function(){
    var $body = $( document.body );
    var btn1 = new Button( 125, 30, "Hello" );
    var btn2 = new Button( 150, 40, "World" );
    btn1.render( $body );
    btn2.render( $body );
} );
```

在面向对象设计模式中我们需要先在父类中定义基础的 render(..)， 然后在子类中重写它。 子类并不会替换基础的 render(..)， 只是添加一些按钮特有的行为。

可以看到代码中出现了丑陋的显式伪多态（参见第 4 章）， 即通过 Widget.call 和 Widget.prototype.render.call 从“子类” 方法中引用“父类” 中的基础方法。

**ES6的class语法糖**  

附录 A 会详细介绍 ES6 的 class 语法糖， 不过这里可以简单介绍一下如何使用 class 来实
现相同的功能：

```javascript
class Widget {
    constructor(width,height) {
        this.width = width || 50;
        this.height = height || 50;
        this.$elem = null;
    } 
    render($where){
        if (this.$elem) {
            this.$elem.css( {
                width: this.width + "px",
                height: this.height + "px"
            } ).appendTo( $where );
        }
    }
}
class Button extends Widget {
    constructor(width,height,label) {
        super( width, height );
        this.label = label || "Default";
        this.$elem = $( "<button>" ).text( this.label );
    } 
    render($where) {
        super( $where );
        this.$elem.click( this.onClick.bind( this ) );
    } 
    onClick(evt) {
    	console.log( "Button '" + this.label + "' clicked!" );
    }
}  

$( document ).ready( function(){
    var $body = $( document.body );
    var btn1 = new Button( 125, 30, "Hello" );
    var btn2 = new Button( 150, 40, "World" );
    btn1.render( $body );
    btn2.render( $body );
} );
```

毫无疑问， 使用 ES6 的 class 之后， 上一段代码中许多丑陋的语法都不见了， super(..)函数棒极了。（尽管深入探究就会发现并不是那么完美！ ）  

### 委托控件对象

下面的例子使用对象关联风格委托来更简单地实现 Widget/Button：  

```javascript
var Widget = {
    init: function(width,height){
        this.width = width || 50;
        this.height = height || 50;
        this.$elem = null;
    },
    insert: function($where){
        if (this.$elem) {
            this.$elem.css( {
                width: this.width + "px",
                height: this.height + "px"
            } ).appendTo( $where );
        }
    }
};

var Button = Object.create( Widget );

Button.setup = function(width,height,label){
// 委托调用
    this.init( width, height );
    this.label = label || "Default";
    this.$elem = $( "<button>" ).text( this.label );
};

Button.build = function($where) {  
    // 委托调用
    this.insert( $where );
    this.$elem.click( this.onClick.bind( this ) );
};
Button.onClick = function(evt) {
	console.log( "Button '" + this.label + "' clicked!" );
};

$( document ).ready( function(){
    var $body = $( document.body );
    var btn1 = Object.create( Button );
    btn1.setup( 125, 30, "Hello" );
    var btn2 = Object.create( Button );
    btn2.setup( 150, 40, "World" );
    btn1.build( $body );
    btn2.build( $body );
} );  
```

使用对象关联风格来编写代码时不需要把 Widget 和 Button 当作父类和子类。 相反，Widget 只是一个对象， 包含一组通用的函数， 任何类型的控件都可以委托， Button 同样只是一个对。（当然， 它会通过委托关联到 Widget ！ ）

从设计模式的角度来说， 我们并没有像类一样在两个对象中都定义相同的方法名render(..)， 相反， 我们定义了两个更具描述性的方法名（insert(..) 和 build(..)）。 同理， 初始化方法分别叫作 init(..) 和 setup(..)。

在委托设计模式中， 除了建议使用不相同并且更具描述性的方法名之外， 还要通过对象关联避免丑陋的显式伪多态调用（Widget.call 和 Widget.prototype.render.call）， 代之以简单的相对委托调用 this.init(..) 和 this.insert(..)。

从语法角度来说， 我们同样没有使用任何构造函数、 .prototype 或 new， 实际上也没必要使用它们。

如果你仔细观察就会发现， 之前的一次调用（var btn1 = new Button(..)） 现在变成了两次（var btn1 = Object.create(Button) 和 btn1.setup(..)）。 乍一看这似乎是一个缺点（需要更多代码）。  

**对象关联可以更好地支持关注分离（separation of concerns） 原则**， 创建和初始化并不需要
合并为一个步骤。  

## 更简洁的设计  

场景中我们有两个控制器对象， 一个用来操作网页中的登录表单， 另一个用来与服务器进行验证（通信）。  

### 传统的类设计模式

我们会把基础的函数定义在名为 Controller 的类中， 然后派生两个子类 LoginController 和 AuthController， 它们都继承自 Controller 并且重写了一些基础行为：

```javascript
// 父类
function Controller() {
	this.errors = [];
} 
Controller.prototype.showDialog(title,msg) {
// 给用户显示标题和消息
};
Controller.prototype.success = function(msg) {
	this.showDialog( "Success", msg );
};
Controller.prototype.failure = function(err) {
    this.errors.push( err );
    this.showDialog( "Error", err );
};

// 子类
function LoginController() {
	Controller.call( this );
} 
// 把子类关联到父类  

LoginController.prototype = Object.create( Controller.prototype );
LoginController.prototype.getUser = function() {
	return document.getElementById( "login_username" ).value;
};
LoginController.prototype.getPassword = function() {
	return document.getElementById( "login_password" ).value;
};
LoginController.prototype.validateEntry = function(user,pw) {
    user = user || this.getUser();
    pw = pw || this.getPassword();
    if (!(user && pw)) {
        return this.failure(
        	"Please enter a username & password!"
        );
    }
    else if (user.length < 5) {
        return this.failure(
        	"Password must be 5+ characters!"
        );
    } 
    // 如果执行到这里说明通过验证
	return true;
};
// 重写基础的 failure()
LoginController.prototype.failure = function(err) {
    //“super” 调用
    Controller.prototype.failure.call(
        this,
        "Login invalid: " + err
    );
};

// 子类
function AuthController(login) {
    Controller.call( this );
    // 合成
    this.login = login;
} 
// 把子类关联到父类
AuthController.prototype = Object.create( Controller.prototype );
AuthController.prototype.server = function(url,data) {
    return $.ajax( {
        url: url,
        data: data
    } );
};
AuthController.prototype.checkAuth = function() {
    var user = this.login.getUser();
    var pw = this.login.getPassword();
    if (this.login.validateEntry( user, pw )) {  
        this.server( "/check-auth",{
            user: user,
            pw: pw
        } )
        .then( this.success.bind( this ) )
        .fail( this.failure.bind( this ) );
    }
};
// 重写基础的 success()
AuthController.prototype.success = function() {
    //“super” 调用
    Controller.prototype.success.call( this, "Authenticated!" );
};
// 重写基础的 failure()
AuthController.prototype.failure = function(err) {
    //“super” 调用
    Controller.prototype.failure.call(
        this,
        "Auth Failed: " + err
    );
};

var auth = new AuthController();
auth.checkAuth(
	// 除了继承， 我们还需要合成
	new LoginController()
);
```

所 有 控 制 器 共 享 的 基 础 行 为 是 success(..)、 failure(..) 和 showDialog(..)。 子 类LoginController 和 AuthController 通过重写 failure(..) 和 success(..) 来扩展默认基础类行为。 此外， 注意 AuthController 需要一个 LoginController 的实例来和登录表单进行交互， 因此这个实例变成了一个数据属性。

另一个需要注意的是我们在继承的基础上进行了一些合成。 AuthController 需要使用LoginController， 因此我们实例化后者（new LoginController()） 并用一个类成员属性this.login 来引用它， 这样 AuthController 就可以调用 LoginController 的行为。  

### 反类

但是， 我们真的需要用一个 Controller 父类、 两个子类加上合成来对这个问题进行建模吗？ 能不能使用对象关联风格的行为委托来实现更简单的设计呢？ 当然可以！

```javascript
var LoginController = {
    errors: [],
    getUser: function() {
        return document.getElementById(
            "login_username"
        ).value;
    },
    getPassword: function() {
        return document.getElementById(
            "login_password"
        ).value;
    },
    validateEntry: function(user,pw) {
        user = user || this.getUser();
        pw = pw || this.getPassword();
        if (!(user && pw)) {
            return this.failure(
                "Please enter a username & password!"
            );
        }
        else if (user.length < 5) {
            return this.failure(
                "Password must be 5+ characters!"
            );
        } 
        // 如果执行到这里说明通过验证
        return true;
    },
    showDialog: function(title,msg) {
    // 给用户显示标题和消息
    },
    failure: function(err) {
        this.errors.push( err );
        this.showDialog( "Error", "Login invalid: " + err );
    }
};
// 让 AuthController 委托 LoginController
var AuthController = Object.create( LoginController );
AuthController.errors = [];
AuthController.checkAuth = function() {
    var user = this.getUser();
    var pw = this.getPassword();
    if (this.validateEntry( user, pw )) {
        this.server( "/check-auth",{
            user: user,  
            pw: pw
        } )
        .then( this.accepted.bind( this ) )
        .fail( this.rejected.bind( this ) );
    }
};
AuthController.server = function(url,data) {
    return $.ajax( {
        url: url,
        data: data
    } );
};
AuthController.accepted = function() {
	this.showDialog( "Success", "Authenticated!" )
};
AuthController.rejected = function(err) {
	this.failure( "Auth Failed: " + err );
};
```

由于 AuthController 只是一个对象（LoginController 也一样）， 因此我们不需要实例化（比如 new AuthController()）， 只需要一行代码就行：

```javascript
AuthController.checkAuth();
```

借助对象关联， 你可以简单地向委托链上添加一个或多个对象， 而且同样不需要实例化：

```javascript
var controller1 = Object.create( AuthController );
var controller2 = Object.create( AuthController );
```

在行为委托模式中， AuthController 和 LoginController 只是对象， 它们之间是兄弟关系，并不是父类和子类的关系。 代码中 AuthController 委托了 LoginController， 反向委托也完全没问题。

这种模式的重点在于只需要两个实体（LoginController 和 AuthController）， 而之前的模式需要三个。

我们不需要 Controller 基类来“共享” 两个实体之间的行为， 因为委托足以满足我们需要的功能。 同样， 前面提到过， 我们也不需要实例化类， 因为它们根本就不是类， 它们只是对象。 此外， 我们也不需要合成， 因为两个对象可以通过委托进行合作。

最后， 我们避免了面向类设计模式中的多态。 我们在不同的对象中没有使用相同的函数名 success(..) 和 failure(..)， 这样就不需要使用丑陋的显示伪多态。 相反， 在AuthController 中它们的名字是 accepted(..) 和 rejected(..)——可以更好地描述它们的行为。

总结： 我们用一种（极其） 简单的设计实现了同样的功能， 这就是对象关联风格代码和行为委托设计模式的力量。  

## 更好的语法  

### class

ES6 的 class 语法可以简洁地定义类方法， 这个特性让 class 乍看起来更有吸引力（附录A 会介绍为什么要避免使用这个特性） ：

```javascript
class Foo {
	methodName() { /* .. */ }
}  
```

现在我们来看看ES6 的 class 机制  

首先回顾一下第 6 章中的 Widget/Button 例子：

```javascript
class Widget {
    constructor(width,height) {
        this.width = width || 50;  
        this.height = height || 50;
        this.$elem = null;
    } 
    render($where){
        if (this.$elem) {
            this.$elem.css( {
                width: this.width + "px",
                height: this.height + "px"
            } ).appendTo( $where );
        }
    }
} 

class Button extends Widget {
    constructor(width,height,label) {
        super( width, height );
        this.label = label || "Default";
        this.$elem = $( "<button>" ).text( this.label );
    } 
    render($where) {
        super( $where );
        this.$elem.click( this.onClick.bind( this ) );
    } 
    onClick(evt) {
    	console.log( "Button '" + this.label + "' clicked!" );
    }
}
```

除了语法更好看之外， ES6 还解决了什么问题呢？

1. （基本上， 下面会详细介绍） 不再引用杂乱的 .prototype 了。
2. Button 声 明 时 直 接“继 承 ” 了 Widget， 不 再 需 要 通 过 Object.create(..) 来 替换 .prototype 对象， 也不需要设置 .__proto__ 或者 Object.setPrototypeOf(..)。
3. 可以通过 super(..) 来实现相对多态， 这样任何方法都可以引用原型链上层的同名方法。 这可以解决第 4 章提到过的那个问题： 构造函数不属于类， 所以无法互相引用——super() 可以完美解决构造函数的问题。
4. class 字面语法不能声明属性（只能声明方法）。 看起来这是一种限制， 但是它会排除掉许多不好的情况， 如果没有这种限制的话， 原型链末端的“实例” 可能会意外地获取其他地方的属性（这些属性隐式被所有“实例” 所“共享”）。 所以， class 语法实际上可以帮助你避免犯错。
5. 可以通过 extends 很自然地扩展对象（子） 类型， 甚至是内置的对象（子） 类型， 比如Array 或 RegExp。 没有 class ..extends 语法时， 想实现这一点是非常困难的， 基本上只有框架的作者才能搞清楚这一点。 但是现在可以轻而易举地做到！

平心而论， class 语法确实解决了典型原型风格代码中许多显而易见的（语法） 问题和缺点。  

#### class陷阱  

首先， 你可能会认为 ES6 的 class 语法是向 JavaScript 中引入了一种新的“类” 机制， 其实不是这样。 class 基本上只是现有 [[Prototype]]（委托！ ） 机制的一种语法糖。

#####  class 不会在声明时静态复制所有行为

也就是说， class 并不会像传统面向类的语言一样在声明时静态复制所有行为。 如果你（有意或无意） 修改或者替换了父“类” 中的一个方法， 那子“类” 和所有实例都会受到影响， 因为它们在定义时并没有进行复制， 只是使用基于 [[Prototype]] 的实时委托：

```javascript
class C {
    constructor() {
        this.num = Math.random();
    } 
    rand() {
    	console.log( "Random: " + this.num );
    }
} 

var c1 = new C();
c1.rand(); // "Random: 0.4324299..."

C.prototype.rand = function() {
	console.log( "Random: " + Math.round( this.num * 1000 ));
};

var c2 = new C();
c2.rand(); // "Random: 867"

c1.rand(); // "Random: 432" ——噢！
```

如果你已经明白委托的原理所以并不会期望得到“类” 的副本的话， 那这种行为才看起来比较合理。 所以你需要问自己： 为什么要使用本质上不是类的 class 语法呢？  

##### class 语法无法定义类成员属性

class 语法无法定义类成员属性（只能定义方法）， 如果为了跟踪实例之间共享状态必须要
这么做， 那你只能使用丑陋的 .prototype 语法， 像这样：

```javascript
class C {
    constructor() {
        // 确保修改的是共享状态而不是在实例上创建一个屏蔽属性！
        C.prototype.count++;
        // this.count 可以通过委托实现我们想要的功能
        console.log( "Hello: " + this.count );
    }  
}

 // 直接向 prototype 对象上添加一个共享状态
C.prototype.count = 0;

var c1 = new C();
// Hello: 1

var c2 = new C();
// Hello: 2

c1.count === 2; // true
c1.count === c2.count; // true
```

这 种 方 法 最 大 的 问 题 是， 它 违 背 了 class 语 法 的 本 意， 在 实 现 中 暴 露（泄 露！ ）了 .prototype。

如果使用 this.count++ 的话， 我们会很惊讶地发现在对象 c1 和 c2 上都创建了 .count 属性， 而不是更新共享状态。 class 没有办法解决这个问题， 并且干脆就不提供相应的语法支持， 所以你根本就不应该这样做。  

##### class 语法仍然面临意外屏蔽  

```javascript
class C {
    constructor(id) {
        // 噢， 郁闷， 我们的 id 属性屏蔽了 id() 方法
        this.id = id;
    } 
    id() {
    	console.log( "Id: " + id );
    }
}
var c1 = new C( "c1" );
c1.id(); // TypeError -- c1.id 现在是字符串 "c1"
```

##### super 并不是动态绑定的

 你可能认为 super 的绑定方法和 this 类似（参见第 2 章）， 也就是说， 无论目前的方法在原型链中处于什么位置， super 总会绑定到链中的上一层。

然而， 出于性能考虑**（this 绑定已经是很大的开销了）， super 并不是动态绑定的**， 它会在声明时“静态” 绑定。 

  思考下面代码中 super 的行为（D 和 E 上） ：

```javascript
class P {
	foo() { console.log( "P.foo" ); }
}

class C extends P {
    foo() {
        super();
    }
}

var c1 = new C();
c1.foo(); // "P.foo"

var D = {
	foo: function() { console.log( "D.foo" ); }
};

var E = {
	foo: C.prototype.foo
};

// 把 E 委托到 D
Object.setPrototypeOf( E, D );

E.foo(); // "P.foo"
```

如果你认为 super 会动态绑定（非常合理！ ）， 那你可能期望 super() 会自动识别出 E 委托了 D， 所以 E.foo() 中的 super() 应该调用 D.foo()。

但事实并不是这样。 出于性能考虑， super 并不像 this 一样是晚绑定（late bound， 或者说动态绑定） 的， 它在 [[HomeObject]].[[Prototype]] 上， [[HomeObject]] 会在创建时静态绑定。

在本例中， super() 会调用 P.foo()， 因为方法的 [[HomeObject]] 仍然是 C， C.[[Prototype]]是 P。  

### 简洁方法声明

在 ES6 中 我 们 可 以 在 任 意 对 象 的 字 面 形 式 中 使 用 简 洁 方 法 声 明（concise method declaration）， 所以对象关联风格的对象可以这样声明（和 class 的语法糖一样） ：

```javascript
var LoginController = {
    errors: [],
    getUser() { // 妈妈再也不用担心代码里有 function 了！
    	// ...
    },
    getPassword() {
    	// ...
    } 
    // ...
};
```

唯一的区别是对象的字面形式仍然需要使用“,” 来分隔元素， 而 class 语法不需要。 这个区别对于整体的设计来说无关紧要。  

此外， 在 ES6 中， 你可以使用对象的字面形式（这样就可以使用简洁方法定义） 来改 写 之 前 繁 琐 的 属 性 赋 值 语 法（比 如 AuthController 的 定 义 ）， 然 后 用 Object.setPrototypeOf(..) 来修改它的 [[Prototype]]：

```javascript
// 使用更好的对象字面形式语法和简洁方法
var AuthController = {
    errors: [],
    checkAuth() {
        // ...
    },
    server(url,data) {
        // ...
    } 
    // ...
};
// 现在把 AuthController 关联到 LoginController
Object.setPrototypeOf( AuthController, LoginController );  
```

使用 ES6 的简洁方法可以让对象关联风格更加人性化（并且仍然比典型的原型风格代码更加简洁和优秀）。 你完全不需要使用类就能享受整洁的对象语法！  

### 反词法

简洁方法有一个非常小但是非常重要的缺点。 思考下面的代码：

```javascript
var Foo = {
    bar() { /*..*/ },
    baz: function baz() { /*..*/ }
};
```

去掉语法糖之后的代码如下所示：

```javascript
var Foo = {
    bar: function() { /*..*/ },
    baz: function baz() { /*..*/ }
};  
```

看 到 区 别 了 吗？ 由 于 函 数 对 象 本 身 没 有 名 称 标 识 符， 所 以 bar() 的 缩 写 形 式
（function()..） 实际上会变成一个匿名函数表达式并赋值给 bar 属性。 相比之下， 具名函
数表达式（function baz()..） 会额外给 .baz 属性附加一个词法名称标识符 baz。  

匿名函数没有 name 标识符， 这会导致：

1. 调试栈更难追踪；
2. 自我引用（递归、 事件（解除） 绑定， 等等） 更难；
3. 代码（稍微） 更难理解。

简洁方法没有第 1 和第 3 个缺点。  

简洁方法无法避免第 2 个缺点， 它们不具备可以自我引用的词法标识符。 

思考下面的代码：

```javascript
var Foo = {
    bar: function(x) {
        if(x<10){
            return Foo.bar( x * 2 );
        }  
        return x;
    },
    baz: function baz(x) {
        if(x < 10){
            return baz( x * 2 );
        }
        return x;
    }
};
```

在本例中使用 Foo.bar(x*2) 就足够了， 但是在许多情况下无法使用这种方法， 比如多个对象通过代理共享函数、 使用 this 绑定， 等等。 这种情况下最好的办法就是使用函数对象的name 标识符来进行真正的自我引用。

使用简洁方法时一定要小心这一点。 如果你需要自我引用的话， 那最好使用传统的具名函数表达式来定义对应的函数（· baz: function baz(){..}· ）， 不要使用简洁方法。  

## 内省  

自省就是检查实例的类型。 类实例的自省主要目的是通过创建方式来判断对象的结构和功能。  

下面的代码使用 instanceof（参见第 5 章） 来推测对象 a1 的功能：

```javascript
function Foo() {
	// ...
} 
Foo.prototype.something = function(){
	// ...
}

var a1 = new Foo();
// 之后
if (a1 instanceof Foo) {
	a1.something();
}
```

因 为 Foo.prototype（不 是 Foo ！ ） 在 a1 的 [[Prototype]] 链 上（参 见 第 5 章 ）， 所 以instanceof 操作（会令人困惑地） 告诉我们 a1 是 Foo“类” 的一个实例。 知道了这点后，我们就可以认为 a1 有 Foo“类” 描述的功能。

当然， Foo 类并不存在， 只有一个普通的函数 Foo， 它引用了 a1 委托的对象（Foo.prototype）。 从语法角度来说， instanceof 似乎是检查 a1 和 Foo 的关系， 但是实际上它想说的是 a1 和 Foo.prototype（引用的对象） 是互相关联的。  



现在回到本章想说的对象关联风格代码， 其内省更加简洁。 我们先来回顾一下之前的 Foo/Bar/b1 对象关联例子（只包含关键代码） ：

```javascript
var Foo = { /* .. */ };

var Bar = Object.create( Foo );
Bar...

var b1 = Object.create( Bar );
```

使用对象关联时， 所有的对象都是通过 [[Prototype]] 委托互相关联， 下面是内省的方法，非常简单：

```javascript
// 让 Foo 和 Bar 互相关联
Foo.isPrototypeOf( Bar ); // true
Object.getPrototypeOf( Bar ) === Foo; // true

// 让 b1 关联到 Foo 和 Bar
Foo.isPrototypeOf( b1 ); // true
Bar.isPrototypeOf( b1 ); // true
Object.getPrototypeOf( b1 ) === Bar; // true
```

我们没有使用 instanceof， 因为它会产生一些和类有关的误解。 现在我们想问的问题是“你是我的原型吗？ ” 我们并不需要使用间接的形式， 比如 Foo.prototype 或者繁琐的 Foo.prototype.isPrototypeOf(..)。  

