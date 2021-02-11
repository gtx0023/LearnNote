# React Eslint 报错解决合集

## 提示错误：'React' must be in scope when using JSX react/react-in-jsx-scope

错误提示：在JSX作用域中使用JSX React/React时，“React”必须在作用域中。是因为React并未导入
此时要把：

```javascript
import { Component } from "react";
```

改为：
```javascript
import React,{ Component } from "react";
```



## 提示错误：max-len

任何语言的非常长的代码行都很难阅读。为了提高可读性和可维护性，许多编码人员已经制定了将代码行限制为 X 个字符（传统上为80个字符）的约定。

```javascript
var foo = { "bar": "This is a bar.", "baz": { "qux": "This is a qux" }, "difficult": "to read" }; // very long
```

### 规则细节

此规则强制执行最大行长度以增加代码的可读性和可维护性。一行的长度定义为该行中的 Unicode 字符数。

### 选项

此规则有一个数字或对象选项：

- **`"code"`（默认`80`）强制执行最大行长度**

- `"tabWidth"`（默认`4`）指定制表符的字符宽度

- `"comments"`为注释强制执行最大行长度; 默认值为`code` 

- `"ignorePattern"`忽略与正则表达式匹配的行; 只能匹配单行，并且在使用 YAML 或 JSON 编写时需要双重转义

- **`"ignoreComments": true` 忽略他们自己的行中的所有尾随评论和评论**

- `"ignoreTrailingComments": true` 只会忽略尾随评论

- `"ignoreUrls": true` 忽略包含 URL 的行

- `"ignoreStrings": true` 忽略包含双引号或单引号字符串的行

- `"ignoreTemplateLiterals": true` 忽略包含模板文字的行

- `"ignoreRegExpLiterals": true` 忽略包含 RegExp 文字的行 

### 代码

```javascript
/*eslint max-len: ["error", 80]*/

var foo = {
  "bar": "This is a bar.",
  "baz": { "qux": "This is a qux" },
  "easier": "to read"
};
```



## 提示错误：new-cap

`new`JavaScript中的运算符创建特定类型对象的新实例。该类型的对象由构造函数表示。由于构造函数只是常规函数，因此唯一的定义特征是`new`作为调用的一部分使用。原生JavaScript函数以大写字母开头，以区分将用作构造函数的那些函数与不用的函数。许多样式指南建议遵循此模式，以便更轻松地确定哪些函数将用作构造函数。

```javascript
var friend = new Person();
```

### 规则细节

此规则要求构造函数名以大写字母开头。某些内置标识符可免除此规则。这些标识符是：

- `Array`

- `Boolean`

- `Date`

- `Error`

- `Function`

- `Number`

- `Object`

- `RegExp`

- `String`

- `Symbol`

此规则的**正确**代码示例：

```javascript
/*eslint new-cap: "error"*/

function foo(arg) {
    return Boolean(arg);
}
```



## 提示错误：import/no-extraneous-dependencies 引入内置模块path报错问题解决

一般配置了**import/no-extraneous-dependencies**，
就会进行模块引入的检查

```javascript
	'import/no-extraneous-dependencies': [2, {
      devDependencies: true,
      peerDependencies: true,
      // optionalDependencies: true,
      // bundledDependencies: true
    }]
```



## 提示错误：Use path.join() or path.resolve() instead of + to create paths.(no-path-concat)



### 在require中使用.

如果在dir2/pathtest.js中调用了require方法，去引入位于dir1目录的js文件，你需要写成

require('../pathtest.js')

因为require中的路径总是相对于包含它的文件，跟你的工作目录没有关系。

### path.resolve([...paths]) 根据参数解析出一个绝对路径

传入参数：.../paths是传入的字符串参数，是路径序列或者路径片段

返回值：字符串

 **以应用程序为根目录**

**普通字符串代表子目录**

**/代表绝对路径根目录**

使用方法：

1 path.resolve()方法可以将路径或者路径片段解析成绝对路径

2 传入的路径从右至左被处理，后面每个path被依次解析，直到构造完成一个绝对路径。例如:path.resolve(‘/foo’，‘/bar’，‘baz’),将返回:/bar/baz

3 如果处理完全部给定的path片段后还未生成一个绝对路径，则当前工作目录(绝对路径)会被用上。

4 当传入的参数没有/时，将被传入解析到当前根目录

5 零长度的路径将被忽略

6 如果没有传入参数，将返回当前根目录

事例代码:

```javascript
path.resolve('/foo/bar', './baz')
// Returns: '/foo/bar/baz'
 

path.resolve('/foo/bar', '/tmp/file/')
// Returns: '/tmp/file'

path.resolve('wwwroot', 'static_files/png/', '../gif/image.gif')
// if the current working directory is /home/myself/node,
// this returns '/home/myself/node/wwwroot/static_files/gif/image.gif'
```

1 传入的路径从右向左解析，遇到第一个绝对路径时完成解析

2 ‘../’方法向前跳了一个目录

path.resolve()是一个修改和整合文件路径的方法

__dirname

运行下面的代码：

![img](https://img2020.cnblogs.com/blog/992473/202005/992473-20200521170321570-2068345043.png)

执行结果：

 ![img](https://img2020.cnblogs.com/blog/992473/202005/992473-20200521170351260-1857111710.png)

结果分析：
1.只传入__dirname也可以自动调用path.resolve方法
2.可以拼接路径字符串，但是不调用path.resolve()方法拼接失败
3.__dirname代表的是当前文件（a.js）的绝对路径
4.从右至左解析，遇到了绝对路径/src，因此直接返回

 

path.resolve()与__dirname最便捷的地方在于，与webpack结合更快捷的访问常用的文件。说白了就是解决了"由于访问文件所在的目录不同，访问固定目录下的某个文件，必须使用烦人的../../"这个痛点；

比如当我们结合webpack为目录设置快捷的import方法的时候。

修改之前 import foo from '../../../util/foo'

修改之后import foo from 'util/foo'

webpack配置 build/webpack.dev.conf.js

```
resolve:{
　　//解析扩展名
   extensions:[],
　alias:{
　　// 快捷访问入口
　　“util”:path.resolve(__dirname,'./src/util')
   }
}
```

### path.join([...paths])

path.join() 方法使用平台特定的分隔符把全部给定的 path 片段连接到一起，并规范化生成的路径。就是把多个参数字符串合并为一个路径字符串

![img](https://img2020.cnblogs.com/blog/992473/202005/992473-20200522095736750-2125347972.png)

 
