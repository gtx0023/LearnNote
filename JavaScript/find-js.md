# JavaScript-call，apply
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


## JavaScript 函数 Apply

### 方法重用

通过 apply() 方法，您能够编写用于不同对象的方法。

### JavaScript apply() 方法

apply() 方法与 call() 方法非常相似：

在本例中，person 的 fullName 方法被*应用*到 person1：

#### 实例

```javascript
var person = {
    fullName: function() {
        return this.firstName + " " + this.lastName;
    }
}
var person1 = {
    firstName: "Bill",
    lastName: "Gates",
}
person.fullName.apply(person1);  // 将返回 "Bill Gates"
```

### call() 和 apply() 之间的区别

不同之处是：

call() 方法分别接受参数。

apply() 方法接受数组形式的参数。

如果要使用数组而不是参数列表，则 apply() 方法非常方便。

### 带参数的 apply() 方法

apply() 方法接受数组中的参数：

#### 实例

```javascript
var person = {
  fullName: function(city, country) {
    return this.firstName + " " + this.lastName + "," + city + "," + country;
  }
}
var person1 = {
  firstName:"John",
  lastName: "Doe"
}
person.fullName.apply(person1, ["Oslo", "Norway"]);
```

与 call() 方法对比：

#### 实例

```javascript
var person = {
  fullName: function(city, country) {
    return this.firstName + " " + this.lastName + "," + city + "," + country;
  }
}
var person1 = {
  firstName:"John",
  lastName: "Doe"
}
person.fullName.call(person1, "Oslo", "Norway");
```

### 在数组上模拟 max 方法

您可以使用 Math.max() 方法找到（数字列表中的）最大数字：

#### 实例

```javascript
Math.max(1,2,3);  // 会返回 3
```

由于 JavaScript 数组没有 max() 方法，因此您可以应用 Math.max() 方法。

#### 实例

```javascript
Math.max.apply(null, [1,2,3]); // 也会返回 3
```

第一个参数（null）无关紧要。在本例中未使用它。

这些例子会给出相同的结果：

#### 实例

```javascript
Math.max.apply(Math, [1,2,3]); // 也会返回 3
```

#### 实例

```javascript
Math.max.apply(" ", [1,2,3]); // 也会返回 3
```

### JavaScript 严格模式

在 JavaScript 严格模式下，如果 apply() 方法的第一个参数不是对象，则它将成为被调用函数的所有者（对象）。在“非严格”模式下，它成为全局对象。
