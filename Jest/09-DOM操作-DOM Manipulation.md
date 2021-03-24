# DOM操作

另一类通常被认为难以测试的函数是直接操作DOM的代码。让我们看看我们如何测试以下jQuery代码片段，该代码片段监听单击事件，异步获取一些数据，并设置跨度的内容。

```
// displayUser.js
'use strict';

const $ = require('jquery');
const fetchCurrentUser = require('./fetchCurrentUser.js');

$('#button').click(() => {
  fetchCurrentUser(user => {
    const loggedText = 'Logged ' + (user.loggedIn ? 'In' : 'Out');
    $('#username').text(user.fullName + ' - ' + loggedText);
  });
});
```

同样，我们在 \_\_tests\_\_/ 文件夹中创建一个测试文件：

```
// __tests__/displayUser-test.js
'use strict';

jest.mock('../fetchCurrentUser');

test('displays a user after a click', () => {
  // Set up our document body
  document.body.innerHTML =
    '<div>' +
    '  <span id="username" />' +
    '  <button id="button" />' +
    '</div>';

  // This module has a side-effect
  require('../displayUser');

  const $ = require('jquery');
  const fetchCurrentUser = require('../fetchCurrentUser');

  // Tell the fetchCurrentUser mock function to automatically invoke
  // its callback with some data
  fetchCurrentUser.mockImplementation(cb => {
    cb({
      fullName: 'Johnny Cash',
      loggedIn: true,
    });
  });

  // Use jquery to emulate a click on our button
  $('#button').click();

  // Assert that the fetchCurrentUser function was called, and that the
  // #username span's inner text was updated as we'd expect it to.
  expect(fetchCurrentUser).toBeCalled();
  expect($('#username').text()).toEqual('Johnny Cash - Logged In');
});
```

正在测试的函数在 \#button DOM元素上添加了一个事件侦听器，因此我们需要为测试正确设置DOM。Jest附带jsdom，它模拟DOM环境，就像您在浏览器中一样。这意味着我们调用的每个DOM API都可以以与在浏览器中观察相同的方式观察！ 

我们正在模拟 fetchCurrentUser.js ，这样我们的测试就不会发出真正的网络请求，而是解析为本地模拟数据。这确保了我们的测试可以在毫秒而不是几秒钟内完成，并保证了快速的单元测试迭代速度。 

此示例的代码可在 [examples/jquery](https://github.com/facebook/jest/tree/master/examples/jquery). 上获得。

