# 监听插件 

Jest watch插件系统提供了一种方法，可以挂接到Jest的特定部分，并定义在按键时执行代码的手表模式菜单提示。这些功能结合起来，您可以为您的工作流开发自定义的交互式体验。 

## 监听插件接口

```
class MyWatchPlugin {
  // Add hooks to Jest lifecycle events
  apply(jestHooks) {}

  // Get the prompt information for interactive plugins
  getUsageInfo(globalConfig) {}

  // Executed when the key from `getUsageInfo` is input
  run(globalConfig, updateConfigAndRun) {}
}
```

## Jest中的钩子函数

要将 监听 插件 连接到 Jest，请在Jest配置中的 watchPlugins 下添加其路径：

```
// jest.config.js
module.exports = {
  // ...
  watchPlugins: ['path/to/yourWatchPlugin'],
};
```

自定义监视插件可以向Jest事件添加钩子。这些钩子可以在监听模式菜单中添加，也可以不添加交互式键。 

### apply（`jestHooks`）

Jest钩子可以通过实现 apply 方法来附加。此方法接收一个jestHooks参数，该参数允许插件挂接到测试运行生命周期的特定部分。

```
class MyWatchPlugin {
  apply(jestHooks) {}
}
```

下面是Jest中可用的钩子。

`jestHooks.shouldRunTestSuite(testSuiteInfo)`

返回布尔值（或 Promise<boolean> 用于处理异步操作），以指定是否应运行测试。

例如：

```
class MyWatchPlugin {
  apply(jestHooks) {
    jestHooks.shouldRunTestSuite(testSuiteInfo => {
      return testSuiteInfo.testPath.includes('my-keyword');
    });

    // or a promise
    jestHooks.shouldRunTestSuite(testSuiteInfo => {
      return Promise.resolve(testSuiteInfo.testPath.includes('my-keyword'));
    });
  }
}
```

`jestHooks.onTestRunComplete(results)`

在每次测试运行结束时调用。它将测试结果作为参数。

例如：

```
class MyWatchPlugin {
  apply(jestHooks) {
    jestHooks.onTestRunComplete(results => {
      this._hasSnapshotFailure = results.snapshot.failure;
    });
  }
}
```

`jestHooks.onFileChange({projects})`

每当文件系统发生更改时调用

`projects: Array<config: ProjectConfig, testPaths: Array<string>`: 包括Jest正在监视的所有测试路径。
例如：

```
class MyWatchPlugin {
  apply(jestHooks) {
    jestHooks.onFileChange(({projects}) => {
      this._projects = projects;
    });
  }
}
```

## 观看菜单集成

自定义监视插件还可以通过在getUseInfo方法中指定键/提示对和用于执行键的run方法来添加或覆盖监视菜单的功能。

### `getUsageInfo(globalConfig)`

要向监视菜单添加键，请实现getUseInfo方法，返回键和提示：

```
class MyWatchPlugin {
  getUsageInfo(globalConfig) {
    return {
      key: 's',
      prompt: 'do something',
    };
  }
}
```

这将在监视模式菜单中添加一行*(`› Press s to do something.`)*

```
Watch Usage
 › Press p to filter by a filename regex pattern.
 › Press t to filter by a test name regex pattern.
 › Press q to quit watch mode.
 › Press s to do something. // <-- This is our plugin
 › Press Enter to trigger a test run.
```

注意：如果插件的密钥已作为默认密钥存在，则插件将覆盖该密钥。

### `run(globalConfig, updateConfigAndRun)`

要处理来自 getUsageInfo 返回的键的按键事件，您可以实现 run 方法。此方法返回一个  ，当插件希望将控制权返回给Jest时，该 Promise\<boolean\> 可以解析。boolean 指定Jest是否应在获得控制权后重新运行测试。

- globalConfig：Jest当前全局配置的表示
- updateConfigAndRun：允许您在交互式插件运行时触发测试运行。

