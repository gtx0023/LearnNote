# React  基础

# JSX 语法  

## DOM 元素  

现在需要描述⼀个按钮（button），这⽤ HTML 语法表⽰⾮常简单：  

```html
<button class="btn btn-blue">
	<em>Confirm</em>
</button>
```

如果转成 JSON 对象

```json
{
    type: 'button',
    props: {
        className: 'btn btn-blue',
        children: [{
            type: 'em',
            props: {
            	children: 'Confirm'
            }
        }]
    }
}
```

组件元素  就可以⽅便地调⽤ Button({color: 'blue',text: 'Confirm'}) 来创建。  

```javascript
const Button = ({ color, text }) => {
    return {
        type: 'button',
        props: {
            className: `btn btn-${color}`,
            children: {
                type: 'em',
                props: {
                	children: text,
                },
            },
        },
    };
}
```



```javascript
const DangerButton = ({ text }) => ({
    type: Button,
    props: {
        color: 'red',
        children: text
    }
});

const DeleteAccount = () => ({
	type: 'div',
    props: {
        children: [
            {
                type: 'p',
                props: {
                    children: 'Are you sure?',
                },
            }, 
            {
                type: DangerButton,
                props: {
                	children: 'Confirm',
                },
            }, 
            {
                type: Button,
                props: {
                    color: 'blue',
                    children: 'Cancel',
                },
        	}
        ],
    }
});
```

假如我们使⽤ JSX 语法来重新表达上述组件元素，只需这么写：  

```javascript
const DeleteAccount = () => (
    <div>
        <p>Are you sure?</p>
        <DangerButton>Confirm</DangerButton>
        <Button color="blue">Cancel</Button>
    </div>
);
```

JSX 在产品打包阶段都已经编译成纯 JavaScript  

React 官⽅在早期为 JSX 语法解析开发了⼀套编译器 JSTransform，⽬前已经不再维护，现在已全部采
⽤ Babel 的 JSX 编译器实现。因为两者在功能上完全重复，⽽ Babel 作为专门的 JavaScript 语法编译
⼯具，提供了更为强⼤的功能，达到了“⼀处配置，统⼀运⾏”的⽬的。  

## JSX 基本语法  

### XML 基本语法  

- 定义标签时，只允许被⼀个标签包裹。例如，const component = \<span>name\</span>\<span> value\</span> 这样写会报错。原因是⼀个标签会被转译成对应的React.createElement 调⽤⽅法，最外层没有被包裹，显然⽆法转译成⽅法调⽤。

- 标签⼀定要闭合。所有标签（⽐如\<div>\</div>、\<p>\</p>）都必须闭合，否则⽆法编译通过。其中HTML 中⾃闭合的标签（如 \<img>）在 JSX 中也遵循同样规则，⾃定义标签可以根据是否有⼦组件或⽂本来决定闭合⽅式。

当然，JSX 报错机制⾮常强⼤，如果有拼写错误时，可以直接在控制台打印出来。  

### 元素类型  

DOM 元素和组件元素  

在 JSX ⾥⾃然会有对应，对应规则是
HTML 标签⾸字⺟是否为⼩写字⺟，其中⼩写⾸字⺟对应 DOM 元素，⽽组件元素⾃然对应⼤写⾸字
⺟。  

### 元素属性  

在 JSX 中，不论是 DOM 元素还是组件元素，它们都有属性。不同的是，DOM 元素的属性是标准规范属性，但有两个例外——class 和 for，这是因为在 JavaScript 中这两个单词都是关键词。因此，我们这么转换：

- class 属性改为 className；
- for 属性改为 htmlFor。  

此外，还有⼀些 JSX 特有的属性表达。  

- Boolean 属性  

  - 省略 Boolean 属性值会导致 JSX 认为 bool 值设为了 true。要传false 时，必须使⽤属性表达
    式。这常⽤于表单元素中，⽐如 disabled、required、checked 和 readOnly 等。  

