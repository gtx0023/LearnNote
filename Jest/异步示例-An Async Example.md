# 异步示例 

首先，在Jest中启用Babel支持，如入门指南中所述。 

让我们实现一个模块，该模块从API获取用户数据并返回用户名。

```
// user.js
import request from './request';

export function getUserName(userID) {
  return request('/users/' + userID).then(user => user.name);
}
```

在上面的实现中，我们希望request.js模块返回一个承诺。我们将调用链接到，然后接收用户名。

现在想象一下request.js的实现，它进入网络并获取一些用户数据：

```
// request.js
const http = require('http');

export default function request(url) {
  return new Promise(resolve => {
    // This is an example of an http request, for example to fetch
    // user data from an API.
    // This module is being mocked in __mocks__/request.js
    http.get({path: url}, response => {
      let data = '';
      response.on('data', _data => (data += _data));
      response.on('end', () => resolve(data));
    });
  });
}
```

因为我们不想在测试中转到网络，所以我们将在__mocks__文件夹中为我们的request.js模块创建一个手动模拟（该文件夹区分大小写，__MOCKS__将不起作用）。它可能看起来是这样的：

```
// __mocks__/request.js
const users = {
  4: {name: 'Mark'},
  5: {name: 'Paul'},
};

export default function request(url) {
  return new Promise((resolve, reject) => {
    const userID = parseInt(url.substr('/users/'.length), 10);
    process.nextTick(() =>
      users[userID]
        ? resolve(users[userID])
        : reject({
            error: 'User with ' + userID + ' not found.',
          }),
    );
  });
}
```

现在，让我们为我们的异步功能编写一个测试。

```
// __tests__/user-test.js
jest.mock('../request');

import * as user from '../user';

// The assertion for a promise must be returned.
it('works with promises', () => {
  expect.assertions(1);
  return user.getUserName(4).then(data => expect(data).toEqual('Mark'));
});
```

我们调用jest.mock（'../request'）来告诉Jest使用我们的手动模拟。它希望返回值是将被解析的承诺。您可以链接任意多的承诺，并随时调用期望，只要您在结尾返回一个承诺。

## `.resolves`

有一种不那么冗长的方法，使用解析将已实现的承诺的值与任何其他匹配器一起展开。如果承诺被拒绝，断言将失败。

```
it('works with resolves', () => {
  expect.assertions(1);
  return expect(user.getUserName(5)).resolves.toEqual('Paul');
});
```

## `async`/`await`

使用async/await语法编写测试也是可能的。以下是您如何编写以前的相同示例：

```
// async/await can be used.
it('works with async/await', async () => {
  expect.assertions(1);
  const data = await user.getUserName(4);
  expect(data).toEqual('Mark');
});

// async/await can also be used with `.resolves`.
it('works with async/await and resolves', async () => {
  expect.assertions(1);
  await expect(user.getUserName(5)).resolves.toEqual('Paul');
});
```

要在项目中启用async/await，请安装@babel/preset-env并在babel.config.js文件中启用该功能。

## 错误处理

可以使用.catch方法处理错误。确保添加预期.assertions以验证是否调用了一定数量的断言。否则，兑现的承诺不会通过测试：

```
// Testing for async errors using Promise.catch.
it('tests error with promises', () => {
  expect.assertions(1);
  return user.getUserName(2).catch(e =>
    expect(e).toEqual({
      error: 'User with 2 not found.',
    }),
  );
});

// Or using async/await.
it('tests error with async/await', async () => {
  expect.assertions(1);
  try {
    await user.getUserName(1);
  } catch (e) {
    expect(e).toEqual({
      error: 'User with 1 not found.',
    });
  }
});
```

## `.rejects`

.rejects帮助程序的工作原理与.resolves帮助程序类似。如果承诺被实现，测试将自动失败。不需要期望.sertions（编号），但建议验证在测试期间是否调用了一定数量的断言。否则，很容易忘记返回/等待.resolves断言。

```
// Testing for async errors using `.rejects`.
it('tests error with rejects', () => {
  expect.assertions(1);
  return expect(user.getUserName(3)).rejects.toEqual({
    error: 'User with 3 not found.',
  });
});

// Or using async/await with `.rejects`.
it('tests error with async/await and rejects', async () => {
  expect.assertions(1);
  await expect(user.getUserName(3)).rejects.toEqual({
    error: 'User with 3 not found.',
  });
});
```

此示例的代码可在示例/async上获得。 如果您想测试计时器，如setTimeout，请查看计时器模拟文档。
