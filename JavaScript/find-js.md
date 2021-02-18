# JavaScript重点
## JavaScript 函数 Call

### 方法重用

使用 call() 方法，您可以编写能够在不同对象上使用的方法。

### 函数是对象方法

在 JavaScript 中，函数是对象的方法。

如果一个函数不是 JavaScript 对象的方法，那么它就是全局对象的函数（参见前一章）。

下面的例子创建了带有三个属性的对象（firstName、lastName、fullName）。

#### 实例

```javascript
var person = {
    firstName:"Bill",
    lastName: "Gates",
    fullName: function () {
        return this.firstName + " " + this.lastName;
    }
}
person.fullName();
```

fullName 属性是一个*方法*。person 对象是该方法的*拥有者*。

fullName 属性属于 *person 对象的方法*。

### JavaScript call() 方法

call() 方法是预定义的 JavaScript 方法。

它可以用来调用所有者对象作为参数的方法。

通过 call()，您能够使用属于另一个对象的方法。

本例调用 person 的 fullName 方法，并用于 person1：

#### 实例

```javascript
var person = {
    fullName: function() {
        return this.firstName + " " + this.lastName;
    }
}
var person1 = {
    firstName:"Bill",
    lastName: "Gates",
}
var person2 = {
    firstName:"Steve",
    lastName: "Jobs",
}
person.fullName.call(person1);  // 将返回 "Bill Gates"
```

本例调用 person 的 fullName 方法，并用于 person2：

#### 实例

```javascript
var person = {
    fullName: function() {
        return this.firstName + " " + this.lastName;
    }
}
var person1 = {
    firstName:"John",
    lastName: "Doe",
}
var person2 = {
    firstName:"Mary",
    lastName: "Doe",
}
person.fullName.call(person2);  // 将返回 "Steve Jobs"
```

### 带参数的 call() 方法

call() 方法可接受参数：

#### 实例

```javascript
var person = {
  fullName: function(city, country) {
    return this.firstName + " " + this.lastName + "," + city + "," + country;
  }
}
var person1 = {
  firstName:"Bill",
  lastName: "Gates"
}
person.fullName.call(person1, "Seattle", "USA");
```