- 展开属性  

  如果事先知道组件需要的全部属性，JSX 可以这样来写：

  ```javascript
  const component = <Component name={name} value={value} />;
  ```

  如果你不知道要设置哪些 props，那么现在最好不要设置它：

  ```javascript
  const component = <Component />;
  component.props.name = name;
  component.props.value = value;
  ```

  上述这样是反模式，因为 React 不能帮你检查属性类型（propTypes）。这样即使组件的属性类型
  有错误，也不能得到清晰的错误提⽰。
  这⾥，可以使⽤ ES6 rest/spread 特性来提⾼效率：

  ```javascript
  const data = { name: 'foo', value: 'bar' };
  const component = <Component name={data.name} value={data.value} />;
  ```

  可以写成：

  ```javascript
  const data = { name: 'foo', value: 'bar' };
  const component = <Component {...data} />;  
  ```

  

- ⾃定义 HTML 属性  

### JavaScript 属性表达式  

属性值要使⽤表达式，只要⽤ {} 替换 "" 即可：

```javascript
// 输⼊（JSX）：
const person = <Person name={window.isLoggedIn ? window.name : ''} />;
// 输出（JavaScript）：
const person = React.createElement(
    Person,
    {name: window.isLoggedIn ? window.name : ''}
);
```

⼦组件也可以作为表达式使⽤：

```javascript
// 输⼊（JSX）：
const content = <Container>{window.isLoggedIn ? <Nav /> : <Login />}</Container>;
// 输出（JavaScript）：
const content = React.createElement(
    Container,
    null,
    window.isLoggedIn ? React.createElement(Nav) : React.createElement(Login)
);  
```

### HTML 转义  

React 会将所有要显⽰到 DOM 的字符串转义，防⽌ XSS。所以，如果 JSX 中含有转义后的实体字符，⽐如 ©（©），则最后 DOM 中不会正确显⽰，因为 React ⾃动把 © 中的特殊字符转义了。有⼏种解决办法：

- 直接使⽤ UTF-8 字符 ©；
- 使⽤对应字符的 Unicode 编码查询编码；
- 使⽤数组组装 \<div>{['cc ', \<span>©\</span>, ' 2015']}\</div>；
- 直接插⼊原始的 HTML。

此外，React 提供了 dangerouslySetInnerHTML 属性。正如其名，它的作⽤就是避免 React 转义字符，在确定必要的情况下可以使⽤它：  



# React 组件  

## 初步的组件

- 基本的封装性。尽管说 JavaScript 没有真正⾯向对象的⽅法，但我们还是可以通过实例化的⽅法来制造对象。
- 简单的⽣命周期呈现。最明显的两个⽅法constructor 和 destroy，代表了组件的挂载和卸载过程。但除此之外，其他过程（如更新时的⽣命周期）并没有体现。
- 明确的数据流动。这⾥的数据指的是调⽤组件的参数。⼀旦确定参数的值，就会解析传进来的参数，根据参数的不同作出不同的响应，从⽽得到渲染结果。  

## React 与 Web Components

从 React 组件上看，它与 Web Components 传达的理念是⼀致的，但两者的实现⽅式不同：

- React ⾃定义元素是库⾃⼰构建的，与 Web Components 规范并不通⽤；
- React 渲染过程包含了模板的概念，即 1.2 节所讲的 JSX；
- React 组件的实现均在⽅法与类中，因此可以做到相互隔离，但不包括样式；
- React 引⽤⽅式遵循 ES6 module 标准。

可以说，React 还是在纯 JavaScript 上下了⼯夫，将 HTML 结构彻底引⼊到 JavaScript 中。尽管这种做法褒贬不⼀，但也有效解决了组件所要解决的问题之⼀。  

## React 组件的构建⽅法  

### React.createClass  