```
class MyWatchPlugin {
  run(globalConfig, updateConfigAndRun) {
    // do something.
  }
}
```

注意：如果您确实调用了updateConfigAndRun，则 run 方法不应解析为真实值，因为这将触发双重运行。

Authorized configuration keys

出于稳定性和安全原因，只能使用 updateConfigAndRun 更新部分全局配置密钥。当前白名单如下：

- [`bail`](https://www.jestjs.cn/docs/configuration#bail-number--boolean)
- [`changedSince`](https://www.jestjs.cn/docs/cli#--changedsince)
- [`collectCoverage`](https://www.jestjs.cn/docs/configuration#collectcoverage-boolean)
- [`collectCoverageFrom`](https://www.jestjs.cn/docs/configuration#collectcoveragefrom-array)
- [`collectCoverageOnlyFrom`](https://www.jestjs.cn/docs/configuration#collectcoverageonlyfrom-array)
- [`coverageDirectory`](https://www.jestjs.cn/docs/configuration#coveragedirectory-string)
- [`coverageReporters`](https://www.jestjs.cn/docs/configuration#coveragereporters-arraystring)
- [`notify`](https://www.jestjs.cn/docs/configuration#notify-boolean)
- [`notifyMode`](https://www.jestjs.cn/docs/configuration#notifymode-string)
- [`onlyFailures`](https://www.jestjs.cn/docs/configuration#onlyfailures-boolean)
- [`reporters`](https://www.jestjs.cn/docs/configuration#reporters-arraymodulename--modulename-options)
- [`testNamePattern`](https://www.jestjs.cn/docs/cli#--testnamepatternregex)
- [`testPathPattern`](https://www.jestjs.cn/docs/cli#--testpathpatternregex)
- [`updateSnapshot`](https://www.jestjs.cn/docs/cli#--updatesnapshot)
- [`verbose`](https://www.jestjs.cn/docs/configuration#verbose-boolean)

## 定制编号

插件可以通过Jest配置自定义。

```
// jest.config.js
module.exports = {
  // ...
  watchPlugins: [
    [
      'path/to/yourWatchPlugin',
      {
        key: 'k', // <- your custom key
        prompt: 'show a custom prompt',
      },
    ],
  ],
};
```

建议的配置名称：

- key：修改插件密钥。
- prompt：允许用户自定义插件提示中的文本。

如果用户提供了自定义配置，则它将作为参数传递给插件构造函数。

```
class MyWatchPlugin {
  constructor({config}) {}
}
```

## 选择一把好钥匙

Jest允许第三方插件覆盖其内置的一些功能键，但不是全部功能键。具体来说，以下键不可覆盖：

c（清除过滤器模式）
i（以交互方式更新不匹配的快照）
q（退出）
u（更新所有不匹配的快照）
w（显示监视模式使用情况/可用操作）

内置功能的以下键**可以被覆盖**：

p（测试文件名模式）
t（测试名称模式）

任何未被内置功能使用的密钥都可以被声明，正如您所期望的那样。尽量避免使用在各种键盘上难以获得的键 (e.g. `é`, `€`), 或默认情况下不可见的键（e.g. 许多Mac键盘没有 `|`, `\`, `[`, etc.）。

### 当冲突发生时

如果您的插件试图覆盖保留密钥，Jest将错误地显示一条描述性消息，类似于：

> 监听插件YourFaultyPlugin尝试注册键 q ，该键是内部保留的，用于退出手表模式。请更改此插件的配置键。

第三方插件也被禁止覆盖已由配置的插件列表（ WatchPlugins 数组设置）中先前存在的另一个第三方插件保留的密钥。发生这种情况时，您还会收到一条错误消息，试图帮助您修复此问题：

> 监听插件 YourFaultyPlugin 和 TheirFaultyPlugin 都尝试注册键 x。请更改其中一个冲突插件的键配置，以避免重叠。





