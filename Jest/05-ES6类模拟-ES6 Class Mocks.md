# ES6类模拟

Jest可用于模拟导入到要测试的文件中的ES6类。 

ES6类是带有一些语法糖的构造函数。因此，ES6类的任何模拟都必须是函数或实际的ES6类（也是另一个函数）。因此，您可以使用模拟函数模拟它们。

## ES6类示例

我们将使用一个人为的示例，说明一个播放声音文件的类SoundPlayer，以及一个使用该类SoundPlayerConsumer的消费者类。我们将在SoundPlayerConsumer的测试中嘲笑SoundPlayer。

```
// sound-player.js
export default class SoundPlayer {
  constructor() {
    this.foo = 'bar';
  }

  playSoundFile(fileName) {
    console.log('Playing sound file ' + fileName);
  }
}
```

```
// sound-player-consumer.js
import SoundPlayer from './sound-player';

export default class SoundPlayerConsumer {
  constructor() {
    this.soundPlayer = new SoundPlayer();
  }

  playSomethingCool() {
    const coolSoundFileName = 'song.mp3';
    this.soundPlayer.playSoundFile(coolSoundFileName);
  }
}
```

## 创建ES6类mock的4种方法

### 自动模拟

调用jest.mock（'./sound-player'）返回一个有用的“自动模拟”，您可以用来监视对类构造函数及其所有方法的调用。它将ES6类替换为模拟构造函数，并将其所有方法替换为始终返回未定义的模拟函数。方法调用保存在AutomaticMock.mock.instances【索引】.methodName.mock.calls中。

请注意，如果您在类中使用箭头函数，它们将不会成为模拟的一部分。原因是箭头函数不存在于对象的原型中，它们只是持有对函数的引用的属性。 

如果您不需要替换类的实现，这是最容易设置的选项。例如：

```
import SoundPlayer from './sound-player';
import SoundPlayerConsumer from './sound-player-consumer';
jest.mock('./sound-player'); // SoundPlayer is now a mock constructor

beforeEach(() => {
  // Clear all instances and calls to constructor and all methods:
  SoundPlayer.mockClear();
});

it('We can check if the consumer called the class constructor', () => {
  const soundPlayerConsumer = new SoundPlayerConsumer();
  expect(SoundPlayer).toHaveBeenCalledTimes(1);
});

it('We can check if the consumer called a method on the class instance', () => {
  // Show that mockClear() is working:
  expect(SoundPlayer).not.toHaveBeenCalled();

  const soundPlayerConsumer = new SoundPlayerConsumer();
  // Constructor should have been called again:
  expect(SoundPlayer).toHaveBeenCalledTimes(1);

  const coolSoundFileName = 'song.mp3';
  soundPlayerConsumer.playSomethingCool();

  // mock.instances is available with automatic mocks:
  const mockSoundPlayerInstance = SoundPlayer.mock.instances[0];
  const mockPlaySoundFile = mockSoundPlayerInstance.playSoundFile;
  expect(mockPlaySoundFile.mock.calls[0][0]).toEqual(coolSoundFileName);
  // Equivalent to above check:
  expect(mockPlaySoundFile).toHaveBeenCalledWith(coolSoundFileName);
  expect(mockPlaySoundFile).toHaveBeenCalledTimes(1);
});
```

### 手动模拟

通过将模拟实现保存在__mocks__文件夹中，创建手动模拟。这允许您指定实现，并且它可以跨测试文件使用。

```
// __mocks__/sound-player.js

// Import this named export into your test file:
export const mockPlaySoundFile = jest.fn();
const mock = jest.fn().mockImplementation(() => {
  return {playSoundFile: mockPlaySoundFile};
});

export default mock;
```

导入所有实例共享的模拟和模拟方法：

```
// sound-player-consumer.test.js
import SoundPlayer, {mockPlaySoundFile} from './sound-player';
import SoundPlayerConsumer from './sound-player-consumer';
jest.mock('./sound-player'); // SoundPlayer is now a mock constructor

beforeEach(() => {
  // Clear all instances and calls to constructor and all methods:
  SoundPlayer.mockClear();
  mockPlaySoundFile.mockClear();
});

it('We can check if the consumer called the class constructor', () => {
  const soundPlayerConsumer = new SoundPlayerConsumer();
  expect(SoundPlayer).toHaveBeenCalledTimes(1);
});

it('We can check if the consumer called a method on the class instance', () => {
  const soundPlayerConsumer = new SoundPlayerConsumer();
  const coolSoundFileName = 'song.mp3';
  soundPlayerConsumer.playSomethingCool();
  expect(mockPlaySoundFile).toHaveBeenCalledWith(coolSoundFileName);
});
```