```javascript
const Button = React.createClass({
    getDefaultProps() {
        return {
            color: 'blue',
            text: 'Confirm',
        };
    },
	render() {
        const { color, text } = this.props;
        return (
            <button className={`btn btn-${color}`}>
            <em>{text}</em>
            </button>
        );
    }
});
```

从表象上看，React.createClass ⽅法就是构建⼀个组件对象。当另⼀个组件需要调⽤ Button组件时，只⽤写成 \<Button />，就可以被解析成 React.createElement(Button) ⽅法来创建 Button 实例，这意味着在⼀个应⽤中调⽤⼏次 Button，就会创建⼏次 Button 实例。  

### ES6 classes  

ES6 classes 的写法是通过 ES6 标准的类语法的⽅式来构建⽅法：

```javascript
import React, { Component } from 'react';

class Button extends Component {
    constructor(props) {
        super(props);
    }
    static defaultProps = {
        color: 'blue',
        text: 'Confirm',
    };
    render() {
        const { color, text } = this.props;
        return (
            <button className={`btn btn-${color}`}>
            <em>{text}</em>
            </button>
        );
    }
} 
```

这⾥的直观感受是从调⽤内部⽅法变成了⽤类来实现。与 createClass 的结果相同的是，调⽤类实现的组件会创建实例对象。  

说明 React 的所有组件都继承⾃顶层类 React.Component。它的定义⾮常简洁，只是初始化了 React.Component ⽅法，声明了props、context、refs 等，并在原型上定义了setState 和 forceUpdate ⽅法。内部初始化的⽣命周期⽅法与 createClass ⽅式使⽤的是同⼀个⽅法创建的。  

### ⽆状态函数  

使⽤⽆状态函数构建的组件称为⽆状态组件，这种构建⽅式是 0.14 版本之后新增的，且官⽅颇为推崇。⽰例代码如下：

```
function Button({ color = 'blue', text = 'Confirm' }) {
    return (
        <button className={`btn btn-${color}`}>
        	<em>{text}</em>
        </button>
    );
} 
```

⽆状态组件只传⼊ props 和 context 两个参数；也就是说，它不存在 state，也没有⽣命周期⽅法，组件本⾝即上⾯两种 React 组件构建⽅法中的 render ⽅法。不过，像 propTypes 和defaultProps 还是可以通过向⽅法设置静态属性来实现的。

在适合的情况下，我们都应该且必须使⽤⽆状态组件。⽆状态组件不像上述两种⽅法在调⽤时会创建新实例，它创建时始终保持了⼀个实例，避免了不必要的检查和内存分配，做到了内部优化。  

# React 数据流

在 React 中，数据是⾃顶向下单向流动的，即从⽗组件到⼦组件。这条原则让组件之间的关系变得简单且可预测。
state 与 props 是 React 组件中最重要的概念。如果顶层组件初始化 props，那么 React 会向下遍历整棵组件树，重新尝试渲染所有相关的⼦组件。⽽ state 只关⼼每个组件⾃⼰内部的状态，这些状态只能在组件内改变。把组件看成⼀个函数，那么它接受了 props 作为参数，内部由 state 作为函数的内部参数，返回⼀个 Virtual DOM 的实现。  

## state  

当组件内部使⽤库内置的 setState ⽅法时，最⼤的表现⾏为就是该组件会尝试重新渲染。  

```
import React, { Component } from 'react';
class Counter extends Component {
    constructor(props) {
        super(props);
        this.handleClick = this.handleClick.bind(this);
        this.state = {
            count: 0,
        };
    }
    handleClick(e) {
        e.preventDefault();
        this.setState({
            count: this.state.count + 1,
        });
    }
    render() {
        return (
            <div>
                <p>{this.state.count}</p>
                <a href="#" onClick={this.handleClick}>更新</a>
            </div>
        );
    }
} 
```

在React 中常常在事件处理⽅法中更新 state，上述例⼦就是通过点击“更新”按钮不断地更新内部count 的值，这样就可以把组件内状态封装在实现中。  

