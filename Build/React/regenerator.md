# 启用async函数
regenerator函数让我们有把异步的函数写成同步的能力，使得代码的可维护性和可阅读性大大提高，而async则是regenerator的语法糖，一般来说regenerator函数的执行是需要co作为其执行器的，而async函数不用，因此使用起来更加方便优雅。关于async函数的知识可以参考阮一峰大神的书籍ECMAScript 6 入门。

如果现在用上面的配置，直接使用regenerator函数的话，会报regeneratorRuntime is not defined错误，这个错误之前也是困扰过我，后面才知道，regenerator无法直接被编译成es5的代码。必须添加polyfill，即regenerator的runtime库才能运行。

先安装对应的babel plugin，这个插件默认会帮我们把async函数转换成regenerator函数。

```javascript
npm install babel-plugin-transform-regenerator
```

代码转换好了后，代码运行时还要一个包 regenerator-runtime/runtime。

```javascript
npm i regenerator-runtime -S
```

然后在用到async函数的文件头部添加


```javascript
import 'regenerator-runtime/runtime';
```
