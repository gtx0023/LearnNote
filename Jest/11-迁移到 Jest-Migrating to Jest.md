# 迁移到 Jest

如果您想使用现有代码库试用Jest，有多种方法可以转换为Jest：

- 如果您使用的是茉莉花，或类似茉莉花的API（例如摩卡）,Jest应该基本上兼容，这使得迁移到的复杂性不那么高。
- 如果您使用的是AVA、期望.js（由Automattic）、茉莉、摩卡、代理查询、应.js或磁带，您可以使用Jest Codemods自动迁移（见下文）。
- 如果你喜欢印度茶，你可以升级到Jest并继续使用印度茶。但是，我们建议尝试Jest的断言及其失败消息。Jest Codemods可以从柴迁移（见下文）。

## jest代码模块

如果您使用的是 [AVA](https://github.com/avajs/ava), [Chai](https://github.com/chaijs/chai), [Expect.js (by Automattic)](https://github.com/Automattic/expect.js), [Jasmine](https://github.com/jasmine/jasmine), [Mocha](https://github.com/mochajs/mocha), [proxyquire](https://github.com/thlorenz/proxyquire), [Should.js](https://github.com/shouldjs/should.js) 或 [Tape](https://github.com/substack/tape) ，您可以使用第三方 [jest-codemods](https://github.com/skovhus/jest-codemods) 来执行大部分脏迁移工作。它使用 [jscodeshift](https://github.com/facebook/jscodeshift) 在代码库中运行代码转换。

要转换现有测试，请导航到包含测试的项目并运行：

```
npx jest-codemods
```

更多信息可在以下网址找到：https://github.com/skovhus/jest-codemods.
