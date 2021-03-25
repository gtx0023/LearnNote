# 故障处理

呃哦，出什么事了？使用本指南解决Jest的问题。

## 测试失败了，你不知道为什么

尝试使用Node中内置的调试支持。注意：这仅在Node.js 8+中有效。
在任何测试中 debugger; 语句，然后在项目目录中运行：

```
node --inspect-brk node_modules/.bin/jest --runInBand [any other arguments here]
or on Windows
node --inspect-brk ./node_modules/jest/bin/jest.js --runInBand [any other arguments here]
```

这将在外部调试器可以连接到的Node进程中运行Jest。请注意，该进程将暂停，直到调试器连接到它。

要在Google Chrome（或任何基于Chromium的浏览器）中调试，请打开浏览器，转到 chrome://inspect，然后单击“Open Dedicated DevTools for Node”，这将为您提供可以连接到的可用节点实例的列表。运行上述命令后，单击终端中显示的地址（通常类似 localhost:9229 ），您将能够使用 Chrome 的 DevTools 调试Jest。

将显示Chrome开发人员工具，并将在Jest CLI脚本的第一行设置断点（这样做是为了让您有时间打开开发人员工具，并防止Jest在您有时间之前执行）。单击屏幕右上角看起来像“播放”按钮的按钮继续执行。当Jest执行包含debugger 语句的测试时，执行将暂停，您可以检查当前作用域和调用堆栈。

> 注意：--runInBand cli 选项确保Jest在同一进程中运行测试，而不是为单个测试生成进程。通常，Jest会跨进程并行测试运行，但很难同时调试多个进程。

## 在VS代码中调试

使用Visual Studio Code的内置调试器调试Jest测试有多种方法。

要连接内置调试器，请按上述方式运行测试：

```
node --inspect-brk node_modules/.bin/jest --runInBand [any other arguments here]
or on Windows
node --inspect-brk ./node_modules/jest/bin/jest.js --runInBand [any other arguments here]
```

然后使用以下 launch.json 配置附加VS Code的调试器：

```
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "attach",
      "name": "Attach",
      "port": 9229
    }
  ]
}
```

要自动启动并附加到运行测试的进程，请使用以下配置：

```
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug Jest Tests",
      "type": "node",
      "request": "launch",
      "runtimeArgs": [
        "--inspect-brk",
        "${workspaceRoot}/node_modules/.bin/jest",
        "--runInBand"
      ],
      "console": "integratedTerminal",
      "internalConsoleOptions": "neverOpen",
      "port": 9229
    }
  ]
}
```

或以下适用于Windows：

```
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug Jest Tests",
      "type": "node",
      "request": "launch",
      "runtimeArgs": [
        "--inspect-brk",
        "${workspaceRoot}/node_modules/jest/bin/jest.js",
        "--runInBand"
      ],
      "console": "integratedTerminal",
      "internalConsoleOptions": "neverOpen",
      "port": 9229
    }
  ]
}
```

如果您使用的是 Facebook 的 create-react-app ，您可以使用以下配置调试 Jest 测试：

```
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug CRA Tests",
      "type": "node",
      "request": "launch",
      "runtimeExecutable": "${workspaceRoot}/node_modules/.bin/react-scripts",
      "args": ["test", "--runInBand", "--no-cache", "--env=jsdom"],
      "cwd": "${workspaceRoot}",
      "protocol": "inspector",
      "console": "integratedTerminal",
      "internalConsoleOptions": "neverOpen"
    }
  ]
}
```

有关节点调试的更多信息，请在此处找到。

## 在WebStorm中调试

在 WebStorm 中调试Jest测试的最简单方法是使用 Jest run/debug configuration 。它将启动测试并自动附加调试器。

在WebStorm菜单中 Run 选择 Edit Configurations... ，然后单击 + 并选择 Jest 。（可选）指定Jest配置文件、其他选项和环境变量。保存配置，将断点放入代码中，然后单击绿色调试图标开始调试。

如果您使用的是 Facebook 的 create-react-app，请在Jest run/debug 配置中在Jest包字段中指定 react-scripts 包的路径，并将 --env=jsdom 添加到Jest选项字段中。

## 缓存问题

转换脚本已更改或Babel已更新，并且Jest无法识别更改？

使用 --no-cache 重试。Jest缓存转换后的模块文件，以加快测试执行速度。如果您使用的是自己的自定义转换器，请考虑向其添加 getCacheKey 函数：[getCacheKey in Relay](https://github.com/facebook/relay/blob/58cf36c73769690f0bbf90562707eadb062b029d/scripts/jest/preprocessor.js#L56-L61)。

## 未解决的承诺

如果承诺根本无法解决，则可能会引发此错误：

```
- Error: Timeout - Async callback was not invoked within timeout specified by jasmine.DEFAULT_TIMEOUT_INTERVAL.`
```

最常见的是，这是由冲突的承诺实现引起的。考虑用您自己的承诺实现替换全局承诺实现，例如 global.Promise = jest.requireActual('promise'); 和 / 或将使用的承诺库合并到单个承诺库。 

如果您的测试长时间运行，您可能需要考虑通过调用 jest.setTimeout 来增加超时

```
jest.setTimeout(10000); // 10 second timeout
```

## 监听器问题

尝试使用 --no-watchman 运行 Jest，或将 watchman 配置选项设置为 false。

另请参见 [监听故障排除](https://facebook.github.io/watchman/docs/troubleshooting)

## Docker和/或持续集成（CI）服务器上的测试非常慢。

虽然Jest在具有快速SSD的现代多核计算机上大多数时候都非常快，但正如我们的用户所发现的那样，在某些设置上可能会很慢。

根据调查结果，缓解此问题并将速度提高50%的一种方法是顺序运行测试。

为此，您可以使用--runInBand在同一线程中运行测试：

```
# Using Jest CLI
jest --runInBand

# Using yarn test (e.g. with create-react-app)
yarn test --runInBand
```

在Travis-CI等持续集成服务器上加快测试执行时间的另一个替代方案是将最大工作池设置为~4。特别是在Travis-CI上，这可以将测试执行时间缩短一半。注意：适用于开源项目的Travis CI免费计划仅包括2个CPU核心。

```
# Using Jest CLI
jest --maxWorkers=4

# Using yarn test (e.g. with create-react-app)
yarn test --maxWorkers=4
```

## `coveragePathIgnorePatterns`似乎没有任何影响。

确保您没有使用babel-插件-istanbul插件。Jest包裹了伊斯坦布尔，因此也告诉伊斯坦布尔要使用覆盖收集来检测哪些文件。当使用babel-插件-istanbul时，Babel处理的每个文件都将具有覆盖收集代码，因此它不会被覆盖路径忽略。

## 定义测试编号

必须同步定义测试，以便Jest能够收集测试。

作为一个例子，说明为什么会出现这种情况，想象一下我们写了一个这样的测试：

```
// Don't do this it will not work
setTimeout(() => {
  it('passes', () => expect(1).toBe(1));
}, 0);
```

当Jest运行测试以收集测试时，它将找不到任何测试，因为我们已将定义设置为在事件循环的下一个刻度上异步发生。

注意：这意味着当您使用test.each时，您不能在beforeEach / beforeAll中异步设置表。



