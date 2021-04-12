# React this.props.children

this.props对象的属性与组件的属性一一对应，但是有一个例外，就是this.props.children属性。它表示组件的所有子节点。

```javascript
var  NotesList = React.createClass({
    render(){
        return (
            <ol>
                {
                    React.Children.map(this.props.children,function(child){
                        return <li>{child}</li>;
                    })
                }
            </ol>
        )
    }
})

ReactDOM.render(
    <NotesList>
        <span>{hello}</span>
        <span>{world}</span>
    </NotesList>,
    document.body
)
```

上面代码的NotesList组件有两个span子节点，他们都可以通过this.props.children读取。

这里需要注意，this.props.children的值有三种可能：如果当前组件没有子节点，它就是undefined;如果有一个子节点，数据类型是Object;如果有多个子节点，数据类型就是array。所以，处理this.props.children的时候要小心。

React提供一个工具方法React.Children来处理this.props.children。我们可以用React.Children.map来遍历子节点，而不用担心this.props.children的数据类型是undefined还是object。
