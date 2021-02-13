## portfinder是什么

如果你使用 [vue-cli](https://link.zhihu.com/?target=https%3A//cli.vuejs.org/zh/) 创建过项目，一定会发现一个现象：同时启多个项目，并不会遇到端口占用的情况。理论上来说使用 **vue-cli** 创建的项目在使用 **npm run serve** 启动后会默认跑在 **8080** 端口。但在不停止第一个项目的情况下，同时启动多个项目，它们占用的端口会“自动”变成 8081、8082……

这是为什么呢，当然不是什么魔法了，是 vue-cli-service 内部依赖了一个叫 [portfinder](https://link.zhihu.com/?target=https%3A//github.com/http-party/node-portfinder) 的库，实现如下：

![img](https://pic3.zhimg.com/80/v2-f584a754c036760ee9bf2b0cdde858ce_720w.jpg)

vue-cli-service 这里只使用了 portfinder 的两个 api —— **basePort** 和 **getPortPromise 。**这两个 api 的作用见名知意，分别是“设置基准端口”和“异步获取可用端口”。

那么 portfinder 究竟是如何这么“**自动**”地实现端口分配呢，我们需要深入到 portfinder 的源码中深究一番了。打开 portfinder 源码一看，其实代码数量并不多，核心只有不到 **500** 行。而且这个库在 github 的关注者也不多，可能是有些人不知道它的存在，也有可能是大多数人仅仅视其为一个工具，不会过多关注。

![img](https://pic2.zhimg.com/80/v2-3669c1c98dc0249bec583bad34ffc125_720w.jpg)

## 为什么分析portfinder源码

那么我为什么要看它的源码呢，原因有以下几个：

1、之前就发现 vue-cli 创建的项目存在自动递增分配端口的功能，觉得很神奇，一直都很好奇其背后的原理

2、发现 portfinder 源码数量很少，觉得会比较容易看完

3、项目中想做一个基于这个东西的工具，便于 mock 数据访问更加自动化

------

多说无益，让我们进入重点——源码分析

## **portfinder原理**

本着先宏观后微观的顺序，我们先了解一下 portfinder 的整体工作流程：

![img](https://pic2.zhimg.com/80/v2-03dc2cba6097da055d9be81a9b0cd51d_720w.jpg)

那么其中每个环节都具体做了什么呢？

**portfinder初始化**：其实就是 portfinder 这个库对外暴露的 _defaultHosts 是一个立即执行函数（最原始的模块化实现方式，用于隔离作用域），此还暴露了以下属性和方法

![img](https://pic1.zhimg.com/80/v2-81e02a2bb19cceb6a649452e91e550b4_720w.jpg)

**读取本机网络接口**：这是通过 os.**networkInterfaces**() 拿到的

**获取网卡ip地址列表**：遍历上一步拿到的网络接口，取其 address 属性，存到一个数组中，最后追加一个 null

**获取可用端口**：这一步是要拿着 portfinder模块通过调用 getPort() / getPortPromise() / getPosts() 方法获得可用端口，这一步会读取参数 options，可以进一步配置 port、host和stopPort，分别作为开始扫描的端口号、主机名和最后一个扫描的端口号。

**测试端口在每个ip地址下的可用性**：这是整个库的**核心**，会去到一开始初始化的 ip列表，并对其依次尝试创建以该 ip 为主机名，以当前端口（配置中的初始端口）为端口的服务器，并建立对服务器的监听。当监听到访问后即证明该端口可用，此时会移除监听并销毁该服务器，并向可用端口列表增加该端口。

**判断端口是否可用**：这个是整个 portfinder 的**转折点**，当遇到可用端口后，如果调用的是获取单个可用端口的方法时，这里就返回结果了；如果没有获得可用端口，则会通过 **nextPort** 方法对当前端口 +1，来进行下一次循环，直到扫描到终止端口，如果还没有获得可用端口，则会结束，并通知无可用端口。

```js
// 
// ### function nextPort (port)
// #### @port {Number} Port to increment from.
// Gets the next port in sequence from the
// specified `port`.
//
exports.nextPort = function (port) {
  return port + 1;
};
```

这就是整个 portfinder 的工作流程和实现细节，整体看下来相对易懂。不过其中运用了 node 比较靠近底层的 **os** 模块（用户查询网卡信息）、**net** 模块（用于创建服务器）以及第三方库 **[async](https://link.zhihu.com/?target=https%3A//github.com/caolan/async)**

关于如何具体判断端口可用，是根据所提供的主机名和端口创建一个服务器，并建立对 **listening** 事件的一次性监听（**once**），如果服务器创建成功，即意味着该 ip 地址下的该端口可用，并会触发关于 listening 的监听。当 listening 的监听回调触发后会移除服务器对 error 的监听，并关闭服务器，完成可用端口获取。

核心代码如下：

```js
internals.testPort = function(options, callback) {
  if (!callback) {
    callback = options;
    options = {};
  }

  options.server = options.server  || net.createServer(function () {
    //
    // Create an empty listener for the port testing server.
    //
  });
  function onListen () {
    debugTestPort("done w/ testPort(): OK", options.host, "port", options.port);

        options.server.removeListener('error', onError);
        options.server.close();
      callback(null, options.port);
  }
  options.server.once('listening', onListen);

  if (options.host) {
    options.server.listen(options.port, options.host);
  } else {
    /*
      Judgement of service without host
      example:
        express().listen(options.port)
    */
    options.server.listen(options.port);
  }
};
```
