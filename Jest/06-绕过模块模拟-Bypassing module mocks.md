# 绕过模块模拟

Jest允许您在测试中模拟整个模块，这对于测试您的代码是否正确调用该模块的函数非常有用。但是，有时您可能希望在测试文件中使用模拟模块的部分，在这种情况下，您希望访问原始实现，而不是模拟版本。

考虑为此createUser函数编写一个测试用例：

```
// createUser.js
import fetch from 'node-fetch';

export const createUser = async () => {
  const response = await fetch('http://website.com/users', {method: 'POST'});
  const userId = await response.text();
  return userId;
};
```

您的测试将希望模拟 fetch 函数，以便我们可以确保它在没有实际发出网络请求的情况下被调用。但是，您还需要使用 Response（包装在 Promise 中）模拟 fetch 的返回值，因为我们的函数使用它来获取创建的用户的ID。因此，您可以尝试最初编写这样的测试：

```
jest.mock('node-fetch');

import fetch, {Response} from 'node-fetch';
import {createUser} from './createUser';

test('createUser calls fetch with the right args and returns the user id', async () => {
  fetch.mockReturnValue(Promise.resolve(new Response('4')));

  const userId = await createUser();

  expect(fetch).toHaveBeenCalledTimes(1);
  expect(fetch).toHaveBeenCalledWith('http://website.com/users', {
    method: 'POST',
  });
  expect(userId).toBe('4');
});
```

但是，如果您运行该测试，您会发现createUser函数将失败，引发错误：TypeError: response.text is not a function。这是因为您从node-fetch导入的 Response 类已被模拟（由于测试文件顶部的 jest.mock 调用）)所以它不再表现出它应该表现的方式。
为了解决这样的问题，Jest提供了 jest.requireActual 助手。要使上述测试工作，请对测试文件中的导入进行以下更改：

```
// BEFORE
jest.mock('node-fetch');
import fetch, {Response} from 'node-fetch';
```

```
// AFTER
jest.mock('node-fetch');
import fetch from 'node-fetch';
const {Response} = jest.requireActual('node-fetch');
```

这允许您的测试文件从 node-fetch 导入实际的 Response 对象，而不是模拟版本。这意味着测试现在将正确通过。
