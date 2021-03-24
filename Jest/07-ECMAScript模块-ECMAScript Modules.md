# ECMAScript模块

Jest附带了对ECMAScript模块（ESM）的实验支持。 

- 请注意，由于Jest的实验性质，在Jest的实现中存在许多已知和未知的错误和缺失的功能。您应查看跟踪问题和问题跟踪器上的标签，以了解最新状态。 
- 另请注意，Jest用于实现ESM支持的API仍被Node视为实验性的（从版本14.13.1开始）。 

在警告消失的情况下，这就是您在测试中激活ESM支持的方式。

1. 确保通过传递 transform: {} 禁用代码转换，或者将转换器配置为发出ESM而不是默认的CommonJS (CJS)。
2. 使用 --experimental-vm-modules 执行 node，例如节点 node --experimental-vm-modules node_modules/.bin/jest 或 node --experimental-vm-modules node_modules/.bin/jest 等。在Windows上，您可以使用 [`cross-env`](https://github.com/kentcdodds/cross-env) 设置环境变量。
3. 除此之外，我们还尝试遵循 node 激活“ESM模式”的逻辑（例如查看package.json或mjs文件中的类型），有关详细信息，请参阅他们的文档。
4. 如果您想将其他文件扩展名（如ts）视为ESM，请使用 [`extensionsToTreatAsEsm`](https://www.jestjs.cn/docs/configuration#extensionstotreatasesm-arraystring) 选项。

## ESM和CommonJS的区别

大多数差异都在Node的文档中解释，但除了其中提到的内容之外，Jest 还将一个特殊变量注入所有执行的文件--jest 对象。要在ESM中访问此对象，您需要从 @jest/globals 模块导入它。

```
import {jest} from '@jest/globals';

jest.useFakeTimers();

// etc.
```

请注意，我们目前不支持 ESM 中的 jest.mock ，但我们打算在未来添加适当的支持。请关注此问题以了解更新。