值得注意的是，setState 是⼀个异步⽅法，⼀个⽣命周期内所有的 setState ⽅法会合并操作。  



这⾥我们针对 activeIndex 作为 state，就有两种不同的视⾓。

- activeIndex 在内部更新。当我们切换 tab 标签时，可以看作是组件内部的交互⾏为，被选择后通过回调函数返回具体选择的索引。
- activeIndex 在外部更新。当我们切换 tab 标签时，可以看作是组件外部在传⼊具体的索引，⽽组件就像“⽊偶”⼀样被操控着。

这两种情形在 React 组件的设计中⾮常常⻅，我们形象地把第⼀种和第⼆种视⾓写成的组件分别称为智能组件（smart component）和⽊偶组件（dumb component）。  

```
constructor(props) {
    super(props);
    
    const currProps = this.props;
    
    let activeIndex = 0;
    
    if ('activeIndex' in currProps) {
        activeIndex = currProps.activeIndex;
    } else if ('defaultActiveIndex' in currProps) {
        activeIndex = currProps.defaultActiveIndex;
    }
    
    this.state = {
        activeIndex,
        prevIndex: activeIndex,
    };
} 
```

这⾥我们定义了两种 state——activeIndex 和 prevIndex。  



## props  

我们在 1.4.1 节中讨论了 Tabs 组件 state 的设置情况。假设 Tabs 组件的数据都是通过 data prop 传⼊的，即 \<Tabs data={data} />。那么，Tabs 组件的 props 还会有哪些。根据之前的经验，它⼀定会有以下⼏项。

- className：根节点的 class。为了⽅便覆盖其原始样式，我们都会在根节点上定义class，这⼀点会在 2.3 节中详细说明。
- classPrefix：class 前缀。对于组件来说，定义⼀个统⼀的 class 前缀，对样式与交互分离起了⾮常重要的作⽤。
- defaultActiveIndex 和 activeIndex：默认的激活索引，这在 1.4.1 节中已说明。
- onChange：回调函数。当我们切换 tab 时，外组件需要知道组件内部的信息，尤其是当前 tab 的索引号的信息。它⼀般与 activeIndex 搭配使⽤。  

### ⼦组件 prop  

```
<Tabs classPrefix={'tabs'} defaultActiveIndex={0} className="tabs-bar"
    children={[  
        <TabPane key={0} tab={'Tab 1'}>第⼀个 Tab ⾥的内容</TabPane>,
        <TabPane key={1} tab={'Tab 2'}>第⼆个 Tab ⾥的内容</TabPane>,
        <TabPane key={2} tab={'Tab 3'}>第三个 Tab ⾥的内容</TabPane>,
    ]}
\>
</Tabs>  
```

实现的基本思路就以 TabContent 组件渲染 TabPane ⼦组件集合为例来讲，其中渲染 TabPane 组件的
⽅法如下：

```
getTabPanes() {
    const { classPrefix, activeIndex, panels, isActive } = this.props;
    
    return React.Children.map(panels, (child) => {
        if (!child) { return; }
        
        const order = parseInt(child.props.order, 10);
        const isActive = activeIndex === order;
        
        return React.cloneElement(child, {
            classPrefix,
            isActive,
            children: child.props.children,
            key: `tabpane-${order}`,
        });
    });
}  
```



### 组件 props  

这就是 component props。对于⼦组件⽽⾔，我们不仅可以直接使⽤this.props.children 定义，也可以将⼦组件以 props 的形式传递。  

在 Tabs 组件中，我们就⽤到了这样的功能，调⽤⽅式如下所⽰：

```
<Tabs classPrefix={'tabs'} defaultActiveIndex={0} className="tabs-bar">
    <TabPane
        order="0"
        tab={<span><i className="fa fa-home"></i>&nbsp;Home</span>}>
        第⼀个 Tab ⾥的内容
    </TabPane>
    
    <TabPane
        order="1"
        tab={<span><i className="fa fa-book"></i>&nbsp;Library</span>}>
        第⼆个 Tab ⾥的内容
    </TabPane>
    
    <TabPane
        order="2"
        tab={<span><i className="fa fa-pencil"></i>&nbsp;Applications</span>}>
        第三个 Tab ⾥的内容
    </TabPane>
</Tabs>
```



