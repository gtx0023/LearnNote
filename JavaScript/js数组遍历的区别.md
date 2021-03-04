# [js数组遍历（for in ,for of ,map,foreach,filter）的区别](https://www.cnblogs.com/qisi007/p/9973887.html)

**一.for in 和for of 的区别**

1.for in 遍历数组时，索引实际上是字符串类型的数字，不能进行运算，我们来输出一下：

 

　　　 let arr = [1,3,5,4]

```
    for (let index in arr) {
        console.log(typeof index)  
    }
```

结果：

![img](https://img2018.cnblogs.com/blog/1412010/201811/1412010-20181117142242216-36019393.png)

2.遍历的顺序有可能不是数组内部的顺序（这个我没有试出来，如果找到例子，以后我再更新）

3.for in 会遍历数组内所有可枚举的属性，包括原型上的属性和方法

```
    let arr = [1,3,5,4]
    arr.name = "数组"
    for (let index in arr) {
        console.log(arr[index])  
    }
```

结果：

![img](https://img2018.cnblogs.com/blog/1412010/201811/1412010-20181117142711898-607935093.png)

**所以，for in更适合遍历对象，尽量不要使用for in 遍历数组**

使用for in 遍历对象时，index为该对象的键，Object[index]能取到每个键对应的值，来看下面的例子：

```
    let obj = {
        name:"张三",
        age:21,
        work:"前端"
    }
    
    for (let index in obj) {
        console.log("key为---",index,"val为---",obj[index])
    }
```

结果：

![img](https://img2018.cnblogs.com/blog/1412010/201811/1412010-20181117143528530-801662699.png)

**2.foreach**

foreach会从头到尾对数组里的每个元素遍历一遍 ，他不会生成新数组，也不改变原数组，回调函数接收三个值，分别是 数组的元素，索引和当前数组

```
    let arr = ["a","b","c","d"]
    arr.forEach((el,index,array) => {
        if(el == "b" ) return
        console.log(el,index,array)
    })
```

结果：

![img](https://img2018.cnblogs.com/blog/1412010/201811/1412010-20181119192618013-498172709.png)

在上边的例子中我加了一个判断，如果满足元素等于b,return出去，按理说遍历时满足这个条件后边就不遍历了，但是foreach不会，他会接着向下进行

 

**3.map**

和foreach类似，map也会把数组的每一项都遍历一遍，他会返回一个新数组，但是原数组保持不变，值得注意的是，map不会对空数组进行检测

 

```
let arr = [1,2,3,4,5]
let  b =  arr.map((el,val,array) => {
   return el > 2
})
console.log(b)
```

 

 结果：

![img](https://img2018.cnblogs.com/blog/1412010/201811/1412010-20181119194854839-704004411.png)

 

**4.filter**

filter为过滤的意思，也就是说它会把满足条件的元素拿出来形成一个新的数组

```
    let arr = [1,2,3,4,5,6,7,8,9]
    let result = arr.filter(el => {
        return el % 2 == 0
    })
    console.log(result)
```

结果：

![img](https://img2018.cnblogs.com/blog/1412010/201811/1412010-20181123170003163-1563013988.png)

 

 

**巧妙的运用filter去除数组中重复的元素：**

 

let phone = ['苹果','锤子','三星','华为','锤子','苹果','小米','锤子']

let result = phone.filter((el,index,arr) => {

　　return arr.indexOf(el) == index

})

console.log(result)

结果：

![img](https://img2018.cnblogs.com/blog/1412010/201811/1412010-20181123170405654-1162349529.png)