### 使用模块工厂参数调用

[`jest.mock`](https://www.jestjs.cn/docs/jest-object#jestmockmodulename-factory-options)()

jest.mock(path, moduleFactory) 采用模块工厂参数。模块工厂是返回模拟的函数。 

为了模拟构造函数，模块工厂必须返回构造函数。换句话说，模块工厂必须是返回函数的函数--高阶函数（HOF）。

```
import SoundPlayer from './sound-player';
const mockPlaySoundFile = jest.fn();
jest.mock('./sound-player', () => {
  return jest.fn().mockImplementation(() => {
    return {playSoundFile: mockPlaySoundFile};
  });
});
```

工厂参数的一个限制是，由于对jest.mock()的调用被提升到文件的顶部，因此不可能首先定义变量，然后在工厂中使用它。对于以单词“模拟”开头的变量，将出现例外。这取决于您保证它们将按时初始化!例如，由于在变量声明中使用了“假”而不是“模拟”，以下操作将引发范围外错误：

```
// Note: this will fail
import SoundPlayer from './sound-player';
const fakePlaySoundFile = jest.fn();
jest.mock('./sound-player', () => {
  return jest.fn().mockImplementation(() => {
    return {playSoundFile: fakePlaySoundFile};
  });
});
```

### 使用替换模拟 

mockImplementation()或mockImplementationOnce()

您可以通过在现有模拟上调用 mockImplementation() 来替换上述所有模拟，以便更改单个测试或所有测试的实现。

对jest.mock的调用被提升到代码的顶部。您可以稍后指定模拟，例如在 beforeAll() 中，通过在现有模拟上调用mockImplementation()（或 mockImplementationOnce(）)，而不是使用工厂参数。这还允许您在测试之间更改模拟（如果需要）：

```
import SoundPlayer from './sound-player';
import SoundPlayerConsumer from './sound-player-consumer';

jest.mock('./sound-player');

describe('When SoundPlayer throws an error', () => {
  beforeAll(() => {
    SoundPlayer.mockImplementation(() => {
      return {
        playSoundFile: () => {
          throw new Error('Test error');
        },
      };
    });
  });

  it('Should throw an error when calling playSomethingCool', () => {
    const soundPlayerConsumer = new SoundPlayerConsumer();
    expect(() => soundPlayerConsumer.playSomethingCool()).toThrow();
  });
});
```

## 深入：了解模拟构造函数函数

使用 jest.fn().mockImplementation() 构建构造函数模拟会使模拟看起来比实际复杂得多。本节介绍如何创建自己的模拟，以说明模拟是如何工作的。

### 手动模拟，这是另一个ES6类

如果您使用与__mocks__文件夹中的模拟类相同的文件名定义ES6类，它将用作模拟。此类将用于代替实际类。这允许您为类注入测试实现，但不提供监视调用的方法。 

对于人为的示例，模拟可能是这样的：

```
// __mocks__/sound-player.js
export default class SoundPlayer {
  constructor() {
    console.log('Mock SoundPlayer: constructor was called');
  }

  playSoundFile() {
    console.log('Mock SoundPlayer: playSoundFile was called');
  }
}
```

### 使用模块工厂参数

传递给 jest.mock(path, moduleFactory) 的模块工厂函数可以是返回函数*的HOF。这将允许在模拟上 new 。同样，这允许您为测试注入不同的行为，但不提供监视呼叫的方法。

*模块工厂函数必须返回一个函数

为了模拟构造函数，模块工厂必须返回构造函数。换句话说，模块工厂必须是返回函数的函数--高阶函数（HOF）。

```
jest.mock('./sound-player', () => {
  return function () {
    return {playSoundFile: () => {}};
  };
});
```

注意：箭头函数不起作用 

请注意，模拟不能是箭头函数，因为JavaScript中不允许在箭头函数上调用new。所以这行不通：

```
jest.mock('./sound-player', () => {
  return () => {
    // Does not work; arrow functions can't be called with new
    return {playSoundFile: () => {}};
  };
});
```

这将引发TypeError: _soundPlayer2.default不是构造函数，除非代码被传输到ES5，例如由@babel/preset-env。（ES5没有箭头函数和类，因此两者都将被转换为普通函数。）

## 跟踪使用情况（监视模拟）

注入测试实现很有帮助，但您可能也希望测试类构造函数和方法是否使用正确的参数调用。 

### 监视构造函数

为了跟踪对构造函数的调用，请将HOF返回的函数替换为Jest模拟函数。用 [`jest.fn()`](https://www.jestjs.cn/docs/jest-object#jestfnimplementation) 创建它，然后用 mockImplementation() 指定它的实现。

```
import SoundPlayer from './sound-player';
jest.mock('./sound-player', () => {
  // Works and lets you check for constructor calls:
  return jest.fn().mockImplementation(() => {
    return {playSoundFile: () => {}};
  });
});
```

这将使我们检查模拟类的使用情况，使用 SoundPlayer.mock.calls : expect(SoundPlayer).toHaveBeenCalled(); 或接近等效：expect(SoundPlayer.mock.calls.length).toEqual(1);

### 模拟非默认类导出

如果类不是模块的默认导出，则需要返回一个键与类导出名称相同的对象。

```
import {SoundPlayer} from './sound-player';
jest.mock('./sound-player', () => {
  // Works and lets you check for constructor calls:
  return {
    SoundPlayer: jest.fn().mockImplementation(() => {
      return {playSoundFile: () => {}};
    }),
  };
});
```

### 监视我们类的方法

我们的模拟类需要提供在测试期间调用的任何成员函数（示例中的playSoundFile），否则我们将在调用不存在的函数时出错。但我们可能也希望监视对这些方法的调用，以确保它们是用预期的参数调用的。 

每次在测试期间调用模拟构造函数时，都会创建一个新的对象。为了监视所有这些对象中的方法调用，我们使用另一个模拟函数填充playSoundFile，并在测试文件中存储对同一模拟函数的引用，以便在测试期间可用。

```
import SoundPlayer from './sound-player';
const mockPlaySoundFile = jest.fn();
jest.mock('./sound-player', () => {
  return jest.fn().mockImplementation(() => {
    return {playSoundFile: mockPlaySoundFile};
    // Now we can track calls to playSoundFile
  });
});
```

与此相当的手动模拟将是：

```
// __mocks__/sound-player.js

// Import this named export into your test file
export const mockPlaySoundFile = jest.fn();
const mock = jest.fn().mockImplementation(() => {
  return {playSoundFile: mockPlaySoundFile};
});

export default mock;
```

用法类似于模块工厂函数，只是您可以省略 jest.mock() 中的第二个参数，并且必须将模拟方法导入到测试文件中，因为它不再在那里定义。为此使用原始模块路径；不包括\_\_mocks\_\_。 

### 在测试之间清理

要清除对模拟构造函数及其方法的调用记录，我们在 beforeEach() 函数中调用 [`mockClear()`](https://www.jestjs.cn/docs/mock-function-api#mockfnmockclear)：

```
beforeEach(() => {
  SoundPlayer.mockClear();
  mockPlaySoundFile.mockClear();
});
```

## 完整示例

这里是一个完整的测试文件，它使用模块工厂参数 jest.mock：

```
// sound-player-consumer.test.js
import SoundPlayer from './sound-player';
import SoundPlayerConsumer from './sound-player-consumer';

const mockPlaySoundFile = jest.fn();
jest.mock('./sound-player', () => {
  return jest.fn().mockImplementation(() => {
    return {playSoundFile: mockPlaySoundFile};
  });
});

beforeEach(() => {
  SoundPlayer.mockClear();
  mockPlaySoundFile.mockClear();
});

it('The consumer should be able to call new() on SoundPlayer', () => {
  const soundPlayerConsumer = new SoundPlayerConsumer();
  // Ensure constructor created the object:
  expect(soundPlayerConsumer).toBeTruthy();
});

it('We can check if the consumer called the class constructor', () => {
  const soundPlayerConsumer = new SoundPlayerConsumer();
  expect(SoundPlayer).toHaveBeenCalledTimes(1);
});

it('We can check if the consumer called a method on the class instance', () => {
  const soundPlayerConsumer = new SoundPlayerConsumer();
  const coolSoundFileName = 'song.mp3';
  soundPlayerConsumer.playSomethingCool();
  expect(mockPlaySoundFile.mock.calls[0][0]).toEqual(coolSoundFileName);
});
```