### ⽤ function prop 与⽗组件通信  

现在我们发现对于 state 来说，它的通信集中在组件内部；对于 props 来说，它的通信是⽗组件向⼦组件的传播。相关代码如下：

```
handleTabClick(activeIndex) {
    // ...
    this.props.onChange({activeIndex, prevIndex});
} 
```

我们通过点击事件 handleTabClick 触发了 onChange prop 回调函数给⽗组件必要的值。对于兄弟组件或不相关组件之间的通信。  



### propTypes

propTypes ⽤于规范 props 的类型与必需的状态。如果组件定义了 propTypes，那么在开发环境下，
就会对组件的 props 值的类型作检查，如果传⼊的 props 不能与之匹配，React 将实时在控制台⾥报
warning。在⽣产环境下，这是不会进⾏检查的。  

现在，我们先来看 Tabs 组件的propTypes：

```
static propTypes = {
    classPrefix: React.PropTypes.string,
    className: React.PropTypes.string,
    defaultActiveIndex: React.PropTypes.number,
    activeIndex: React.PropTypes.number,
    onChange: React.PropTypes.func,
    children: React.PropTypes.oneOfType([
        React.PropTypes.arrayOf(React.PropTypes.node),
        React.PropTypes.node,
    ]),
};
```

我们很清晰地列举了所有可能的 props，并对它们的类型进⾏定义。再来看看 TabPane 组件的propTypes：

```
static propTypes = {
    tab: React.PropTypes.oneOfType([
        React.PropTypes.string,
        React.PropTypes.node,
    ]).isRequired,
    order: React.PropTypes.string.isRequired,
    disable: React.PropTypes.bool,
};  
```



# React ⽣命周期

我们可以把 React ⽣命周期分成两类：

- 当组件在挂载或卸载时；
- 当组件接收新的数据时，即组件更新时。  

## 挂载或卸载过程  

### 组件的挂载  

组件挂载是最基本的过程，这个过程主要做组件状态的初始化。我们推荐以下⾯的例⼦为模板写初始化组件：

```
import React, { Component, PropTypes } from 'react';

class App extends Component {
    static propTypes = {
    // ...
    };
    
    static defaultProps = {
    // ...
    };
    
    constructor(props) {
        super(props);
        this.state = {
        // ...
        };
    }
    
    componentWillMount() {
    // ...
    }
    
    componentDidMount() {
    // ...
    }
    
    render() {
    	return <div>This is a demo.</div>;
    }
}  
```

我们看到 propTypes 和 defaultProps 分别代表 props 类型检查和默认类型。这两个属性被声明成静态属性，意味着从类外⾯也可以访问它们，⽐如可以这么访问：App.propTypes 和App.defaultProps。

之后会看到两个明显的⽣命周期⽅法，其中 componentWillMount ⽅法会在 render ⽅法之前执⾏，⽽ componentDidMount ⽅法会在 render ⽅法之后执⾏，分别代表了渲染前后的时刻。

这个初始化过程没什么特别的，包括读取初始 state 和 props 以及两个组件⽣命周期⽅法componentWillMount 和 componentDidMount，这些都只会在组件初始化时运⾏⼀次。

如果我们在 componentWillMount 中执⾏ setState ⽅法，会发⽣什么呢？组件会更新 state，但组件只渲染⼀次。因此，这是⽆意义的执⾏，初始化时的 state 都可以放在this.state。

如果我们在 componentDidMount 中执⾏ setState ⽅法，⼜会发⽣什么呢？组件当然会再次更新，不过在初始化过程就渲染了两次组件，这并不是⼀件好事。但实际情况是，有⼀些场景不得不需要setState，⽐如计算组件的位置或宽⾼时，就不得不让组件先渲染，更新必要的信息后，再次渲染。



