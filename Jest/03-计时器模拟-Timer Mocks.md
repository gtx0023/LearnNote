# 计时器模拟

本机计时器函数（即setTimeout、setInterval、clearTimeout、clearInterval）对于测试环境来说并不理想，因为它们依赖于实时经过。Jest可以用允许您控制时间流逝的函数交换计时器。伟大的斯科特！

```
// timerGame.js
'use strict';

function timerGame(callback) {
  console.log('Ready....go!');
  setTimeout(() => {
    console.log("Time's up -- stop!");
    callback && callback();
  }, 1000);
}

module.exports = timerGame;
```

```
// __tests__/timerGame-test.js
'use strict';

jest.useFakeTimers();

test('waits 1 second before ending the game', () => {
  const timerGame = require('../timerGame');
  timerGame();

  expect(setTimeout).toHaveBeenCalledTimes(1);
  expect(setTimeout).toHaveBeenLastCalledWith(expect.any(Function), 1000);
});
```

在这里，我们通过调用jest.useFakeTimers()来启用假计时器。这将使用模拟函数模拟setTimeout和其他计时器函数。计时器可以使用jest.useRealTimers()恢复到其正常行为。 

虽然您可以从任何地方（顶级、it块内部等）调用jest.useFakeTimers()或jest.useRealTimers()，但它是一个全局操作，将影响同一文件中的其他测试。此外，您需要在每次测试前调用jest.useFakeTimers()重置内部计数器。如果您计划在所有测试中不使用假计时器，您将希望手动清理，否则假计时器将在测试中泄漏：

```
afterEach(() => {
  jest.useRealTimers();
});

test('do something with fake timers', () => {
  jest.useFakeTimers();
  // ...
});

test('do something with real timers', () => {
  // ...
});
```

## 运行所有计时器

我们可能要为此模块编写的另一个测试是断言回调在1秒后调用。为此，我们将使用Jest的计时器控制API在测试中间快进时间：

```
test('calls the callback after 1 second', () => {
  const timerGame = require('../timerGame');
  const callback = jest.fn();

  timerGame(callback);

  // At this point in time, the callback should not have been called yet
  expect(callback).not.toBeCalled();

  // Fast-forward until all timers have been executed
  jest.runAllTimers();

  // Now our callback should have been called!
  expect(callback).toBeCalled();
  expect(callback).toHaveBeenCalledTimes(1);
});
```

## 运行挂起计时器

在某些情况下，您可能有一个递归计时器--这是一个在自己的回调中设置新计时器的计时器。对于这些，运行所有的计时器将是一个无休止的循环...因此，像jest.runAllTimers()这样的东西是不可取的。对于这些情况，您可以使用jest.runOnlyPendingTimers()：

```
// infiniteTimerGame.js
'use strict';

function infiniteTimerGame(callback) {
  console.log('Ready....go!');

  setTimeout(() => {
    console.log("Time's up! 10 seconds before the next game starts...");
    callback && callback();

    // Schedule the next game in 10 seconds
    setTimeout(() => {
      infiniteTimerGame(callback);
    }, 10000);
  }, 1000);
}

module.exports = infiniteTimerGame;
```

```
// __tests__/infiniteTimerGame-test.js
'use strict';

jest.useFakeTimers();

describe('infiniteTimerGame', () => {
  test('schedules a 10-second timer after 1 second', () => {
    const infiniteTimerGame = require('../infiniteTimerGame');
    const callback = jest.fn();

    infiniteTimerGame(callback);

    // At this point in time, there should have been a single call to
    // setTimeout to schedule the end of the game in 1 second.
    expect(setTimeout).toHaveBeenCalledTimes(1);
    expect(setTimeout).toHaveBeenLastCalledWith(expect.any(Function), 1000);

    // Fast forward and exhaust only currently pending timers
    // (but not any new timers that get created during that process)
    jest.runOnlyPendingTimers();

    // At this point, our 1-second timer should have fired it's callback
    expect(callback).toBeCalled();

    // And it should have created a new timer to start the game over in
    // 10 seconds
    expect(setTimeout).toHaveBeenCalledTimes(2);
    expect(setTimeout).toHaveBeenLastCalledWith(expect.any(Function), 10000);
  });
});
```

## 按时间提前计时器

另一种可能性是使用jest.advanceTimersByTime(msToRun)。调用此API时，所有计时器都会提前msToRun毫秒。所有已通过setTimeout()或setInterval()排队并将在此时间范围内执行的挂起的“宏任务”都将被执行。此外，如果这些宏任务计划在同一时间框架内执行的新宏任务，则这些宏任务将被执行，直到队列中没有更多的宏任务应在msToRun毫秒内运行。

```
// timerGame.js
'use strict';

function timerGame(callback) {
  console.log('Ready....go!');
  setTimeout(() => {
    console.log("Time's up -- stop!");
    callback && callback();
  }, 1000);
}

module.exports = timerGame;
```

```
it('calls the callback after 1 second via advanceTimersByTime', () => {
  const timerGame = require('../timerGame');
  const callback = jest.fn();

  timerGame(callback);

  // At this point in time, the callback should not have been called yet
  expect(callback).not.toBeCalled();

  // Fast-forward until all timers have been executed
  jest.advanceTimersByTime(1000);

  // Now our callback should have been called!
  expect(callback).toBeCalled();
  expect(callback).toHaveBeenCalledTimes(1);
});
```

最后，在某些测试中，能够清除所有挂起的计时器可能偶尔会很有用。为此，我们有jest.ClearAllTimers()。 

此示例的代码可在示例/计时器上获得。
