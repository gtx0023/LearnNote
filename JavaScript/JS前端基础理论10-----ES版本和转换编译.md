# JS前端基础理论-----ES? 现在与未来 

## 版本

- 第一个流行起来的 JavaScript 版本是 ES3，它成为浏览器 IE6-8 和早前的旧版 Android 2.x 移动浏览器的JavaScript 标准。  
- 出于某些政治原因，倒霉的 ES4 从来没有成形  
- 2009 年， ES5 正式发布（然后是 2011 年的 ES5.1），在当代浏览器（包括 Firefox、 Chrome、Opera、 Safari 以及许多其他类型）的进化和爆发中成为 JavaScript 广泛使用的标准。  
- 下一个 JavaScript 版本（发布日期从 2013 年拖到 2014 年，然后又到 2015 年）标签，之前的共识显然是 ES6。  
- 在 ES6 规范发展后期， 出现了这样的方案：有人建议未来的版本应该改成基于年份。
  - 比如 ES2016（也就是 ES7）来标示在 2016 年结束之前敲定的任何版本的规范。

## 转换＋编译  

落后规范这么多年对于 JavaScript 生态系统的未来是有害的。  只能利用专门的工具把你的 ES6 代码转化为等价（或近似！）的可以在 ES5 环境下工作的代码。  

工具化（提供工具）。具体来说是一种称为transpiling（transformation ＋ compiling，转换＋编译）的技术。  

通常在构建过程中使用 transpiler 执行这些转换，如同执行 linting、 minification，或者其他类似操作的步骤。  

### shim/polyfill

并非所有的 ES6 新特性都需要使用 transpiler，还有 polyfill（也称为 shim）这种模式。在可能的情况下， polyfill 会为新环境中的行为定义在旧环境中的等价行为。语法不能polyfill，而 API 通常可以。

举例来说， Object.is(..) 是一个用于检查两个值严格相等的新工具，而且不像 === 那样在处理 NaN 和 -0 值的时候有微妙的例外情况。对 Object.is(..) 应用 polyfill 非常简单：

```javascript
if (!Object.is) {
    Object.is = function(v1, v2) {
        // 检查-0
        if (v1 === 0 && v2 === 0) {
            return 1 / v1 === 1 / v2;
        }
        // 检查NaN
        if (v1 !== v1) {
            return v2 !== v2;
        }
        // 其余所有情况
        return v1 === v2;
    };
}  
```