### 组件的卸载  

组件卸载⾮常简单，只有 componentWillUnmount 这⼀个卸载前状态：

```
import React, { Component, PropTypes } from 'react';

class App extends Component {
    componentWillUnmount() {
    // ...
    }
    
    render() {
    	return <div>This is a demo.</div>;
    }
} 
```

在componentWillUnmount ⽅法中，我们常常会执⾏⼀些清理⽅法，如事件回收或是清除定时器。  



## 数据更新过程  

更新过程指的是⽗组件向下传递 props 或组件⾃⾝执⾏ setState ⽅法时发⽣的⼀系列更新动作。这⾥
我们屏蔽了初始化的⽣命周期⽅法，以便观察更新过程的⽣命周期：

```
import React, { Component, PropTypes } from 'react';

class App extends Component {
    componentWillReceiveProps(nextProps) {
    	// this.setState({})
    }
    shouldComponentUpdate(nextProps, nextState) {
    	// return true;
    }
    componentWillUpdate(nextProps, nextState) {
    	// ...
    }
    componentDidUpdate(prevProps, prevState) {
    	// ...
    }
    render() {
    	return <div>This is a demo.</div>;
    }
} 
```

如果组件⾃⾝的 state 更新了，那么会依次执⾏ shouldComponentUpdate、componentWillUpdate、render 和 componentDidUpdate。

shouldComponentUpdate 是⼀个特别的⽅法，它接收需要更新的 props 和 state，让开发者增加必要的条件判断，让其在需要时更新，不需要时不更新。因此，当⽅法返回 false 的时候，组件不再向下执⾏⽣命周期⽅法。

shouldComponentUpdate 的本质是⽤来进⾏正确的组件渲染。怎么理解呢？我们需要先从初始化组件的过程开始说起，假设有如图 1-8 所⽰的组件关系，它呈三级的树状结构，其中空⼼圆表⽰已经渲染的节点。  



### 初始化渲染结构

当⽗节点 props 改变的时候，在理想情况下，只需渲染在⼀条链路上有相关 props 改变的节点即可  

### props 改变时 React 节点的渲染路径

⽽默认情况下，React 会渲染所有的节点，因为 shouldComponentUpdate 默认返回 true。正确的组件渲染从另⼀个意义上说，也是性能优化的⼿段之⼀。  

### ⽆状态组件

值得注意的是，⽆状态组件是没有⽣命周期⽅法的，这也意味着它没有 shouldComponentUpdate。

**渲染到该类组件时，每次都会重新渲染。**当然，不少开发者在使⽤⽆状态组件时会纠结这⼀点。为了更放⼼地使⽤，我们可以选择引⽤ Recompose 库的 pure ⽅法：

```
const OptimizedComponent = pure(ExpensiveComponent);  
```

- componentWillUpdate ⽅法提供需要更新的 props 和 state，
- componentDidUpdate 提供更新前的 props 和 state。  
- 不能在 componentWillUpdate 中执⾏ setState。  

- 如果组件是由⽗组件更新 props ⽽更新的，那么在 shouldComponentUpdate 之前会先执⾏componentWillReceiveProps ⽅法。此⽅法可以作为 React 在 props 传⼊后，渲染之前setState 的机会。在此⽅法中调⽤ setState 是不会⼆次渲染的。  



# React 与 DOM

## ReactDOM

ReactDOM 中的 API ⾮常少，只有 findDOMNode、unmountComponentAtNode 和 render。  

### findDOMNode  

当组件被渲染到 DOM 中后，findDOMNode 返回该 React 组件实例相应的 DOM 节点。它可以⽤于获取表单的 value 以及⽤于 DOM 的测量。  

```
import React, { Component } from 'react';
import ReactDOM from 'react-dom';
class App extends Component {
    componentDidMount() {
        // this 为当前组件的实例
        const dom = ReactDOM.findDOMNode(this);
    }
    render() {}
}
```

### render  

要把 React 渲染的 Virtual DOM渲染到浏览器的 DOM 当中，就要使⽤ render ⽅法了：  

```
ReactComponent render(
    ReactElement element,
    DOMElement container,
    [function callback]
)
```



## ReactDOM 的不稳定⽅法  

这⾥我们引⼊ Portal 组件，这是⼀个经典的实现，最初的实现来源于 React Bootstrap 组件库中的
Overlay Mixin，后来使⽤越来越⼴泛。我们截取关键部分的源码：

```
import React from 'react';
import ReactDOM, { findDOMNode } from 'react-dom';
import CSSPropertyOperations from 'react/lib/CSSPropertyOperations';

export default class Portal extends React.Component {
    constructor() {
        // ...
    }
    openPortal(props = this.props) {
        this.setState({ active: true });
        this.renderPortal(props);
        this.props.onOpen(this.node);
    }
    closePortal(isUnmounted = false) {
        const resetPortalState = () => {
            if (this.node) {
                ReactDOM.unmountComponentAtNode(this.node);
                document.body.removeChild(this.node);
            }
            this.portal = null;
            this.node = null;
            if (isUnmounted !== true) {
                this.setState({ active: false });
            }
        };
        if (this.state.active) {
            if (this.props.beforeClose) {
                this.props.beforeClose(this.node, resetPortalState);
            } else {
                resetPortalState();
            }
            this.props.onClose();
        }
    }
    renderPortal(props) {
        if (!this.node) {
            this.node = document.createElement('div');
            // 在节点增加到 DOM 之前，执⾏ CSS 防⽌⽆效的重绘
            this.applyClassNameAndStyle(props);
            document.body.appendChild(this.node);
        } else {
            // 当新的 props 传下来的时候，更新 CSS
            this.applyClassNameAndStyle(props);
        }
        let children = props.children;
        // https://gist.github.com/jimfb/d99e0678e9da715ccf6454961ef04d1b
        if (typeof props.children.type === 'function') {
            children = React.cloneElement(props.children, { closePortal: this.closePortal });
        }
        this.portal = ReactDOM.unstable_renderSubtreeIntoContainer(
            this,
            children,
            this.node,
            this.props.onUpdate
        );
    }
    render() {
        if (this.props.openByClickOn) {
            return React.cloneElement(this.props.openByClickOn, { onClick: this.handleWrapperClick });
        }
        return null;
    }
} 
```

从Portal 组件可以看出，我们实现了⼀个“壳”，其中包括触发事件、渲染的位置以及暴露的⽅法，但它并不关⼼⼦组件的内容。当我们使⽤它的时候，可以这么写：

```
<Portal ref="myPortal">
    <Modal title="My modal">
    	Modal content
    </Modal>
</Portal>  
```



## refs  

refs 就是为此⽽⽣的，它是 React 组件中⾮常特殊的 prop，可以附加到任何⼀个组件上。从字⾯意思来看，refs 即 reference，组件被调⽤时会新建⼀个该组件的实例，⽽ refs 就会指向这个实例。  

```
import React, { Component } from 'react';
class App extends Component {
    componentDidMount() {
        // myComp 是 Comp 的⼀个实例，因此需要⽤ findDOMNode 转换为相应的 DOM
        const myComp = this.refs.myComp;
        const dom = findDOMNode(myComp);
    }
    render() {
        return (
            <div>
                <Comp ref="myComp" />
            </div>
        );
    }
}
```

- 可以使⽤ refs 来获取你拥有的⼦组件的引⽤。
- 当卸载⼀个组件的时候，组件⾥所有的 refs 就会变为 null。
- findDOMNode 和 refs 都⽆法⽤于⽆状态组件中。⽆状态组件挂载时只是⽅法调⽤，没有新建实例。  

